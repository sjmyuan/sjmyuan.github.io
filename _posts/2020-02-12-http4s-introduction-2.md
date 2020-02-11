---
title: Http4s v0.20 Introduction
tags:
- Scala
categories:
- Scala Tutorial
date: 2020-02-12 00:18 +0800
---
# What is http4s?

> http4s is a typeful, functional, streaming HTTP for Scala. 

The current status of each version

![enter image description here](https://tva1.sinaimg.cn/large/006tNbRwly1gblwx28ndlj30mz0ax751.jpg)

This introduction is based on the stable version 0.20

# How to install?

Add following configuration into your **build.sbt**

```scala
val Http4sVersion = "0.20.17"
libraryDependencies += Seq(
  "org.http4s"  %% "http4s-blaze-server"  % Http4sVersion, // http server implementation
  "org.http4s"  %% "http4s-blaze-client"  % Http4sVersion, // http client implementation
  "org.http4s"  %% "http4s-circe"         % Http4sVersion, // supply some utility methods to convert the Encoder/Decoder of circe to the EntityEncoder/EntityDecoder of http4s
  "org.http4s"  %% "http4s-dsl"           % Http4sVersion  // supply lots of syntax to parse request
 ) 
```

Add following package into you import header

``` scala
import org.http4s.dsl.io._
import org.http4s.implicits._
import org.http4s.circe._
```

# Core Concept

![](https://tva1.sinaimg.cn/large/0082zybply1gbrq9tdj7xj30hu0lzjsb.jpg)


## Request

```scala
sealed abstract case class Request[F[_]](
  method: Method = Method.GET,
  uri: Uri = Uri(path = "/"),
  httpVersion: HttpVersion = HttpVersion.`HTTP/1.1`,
  headers: Headers = Headers.empty,
  body: EntityBody[F] = EmptyBody,
  attributes: Vault = Vault.empty
) extends Message[F]
```

The `Request` model contains all the information needed by a `http request`.
The body of `http request` is `EntityBody` which is a `fs2.Stream[F, Byte]`. 

Obviously when the server receive a request, it's not an easy work to convert `Stream` to a data model.
So there is a type class called `EntityDecoder` to help us to do this work.

```scala
trait EntityDecoder[F[_], T] { 
  def decode(msg: Message[F], strict: Boolean): DecodeResult[F, T]
}
```

its consumer is defined in `Message` which is the parent of `Request`

```scala
trait Message[F[_]] {
  def attemptAs[T](implicit decoder: EntityDecoder[F, T]): DecodeResult[F, T] =
    decoder.decode(this, strict = false)

  def as[A](implicit F: MonadError[F, Throwable], decoder: EntityDecoder[F, A]): F[A] =
    attemptAs.leftWiden[Throwable].rethrowT
}
```

Actually, the `EntityDecoder` didn't help us too much, because the `decode` method still get a `Message` which contains `Stream` body.
In term of the widely usage of JSON and circe in the RestfulAPI, http4s supply `jsonOf` to let us convert the `Decoder` of circe to `EntityDecoder`

```scala
import org.http4s.circe._

implicit val personDecoder: Decoder[Person] = new Decoder[Person] {
  override def apply(c: HCursor): Decoder.Result[Person] =
    for {
      name <- c.get[String]("name")
      age <- c.get[Option[Int]]("age")
      phoneNumbers <- c.get[List[String]]("phone")
    } yield Person(name, age, phoneNumbers)
}
implicit val personEntityDecoder: EntityDecoder[IO, Person] = jsonOf[IO, Person]
```

But when you want to send a `http request` by a `http client`, you need to convert a data model to `Stream`.
http4s supply another type class called `EntityEncoder` to do this work and also supply `jsonOfEncoder` to let you use JSON and circe `Encoder`.

```scala
final case class Entity[+F[_]](body: EntityBody[F], length: Option[Long] = None)
trait EntityEncoder[F[_], A] {
  def toEntity(a: A): Entity[F]
}
```

```scala
trait Message[F[_]] {
  def withEntity[T](b: T)(implicit w: EntityEncoder[F, T]): Self
}
```

```scala
import org.http4s.circe._

implicit val personEncoder: Encoder[Person] = new Encoder[Person] {
  override def apply(a: Person): Json = Json.obj(
    "name" -> a.name.asJson,
    "age" -> a.age.asJson,
    "phone" -> a.phoneNumbers.asJson
  )
}
implicit val personEntityEncoder: EntityEncoder[IO, Person] = jsonEncoderOf[IO, Person]
```

## Response

```scala
final case class Response[F[_]](
  status: Status = Status.Ok,
  httpVersion: HttpVersion = HttpVersion.`HTTP/1.1`,
  headers: Headers = Headers.empty,
  body: EntityBody[F] = EmptyBody,
  attributes: Vault = Vault.empty
) extends Message[F]
```

Except the `status` attribute, the usage of `EntityEncoder` and `EntityDecoder`, most parts of `Response` are same as `Request`.

When a server want to return a `http response`, the `EntityEncoder` will be used.

```scala
case class Person(name: String, age: Option[Int], phoneNumbers: List[String])
implicit val personEntityEncoder: EntityEncoder[IO, Person] = jsonEncoderOf[IO, Person]

Ok(Person("Job", Some(18), List("111111"))) // send back Ok status with body
```

When a clent want to parse a `http response`, the `EntityDecoder` will be used.

```scala
trait Client[F[_]] {
  def expect[A](req: F[Request[F]])(implicit d: EntityDecoder[F, A]): F[A]
  def expect[A](uri: Uri)(implicit d: EntityDecoder[F, A]): F[A]
  def fetchAs[A](req: Request[F])(implicit d: EntityDecoder[F, A]): F[A]
  def fetchAs[A](req: F[Request[F]])(implicit d: EntityDecoder[F, A]): F[A]
}
```

## Routes

```scala
  type Http[F[_], G[_]] = Kleisli[F, Request[G], Response[G]]
  type HttpApp[F[_]] = Http[F, F]
  type HttpRoutes[F[_]] = Http[OptionT[F, ?], F]
```

A `HttpRoutes` is just a function which take `Request` as input and produce `Response` as output.

Usually we will build differnt `Routes` for differnt business logic, so the `HttpRoutes` won't return `Response` for every `Request`, that's why it return `Option[Response]`.

But the `http server` should be able to handle all the `Request`, so we need to give a default route, then the `HttpRoutes` will become `HttpApp` which is used in `Server`

## Middleware

A middleware is just a function `HttpApp[F[_], G[_]] => HttpApp[F[_], G[_]]`

# Usage

## How to start a server?

``` scala
object Main extends IOApp {
  def helloWorldRoutes: HttpRoutes[IO] = HttpRoutes
      .of[IO]({
        case GET -> Root / "hello" => Ok("Hello!!")
        })
  override def run(args: List[String]): IO[ExitCode] = {
    val resource = for {
      server <- BlazeServerBuilder[IO]
        .bindHttp(8888, "0.0.0.0")
        .withHttpApp(helloWorldRoutes.orNotFound)
        .resource
    } yield server
    resource.use(_ => IO.never)
  }
}
```

## How to create a route?

`HttpRoutes` is a type alias of `Kleisli[OptionT[F, ?], Request[F], Response[F]]` which is complicated,
http4s supply `HttpRoutes.of` to help us convert a paritial function `Request[F] => Response[F]` to `HttpRoutes`

~~~ scala
def helloWorldRoutes: HttpRoutes[IO] = HttpRoutes
    .of[IO]({
      case GET -> Root / "hello" => Ok("Hello!!")
     })
~~~

`HttpRoutes` is the main place of business logic, in which we will parse `Request`, do computaion then construct the `Response`.

### How to parse the Uri of Request?

To parse the Uri, http4s supply a set of dsl to help us and it already give the dsl implementation of `IO`.

Here we wil use `IO` as our effect, so we need to import its dsl when we parse the Uri

```scala
import org.http4s.dsl.io._
```

* Method

  ```scala
    case (GET|POST) -> Root / "hello" =>
    //       ^
    //     Method
  ``` 

* Path parameter

  ``` scala
    case GET -> Root / "hello" / name / IntVar(id) / LongVar(hash) =>
    //                            ^        ^            ^
    //                         String     Int          Long
  ```

* Query Paramerter
  * Required query parameter

    ``` scala
      // url is /hello?user_name=john

      object UserName extends QueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? UserName(name) =>
      //                                      ^
      //                            Required query parameter(String)
    ```

  * Optional query parameter

    ``` scala
      // url is /hello?user_name=john
      // or /hello

      object UserName extends OptionalQueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? UserName(name) =>
      //                                      ^
      //                           Optional query parameter(Option[String])
    ```

  * Multiple query parameter

    ``` scala
      // url is /hello?user_name=john&age=18

      object UserName extends QueryParamDecoderMatcher[String]("user_name")
      object Age extends OptionalQueryParamDecoderMatcher[Int]("age")
      case GET -> Root / "hello" :? UserName(name) +& Age(age) =>
      //                                           ^
      //                                 Multiple query parameter
    ```

  * Optional query parameter which occurs multiple times

    ``` scala
      // url is /hello?user_name=john&user_name=lili
      // or /hello

      object UserNames extends OptionalMultiQueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? UserNames(names) =>
      //                                      ^
      //                        Occurs multiple times query parameter(List[String])
    ```


### How to get the hearder of Request?

``` scala
  case request@GET -> Root / "hello" =>
    println(request.headers)
    Ok()
```

### How to get the body of Request?

Assume we have a model `Person` in our program

```scala
case class Person(name: String, age: Option[Int], phoneNumbers: List[String])
```

And we already defined the `EntityDecoder` of `Person`

```scala
object Person {
  implicit val personDecoder: Decoder[Person] = new Decoder[Person] {
    override def apply(c: HCursor): Decoder.Result[Person] =
      for {
        name <- c.get[String]("name")
        age <- c.get[Option[Int]]("age")
        phoneNumbers <- c.get[List[String]]("phone")
      } yield Person(name, age, phoneNumbers)
  }

  implicit val personEntityDecoder: EntityDecoder[IO, Person] = jsonOf[IO, Person]
}
```

Then if the body of `Request` is a JSON

```json
{
  "name": "Job",
  "age": 18,
  "phone": ["1", "2"]
}
```

We can get the body of `Request` like this

``` scala
  case request@POST -> Root / "hello" =>
    println(request.as[Person])
    Ok()
```

## How to construct a Response?

http4s already defined the `Status` model for each status code, it's a simple entry of `Response`. we need to choose the staus code first, then pass the header or body to it.

Here we will use `200` to give an example.

* Header

  ``` scala
  Ok(Header("Vary", "Origin,Access-Control-Request-Methods"))
  ```

* Body

  Assume we already defined the `EntityEncoder` of `Person`

  ```scala
  object Person {
    implicit val personEncoder: Encoder[Person] = new Encoder[Person] {
      override def apply(a: Person): Json = Json.obj(
        "name" -> a.name.asJson,
        "age" -> a.age.asJson,
        "phone" -> a.phoneNumbers.asJson
      )
    }

    implicit val personEntityEncoder: EntityEncoder[IO, Person] = jsonEncoderOf[IO, Person]
  }
  ```

  Then we can return `Person` as body like this

  ``` scala
  Ok(Person("Job", Some(18), List("1111")))
  ```

## How to add middleware?

### How to support CORS?

``` scala
def helloWorldRoutes: HttpRoutes[IO] = CORS(HttpRoutes
    .of[IO]({
      case GET -> Root / "hello" => Ok("Hello!!")
      }))
```

## How to create a client?

```scala
val resource = for {
 client <- BlazeClientBuilder[IO](scala.concurrent.ExecutionContext.Implicits.global).resource
} yield client

resource.use(client => client.expect[Person](Uri.unsafefromString("http://localhost:8888/hello"))).unsafeRunSync()
```

## How to send a GET request?

```scala
val response: IO[Person] = client.expect[Person](Uri.unsafefromString("http://localhost:8888/hello"))
```

## How to send a POST request?

```scala
val request: Request[IO] = Request[IO](Method.POST, Uri.unsafefromString("http://localhost:8888/hello")).withEntity(Person("Jon", Some(18), List("1111")))
val response: IO[Response[IO]] = client.expect[String](IO(request))
```

# Summary

Now you should be able to start a simple http server and send http request to remote server.
if you want to try more feature, please clone the following repo and have a fun.

```sh
git clone git@github.com:sjmyuan/http4s-intro.git
```

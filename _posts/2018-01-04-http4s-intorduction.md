---
layout: post
title: Http4s Intorduction
excerpt: ""
---
# Contents
{:.no_toc}

* Toc
{:toc}

## What is http4s?
> Http4s is a minimal, idiomatic Scala interface for HTTP services. Http4s is Scala's answer to Ruby's Rack, Python's WSGI, Haskell's WAI, and Java's Servlets.

The main versions of http4s:

* 0.16

  The final release based on Scalaz and scalaz-stream

* 0.17

  The first release based on Cats and FS2

* 0.18

  The first release based on cats-effect

The current status of each version

![enter image description here](/images/http4s-version-status.png)

This introduction is based on the version 0.17

## How to install?

Add following configuration into your **build.sbt**

~~~ scala
Http4sVersion=0.17.5
libraryDependencies += Seq(
  "org.http4s"  %% "http4s-blaze-server"  % Http4sVersion, // htt4s native backend server
  "org.http4s"  %% "http4s-blaze-client"  % Http4sVersion, // http4s http client
  "org.http4s"  %% "http4s-circe"         % Http4sVersion, // support encoding and decoding json based on circe
  "org.http4s"  %% "http4s-dsl"           % Http4sVersion) // support process request in type level
~~~

Add following package into you import header

~~~ scala
import org.http4s._
import org.http4s.dsl._
import org.http4s.circe._
~~~

## How to setup a server?

~~~ scala
import org.http4s.server.blaze.BlazeBuilder
BlazeBuilder.bindHttp(port, "0.0.0.0")
  .mountService(HellowordService(), "/helloworld") // HellowordService is our HttpService, will explain it later.
                                                   // /helloworld is the route endpoint
  .serve                                           // will block the main process
~~~

### How to create a service?

An HttpService is a simple alias for [Kleisli[Task, Request, Response]](https://typelevel.org/cats/datatypes/kleisli.html), you can create a service simply like this:

~~~ scala
object HellowordService{
  def apply():HttpService = HttpService {
    case GET -> Root / "hello" / name =>
      Ok(s"Hello, $name.")
  }
}
~~~

### How to process the request uri?

* Method

  ~~~ scala
    case (GET|POST) -> Root / "hello" / name =>
    //       ^
    //     Method
  ~~~

* Path parameter

  ~~~ scala
    case GET -> Root / "hello" / name / IntVar(id) / LongVar(hash) =>
    //                            ^        ^            ^
    //                         String     Int          Long
  ~~~

* Query Paramerter
  * Required query parameter

    ~~~ scala
      // url is /hello?user_name=john

      object userName extends QueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? userName(name) =>
      //                                      ^
      //                            Required query parameter
    ~~~

  * Optional query parameter

    ~~~ scala
      // url is /hello?user_name=john
      // or /hello

      object userName extends OptionalQueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? userName(name) =>
      //                                      ^
      //                           Optional query parameter
    ~~~

  * Multiple query parameter

    ~~~ scala
      // url is /hello?user_name=john&password=123

      object userName extends QueryParamDecoderMatcher[String]("user_name")
      object password extends QueryParamDecoderMatcher[String]("password")
      case GET -> Root / "hello" :? userName(name) +& password(psw) =>
      //                                           ^
      //                                 Multiple query parameter
    ~~~

  * Optional query parameter which occurs multiple times

    ~~~ scala
      // url is /hello?user_name=john&user_name=lili
      // or /hello

      object userName extends OptionalMultiQueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? userName(name) =>
      //                                      ^
      //                        Occurs multiple times query parameter(List(john,lili))
    ~~~


### How to process request?

* Header

  ~~~ scala
    case request@GET -> Root / "hello" =>
    puts request.headers
  ~~~

* Body

  ~~~ scala
    // body is {"name": "john"}

    case class Name(name:String)
    val decoder = implicitly[Decoder[Name]]
    implicit val entityDecoder = jsonOf(decoder)

    case request@GET -> Root / "hello" =>
    puts request.as[Name]
  ~~~

### How to return response?

* Header

~~~ scala
Ok().putHeaders(Header("Vary", "Origin,Access-Control-Request-Methods"))
~~~

* Status

~~~ scala
Ok()
BadRequest()
~~~

* Body

~~~ scala
case class Name(name:String)
val lili = Name("lili")
Ok(lili.asJson.noSpaces)
~~~

## How to add middleware?

### How to support CORS?

~~~ scala
object HellowordService{
  def apply():HttpService = CORS(HttpService {
  //                         ^
  //                    CORS middleware
    case GET -> Root / "hello" / name =>
      Ok(s"Hello, $name.")
  })
}
~~~


## How to test?

### Unit test

* Mock request

  ~~~ scala
  val mockRequest = Request(Method.GET, Uri(path = "/", query = Query(("name", Some("lili")))))
  HellowordService().run(mockRequest).unsafeRun() // return the response of this request
  ~~~

* Mock response

  ~~~ scala
  val content = Source.fromResource("content.json").mkString.getBytes
  val response = DisposableResponse(Response(body = Stream.emits[Task,Byte](content)), Task.now(()))
  ~~~

### E2E test

* Mock server

  ~~~ scala
  def mockServer(port: Int, mounts: Map[String, HttpService]): Server = {
  //                          ^
  //                        routes
    mounts.foldLeft[ServerBuilder](BlazeBuilder.bindHttp(port, "localhost"))((builder, mount) => {
      builder.mountService(mount._2, mount._1)
    }).run
  }

  val server = mockServer(8080,Map( "/hello", HellowordService()))
  //do test
  server.shutdownNow()
  ~~~

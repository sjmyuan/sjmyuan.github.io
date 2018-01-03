---
layout: post
title: Http4s Intorduction
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

An HttpService is a simple alias for Kleisli[Task, Request, Response], you can create a service simply like this:

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
  case GET -> Root / "hello" / name =>
  //    ^
  //  Method
~~~

* Path parameter

~~~ scala
  case GET -> Root / "hello" / name =>
  //                            ^
  //                        Path parameter
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
      object userName extends OptionalQueryParamDecoderMatcher[String]("user_name")
      case GET -> Root / "hello" :? userName(name) =>
      //                                      ^
      //                           Optional query parameter
    ~~~

### How to process request?

#### Header
#### Status
#### Body

### How to return response?

#### Header
#### Status
#### Body

## How to add plugin?

### How to support CORS?

## How to test?

### Unit test
### E2E test

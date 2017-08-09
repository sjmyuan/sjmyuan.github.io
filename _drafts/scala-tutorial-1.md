title: Scala Tutorial 1
excerpt: ""
---
# Contents
{:.no_toc}

* Toc
{:toc}

## What does the scala project looks like?
By default, sbt follows Maven project layout i.e. Scala source files are placed inside src/main/scala and test source files are placed inside src/test/scala
![scala-project-structure](/images/scala-project-structure.png)

+ build.sbt
  Setting dependency, project configuration, task, scalac options, and so on.

+ plugin.sbt
  Setting the plugin of this project

+ build.properties
  Setting the sbt version

+ Dependencies.scala
  Define the custom task or configuration

## How to create a scala project?
+ Copy
  Copy an exsiting project, then modify the build.sbt, plugin.sbt, build.properties to use your project configuration

+ Shell
  Generate an universal project structure by shell script, here is an [example](https://github.com/sjmyuan/sbt-structure-generator)

+ Giter8
  sbt support Giter8 template, we can generate an project structure from an template

  ~~~ scala
  sbt new scala/scala-seed.g8
  ~~~

  For more template, please visit the Giter8 [wiki](https://github.com/foundweekends/giter8/wiki/giter8-templates)

## How to config project?
:= is a function defined in the sbt library.
It is used to define a setting that overwrites any previous value without referring to other settings.

![sbt-overriding-values](/images/sbt-overriding-settings.png)

+ Common options
  ~~~ scala
  name := "tasky"
  version := "0.1.0"
  scalaVersion := "2.11.6"
  sbtVersion := "0.13.8"
  ~~~

+ scala compiler options
  ~~~ scala
  scalacOptions ++= Seq("-feature", "-language:_", "-unchecked", "-deprecation", "-encoding", "utf8")
  ~~~

You can find all the options by following command
~~~ scala
sbt settings
~~~

## How to add dependency?
Dependency has following format in which the configuration is optional:

~~~ scala
groupID % artifactID % version % configuration
~~~

We can add dependency like this:

![sbt-appending-settings-1](/images/sbt-appending-settings-1.png)
![sbt-appending-settings-2](/images/sbt-appending-settings-2.png)

~~~ scala
libraryDependencies += "org.scalatest" %% "scalatest" % "2.2.6" % "test" // add single dependency
libraryDependencies ++= Seq(
  "com.typesafe.slick" %% "slick"                   % "2.1.0",
  "net.databinder"     %% "unfiltered-netty-server" % "0.8.4")           // add multiple dependencies
~~~

## How to add plugin?
Add following settings into plugins.sbt

~~~
addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.7.4")
~~~

## How to add another repository?

~~~ scala
resolvers += "Scalaz Bintray Repo" at "http://dl.bintray.com/scalaz/releases"
~~~

## How to define tasks?

+ Defination

~~~scala
val gitCommitCountTask = taskKey[String]("Prints commit count of the current branch")
gitCommitCountTask := {
  val branch = Process("git symbolic-ref -q HEAD").lines.head.replace("refs/heads/","")
  val commitCount = Process(s"git rev-list --count $branch").lines.head
  println(s"total number of commits on [$branch]: $commitCount")
  commitCount
  }
~~~

+ Depend other task, just use other task key like a function

~~~scala
val logTask = taskKey[String]("log the commit count")
gitCommitCountTask := {
  val commitCount = gitCommitCountTask.value
  println(s"Info: $commitCount")
  ""
  }
~~~

You can find all the task by following command
~~~ scala
sbt tasks
~~~

## How to publish package?

~~~ scala
sbt build
sbt test
sbt run
sbt package
~~~

## How to write main method?
+ App
+ Object
## How to write test?

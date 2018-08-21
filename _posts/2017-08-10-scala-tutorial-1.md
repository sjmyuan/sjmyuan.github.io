---
title: Scala Tutorial 1
excerpt: ""
---

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
  sbt new sjmyuan/scala-seed.g8
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
+ App trait

  ~~~ scala
  object Hello extends Greeting with App {
    println(greeting)
  }
  ~~~

+ Object main method

  ~~~ scala
  object Hello1 extends Greeting {
    def main(args:Array[String]): Unit = {
      println(greeting)
    }
  }
  ~~~

## How to write test?
+ FunSuite

  ~~~ scala
  import org.scalatest.FunSuite
  class SetSuite extends FunSuite {
    test("An empty Set should have size 0") {
      assert(Set.empty.size == 0)
    }

    test("Invoking head on an empty Set should produce NoSuchElementException") {
      assertThrows[NoSuchElementException] {
        Set.empty.head
      }
    }
  }
  ~~~

+ FlatSpec

  ~~~ scala
  import org.scalatest.FlatSpec
  class SetSpec extends FlatSpec {
    "An empty Set" should "have size 0" in {
      assert(Set.empty.size == 0)
    }

    it should "produce NoSuchElementException when head is invoked" in {
      assertThrows[NoSuchElementException] {
        Set.empty.head
      }
    }
  }
  ~~~

+ FunSpec

  ~~~ scala
  import org.scalatest.FunSpec
  class SetSpec extends FunSpec {
    describe("A Set") {
      describe("when empty") {
        it("should have size 0") {
          assert(Set.empty.size == 0)
        }
        it("should produce NoSuchElementException when head is invoked") {
          assertThrows[NoSuchElementException] {
            Set.empty.head
          }
        }
      }
    }
  }
  ~~~

+ WordSpec

  ~~~ scala
  import org.scalatest.WordSpec

  class SetSpec extends WordSpec {
    "A Set" when {
      "empty" should {
        "have size 0" in {
          assert(Set.empty.size == 0)
        }
        "produce NoSuchElementException when head is invoked" in {
          assertThrows[NoSuchElementException] {
            Set.empty.head
          }
        }
      }
    }
  }
  ~~~

+ FreeSpec

  ~~~ scala
  import org.scalatest.FreeSpec

  class SetSpec extends FreeSpec {
    "A Set" - {
      "when empty" - {
        "should have size 0" in {
          assert(Set.empty.size == 0)
        }
        "should produce NoSuchElementException when head is invoked" in {
          assertThrows[NoSuchElementException] {
            Set.empty.head
          }
        }
      }
    }
  }
  ~~~

+ PropSpec

  ~~~ scala
  import org.scalatest._import

  prop._import scala.collection.immutable._

  class SetSpec extends PropSpec with TableDrivenPropertyChecks with Matchers {
    val examples = Table("set", BitSet.empty, HashSet.empty[Int], TreeSet.empty[Int])
    property("an empty Set should have size 0") {
      forAll(examples) { set => set.size should be(0) }
    }
    property("invoking head on an empty set should produce NoSuchElementException") {
      forAll(examples) { set => a[NoSuchElementException] should be thrownBy {
        set.head
      }
      }
    }
  }
  ~~~

+ FeatureSpec

  ~~~ scala
  import org.scalatest._

  class TVSet {
    private var on: Boolean = false
    def isOn: Boolean = on
    def pressPowerButton() {
      on = !on
    }
  }

  class TVSetSpec extends FeatureSpec with GivenWhenThen {
    info("As a TV set owner")
    info ("I want to be able to turn the TV on and off")
    info ("So I can watch TV when I want")
    info ("And save energy when I'm not watching TV")
    feature("TV power button") {
      scenario("User presses power button when TV is off") {
        Given("a TV set that is switched off")
        val tv = new TVSet assert (!tv.isOn)

        When("the power button is pressed")
        tv.pressPowerButton()

        Then("the TV should switch on")
        assert (tv.isOn)
      }
      scenario("User presses power button when TV is on") {
        Given("a TV set that is switched on")
        val tv = new TVSet tv.pressPowerButton()
        assert (tv.isOn)

        When("the power button is pressed")
        tv.pressPowerButton()

        Then("the TV should switch off")
        assert (!tv.isOn)
      }
    }
  }
  ~~~

+ RefSpec (JVM only)

  ~~~ scala
  import org.scalatest.refspec.RefSpec

  class SetSpec extends RefSpec {

    object `A Set` {

      object `when empty` {
        def `should have size 0` {
          assert(Set.empty.size == 0)
        }

        def `should produce NoSuchElementException when head is invoked` {
          assertThrows[NoSuchElementException] {
            Set.empty.head
          }
        }
      }

    }

  }
  ~~~

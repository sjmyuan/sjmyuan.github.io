---
title: How to publish Scala library to Central Repository?
tags:
- Scala
date: 2021-12-01 23:00 +0800
---
# What is Central Repository?
The [Central Repository](https://central.sonatype.org/) is the largest collection of Java and other open-source components. It provides the easiest way to access and distribute your software components to millions of developers. It is the default repository for Apache Maven, SBT, and other build systems and can be easily used from Apache Ant/Ivy, Gradle, and many other tools.

# What is Sonatype?
[Sonatype](https://www.sonatype.com/) is a Fulton, Maryland-based cybersecurity company helping enterprises get precise intelligence about open source components and software security

# What is the relationship between Sonatype and Central Repository?
Sonatype is the steward, maintainer, and financial sponsor of the Central Repository and is determined to continue to provide it to the community of consumers and providers of components.
# How to prepare the publishing?
1. [Sign up Sonatype JIRA account](https://issues.sonatype.org/secure/Signup!default.jspa)
2. [Create a New project ticket](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)

   ![ticket form](https://images.shangjiaming.com/screenshot-1638363380158.png)

	 The most important field is **Group Id**, you need to own the corresponding domain, if you don't own a domain, you can also use the GitHub account **io.github.(account)**, here is the feedback I got from Sonatype

	 ![sonatype reply](https://images.shangjiaming.com/screenshot-1638363844535.png)

# How to publish Scala artifact?
## Add plugins

Add the following line in `project/plugins.sbt`

```scala
addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "3.9.9") // manage sonatype related stuff
addSbtPlugin("com.jsuereth" % "sbt-pgp" % "2.0.1") // manage gpg related stuff
addSbtPlugin("com.github.sbt" % "sbt-release" % "1.1.0") // manage the release process
```
## Rewrite release task

Add the following line in `build.sbt`
```scala
    releaseProcess := Seq[ReleaseStep](
      checkSnapshotDependencies,
      inquireVersions,
      runClean,
      runTest,
      setReleaseVersion,
      commitReleaseVersion,
      tagRelease,
      releaseStepCommandAndRemaining("+publishSigned"),
      releaseStepCommand("sonatypeBundleRelease"),
      setNextVersion,
      commitNextVersion,
      pushChanges
    )
```
## Generate GPG key
1. Run the following command to generate key
   ```sh
   gpg --gen-key
   ```
	 ![generate key process](https://images.shangjiaming.com/screenshot-1638362859300.png)
3. Send the key to the server supported by Sonatype
   ```sh
	 gpg --keyserver keyserver.ubuntu.com --send-keys <gpg key>
	 ```
## Add required credentials
1. Create a `sonatype.sbt` in `~/.sbt/<version>/sonatype.sbt`
2. Add the following line in `sonatype.sbt`
   ```scala
   credentials += Credentials("Sonatype Nexus Repository Manager",
         "s01.oss.sonatype.org",
         "<Sonatype JIRA Account Name>",
         "<Sonatype JIRA Account Password>")
   credentials += Credentials(
     "<gpg key uid>",
     "gpg",
     "<gpg key>", // key identifier
     "ignored" // this field is ignored; passwords are supplied by pinentry
   )
   ```
## Run release command

Run `release` in sbt
```sh
sbt> release
```
![chose version screen shot](https://images.shangjiaming.com/screenshot-1638364157515.png)

Then you can view the library in Central Repository

![lib in central repository](https://images.shangjiaming.com/screenshot-1638370171682.png)

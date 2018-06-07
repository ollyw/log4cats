# log4cats [![Build Status](https://travis-ci.org/ChristopherDavenport/log4cats.svg?branch=master)](https://travis-ci.org/ChristopherDavenport/log4cats) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.chrisdavenport/log4cats_2.12/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.chrisdavenport/log4cats_2.12)

## Quick Start

To use log4cats in an existing SBT project with Scala 2.11 or a later version, add the following dependency to your
`build.sbt`:

```scala
libraryDependencies ++= Seq(
  "io.chrisdavenport" %% "log4cats-core"    % "<version>",  // Only if you want to Support Any Backend
  "io.chrisdavenport" %% "log4cats-log4s"   % "<version>",  // For Log4s Support
  "io.chrisdavenport" %% "log4cats-scribe"  % "<version>",   // For Scribe Support
)
```

## Examples

```scala
import io.chrisdavenport.log4cats.Logger
import io.chrisdavenport.log4cats.slf4j.Slf4jLogger
import cats.effect.Sync
import cats.implicits._

object MyThing {
  // Impure But What 90% of Folks I know do with log4s
  implicit def unsafeLogger[F[_]: Sync] = Slf4jLogger.unsafeCreate[F]

  // Arbitrary Local Function Declaration
  def doSomething[F[_]: Sync]: F[Unit] =
    Logger[F].info("Logging Start Something") *>
    Sync[F].delay(println("I could be doing anything"))
      .attempt.flatMap{
        case Left(e) => Logger[F].error(e)("Something Went Wrong")
        case Right(_) => Sync[F].pure(())
      }

  def safelyDoThings[F[_]: Sync]: F[Unit] = for {
    logger <- Slf4jLogger.create[F]
    _ <- logger.info("Logging at start of safelyDoThings")
    something <- Sync[F].delay(println("I could do anything"))
      .onError{case e => logger.error(e)("Something Went Wrong in safelyDoThings")}
    _ <- logger.info("Logging at end of safelyDoThings")
  } yield something

  def passForEasierUse[F[_]: Sync: Logger] = for {
    _ <- Logger[F].info("Logging at start of passForEasierUse")
    something <- Sync[F].delay(println("I could do anything"))
      .onError{case e => Logger[F].error(e)("Something Went Wrong in passForEasierUse")}
    _ <- Logger[F].info("Logging at end of passForEasierUse")
  } yield something
}
```
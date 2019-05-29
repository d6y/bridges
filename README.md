# Bridges

Generate bindings for Scala types in other programming languages.

Copyright 2017-2018 Dave Gurnell. Licensed [Apache 2.0][license].

Maintainers:

 - Dave Gurnell (Flow and Typescript support)
 - Pere Villega (Elm support)

[![Build Status](https://travis-ci.org/davegurnell/bridges.svg?branch=develop)](https://travis-ci.org/davegurnell/bridges)
[![Coverage status](https://img.shields.io/codecov/c/github/davegurnell/bridges/develop.svg)](https://codecov.io/github/davegurnell/bridges)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.davegurnell/bridges_2.12/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.davegurnell/bridges_2.12)

## Getting Started

Grab the code by adding the following to your `build.sbt`:

~~~scala
libraryDependencies += "com.davegurnell" %% "bridges" % "<<VERSION>>"
~~~

## Synopsis

### Render Typescript/Flow Declarations for Scala ADTs

First, create a simple data model:

~~~scala
final case class Color(red: Int, green: Int, blue: Int)

sealed abstract class Shape extends Product with Serializable
final case class Circle(radius: Double, color: Color) extends Shape
final case class Rectangle(width: Double, height: Double, color: Color) extends Shape

~~~

The `bridges.typescript` and `bridges.flow`
packages expose similar APIs for encoding ADTs as structural types.

The `decl[Foo]` method creates an object representing
a type declaration for `Foo`.

The `render(...)` method writes a list of declaration objects
to a String in the relevant target language.
Here's an example for Typescript:

~~~scala
import bridges.typescript._
import bridges.typescript.syntax._

Typescript.render(List(
  decl[Color],
  decl[Circle],
  decl[Rectangle],
  decl[Shape]
))
// res1: String =
// export type Color = {
//   red: number,
//   green: number,
//   blue: number
// };
//
// export type Circle = {
//   radius: number,
//   color: Color
// };
//
// export type Rectangle = {
//   width: number,
//   height: number,
//   color: Color
// };
//
// export type Shape =
//   {
//     type: "Circle",
//     radius: number,
//     color: Color
//   } |
//   {
//     type: "Rectangle",
//     width: number,
//     height: number,
//     color: Color
//   };

~~~

Simply replace the imports to target Flow instead:

~~~scala
import bridges.flow._
import bridges.flow.syntax._

Flow.render(List(
  decl[Color],
  decl[Circle],
  decl[Rectangle],
  decl[Shape]
))
~~~

The `syntax` packages also provide a simple DSL
for defining structural types directly:

~~~scala
import bridges.typescript._
import bridges.typescript.TsType._
import bridges.typescript.syntax._

val logMessage: TsDecl =
  decl("LogMessage")(struct(
    "level" -> union(lit("error"), lit("warning")),
    text    -> Str
  )

Typescript.render(logMessage)
// res0: String =
// export interface LogMessage {
//   level: "error" | "warning";
//   text: string;
// };
~~~

You can even create generic types using the DSL,
which is something the shapeless derivation currently can't handle:

~~~scala
import bridges.typescript._
import bridges.typescript.TsType._
import bridges.typescript.syntax._

val pair: TsDecl =
  decl("Pair", "A", "B")(struct(
    "head" -> Ref("A"),
    "tail" -> Ref("B"),
  )

Typescript.render(pair)
// res0: String =
// export interface Pair<A, B> {
//   head: A;
//   tail: B;
// };
~~~

### Render Elm Definitions and JSON Codecs for Scala ADTs

The `bridges.elm` package generates type definitions and JSON codecs
for [Elm](https://elm-lang.org):

 - `Elm.render` generates a type definition for a list of declarations;
 - `Elm.jsonDecoder` generates an Elm `Decode.Decoder`
 - `Elm.jsonEncoder` generates an Elm `Encoder`
 - `Elm.buildFile` returns a pair (String, String)
   where the `first element` is the name of an Elm file
   and the `second element` is the content for that file,
   containing type and JSON codec definitions.

To avoid circular references, `Elm.buildFile` can generate
a single file for a list of declarations.

If you want to use any of the JSON encoders or decoders generated by the project
you need to do the following:

 - Your Elm project *must* include the
   `NoRedInk/elm-decode-pipeline` dependency
 - We assume any non-primitive type in your ADT will be
   generated by Bridges in the same module as the current ADT,
   to be able to define the right imports for them.
 - If your ADT contains a sum type,
   the generated json must distinguish between alternatives
   using a field named `type` that encodes
   the name of the product instance as a `String`.
   If you use [Circe](https://circe.github.io/circe/),
   see [this link](https://github.com/circe/circe/pull/429).

NOTE: automatic encoder and decoder generation doesn't work for types with Generics. You will need to manually create your own.

### Working with Refined types

If you are interested in this library
you are most likely using [Refined](https://github.com/fthomas/refined).

We have provided a default encoder for refined types. It will defaults
to the basic type associated with the refined type. For example:

* For `Int Refined Greater[W.`6`.T]` we treat the type as an `Int`
* For `String Refined Size[ClosedOpen[W.`1`.T, W.`100`.T]]` we treat the type as a `String`
* etc

This should cover most (if not all) use cases of refined types when converting to other languages.
You can still override the default encoder with your own higher-precedence encoder.

You can see an example of this in tests for class `ClassWithRefinedType`.

## Developing

- Development should be completed on feature branches and submitted by PR to master.
- Commits to master are published as snapshot builds.
- Tags of the form `x.y.z` are published as release builds.

[license]: http://www.apache.org/licenses/LICENSE-2.0

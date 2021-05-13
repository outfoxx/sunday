---
title: Sunday - Generator - Why? RAML
---
# Why RAML?

RAML is a succint and extensible languagee for modeling REST APIs. While we chose to focus on supporting RAML because it's a great language in general, the main reason was due to it's extensibility.

* [Learn more about RAML](http://raml.org)
* [RAML Specification](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md)

## Extensibility

RAML's extensions are defined in RAML which makes them easy to understand and most importantly they are validated during authorship. This ensures authors can get immediate feedback for invalid Sunday's extensions; the same feedback they get when authoring standard RAML features.

RAML's extensibility makes it perfect for Sunday which defines numerous extensions to make features like inhertance, nested types, and declaring and using problem response definitions easy to implement.

* [Sunday's RAML extensions](../raml-extensions).



## What about OpenAPI?
OpenAPI 3.0 has limitations for defining extensions and can at times be very verbose. These issues and more are solved in OpenAPI 3.1. Once parsers and other tools implement the 3.1 specification the Sunday generator plans to support this language as well.


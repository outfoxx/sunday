---
title: Sunday - About
---
# What is Sunday?

Sunday is a code generator and family of client libraries for implementing _generated_ REST clients and server stubs from [RAML](generator/why-raml.md).

## Features

Sunday focuses on implmenting a consistent set of advanced REST features across client & server libraries and languages.

The key features Sunday provides:

* [Detailed Error Responses via Problems](problems.md)
* [Efficient Notifications via Server-Sent Events](server-sent-events.md)
* [Dynamic Content-Type Negotiation](dynamic-content-type-negotiation.md)
* [Alternative Request/Response Encodings](alternative-request-response-encodings.md)
* [Date/Time Data Types](date-time-data-types.md)
* [Simple & Beautiful Code](client-code.md)

## Generator

The binding technology for Sunday is the [Generator](generator/index.md) which generates clients for all of Sunday's client library implementations.

In addition, the generator also supports generating server and client interfaces for JAX-RS that are customizable for a number of implementations (e.g [Quarkus](https://quarkus.io)).

## Client Libraries

Sunday provides dedicated client libraries for the following languages:

* [Swift client library for Apple platforms](sunday-swift/index.md) 
	* macOS , iOS , iPadOS , WatchOS
* [Kotlin client library for JVM platforms](sunday-kotlin/index.md)
	* Java, Android
* [TypeScript client library for Web platforms](sunday-js/index.md)
	* Browsers (Chrome , Safari , Firefox)

### Concurrency

The client library implementations use the latest concurrency features of their respective languages to make their usage effeciant and enjoyable. Concurrency featuers like `async` / `await` and coroutines are supported where available.


## Client Generation

The Sunday code generator generates client APIs targeting the following client libraries:

* [Sunday](#)
* [JAX-RS](https://en.wikipedia.org/wiki/Jakarta_RESTful_Web_Services) ([Java](https://www.java.com))

## Server Generation

The Sunday code generator generates server stubs for the following standards based libraries:

* [JAX-RS](https://en.wikipedia.org/wiki/Jakarta_RESTful_Web_Services) ([Java](https://www.java.com))

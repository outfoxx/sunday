---
title: Sunday - Features - Simple & Beautiful Code
---
# Simple & Beautiful Code

Sunday client libraries are designed to enable Sunday's [Generator](generator/index.md) to produce client APIs that are comprised of a small amount of code that is efficient and eschews dynamic code (i.e. reflection, proxies, etc.) for static generation.

## Reduced Regeneration

The generated clients delegate the bulk of their service method implementations to request factory interfaces to simplify the generated client code. This simplification of client code goes beyond making the code simple and easy to read, it reduces the chances that clients need to be regenerated when the client libraries are upggraded for bug fixes releases.

## Concurrency

The generated client APIs are built on the client implementations that use the latest concurrency features of their respective languages. Concurrency features like `async` / `await` and coroutines are supported when available. For example, Kotlin service methods are generated as `suspend` functions with regular return types and TypeScript service methods are generated as `async` functions with return values wrapped in a `Promise`.

## Poetry

Sunday Generator aims to produce code that looks like it was written by you. To that end it does not use traditional template based code generation, instead it uses "Poet" code generation libraries that produce great looking, simple and well formatted code. For generating each client or server it uses either [KotlinPoet](https://square.github.io/kotlinpoet), [JavaPoet](https://github.com/square/javapoet), [SwiftPoet](https://github.com/outfoxx/SwiftPoet), or [TypeScriptPoet](https://github.com/outfoxx/TypeScriptPoet). 

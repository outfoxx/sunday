---
title: Sunday - Generator - Target - Kotlin - JAX-RS
---
# Kotlin - JAX-RS

Details the specific features of the Kotlin/JAX-RS code generation target.

## Generated Types

Types generated for the JAX-RS target are the same as those generated for all Kotlin target (see [Generated Types for Kotlin Targets](target-kotlin-common-features.md#generated-types))

## Generated Services

Services are generated as interfaces with a service method for each API endpoint. The interfaces are adorned with all of the JAX-RS and Jackson annotations required to invoke the remote services methods.

!!! example "Example Generated Service"
    
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    mediaType: [application/json]
    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API  {

      @GET
      @Path("items/{id}")
      public fun fetchItem(@PathParam("id") id: String): Response

    }
    ```

### Client / Server Support

The Kotlin/JAX-RS target supports generating JAX-RS compatible interfaces that can be used as client or server stubs and is controlled by the [Generation Mode Option](#generator-options).

#### Server Stubs

In server mode, the generated JAX-RS interfaces are designed to favor supporting the full range of JAX-RS capabilities over convenience. For example, all server service methods return a `javax.ws.rs.Response` instead of returning the type defined in the RAML definition. This allows doing advanced tasks like adding headers to the response.

#### Client Stubs

Some JAX-RS client implementations (e.g. [RESTEasy](https://resteasy.github.io)) support generating client proxies from JAX-RS style interfaces. The Sunday generator supports this feature by tweaking the standard generated interface to be more usable in a client environment.

Client mode service methods will return a type compatible with that declared in the RAML definitions. This is different from server stubs that always return `javax.ws.rs.Response`.

!!! note "Asynchronous Methods Not Supported"
    Service methods flagged as asynchronous are generated as regular service methods in client mode. See [Reactive Support](#reactive-support) for a solution to generating asynchronous methods.

### Server-Sent Events

Service methods flagged as producing `Server-Sent Events` are generated with extra parameters to support producing or consuming events.

Server method stubs include the extra JAX-RS parameters `Sse` and `SseEventSink` to support producing events.

??? example "Example SSE Service Method Generation (Server)"

    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    uses:
      sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    mediaType: [application/json]
    types:

    /events/{deviceId}:
      get:
        displayName: listenToEvents
        (sunday.eventSource): true
        responses:
          200:
            body:
              text/event-stream:
                type: object

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API {

      @GET
      @Path(value = "/events/{deviceId}")
      public fun listenToEvents(@PathParam("id") id: String, @Context sse: Sse, @Context sseEvents: SseEventSink): Unit

    }
    ```

Client method stubs return a JAX-RS `SseEventSource` to support consuming events.

??? example "Example SSE Service Method Generation (Client)"

    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    uses:
      sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    mediaType: [application/json]
    types:

    /events/{deviceId}:
      get:
        displayName: listenToEvents
        (sunday.eventSource): true
        responses:
          200:
            body:
              text/event-stream:
                type: object

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API {

      @GET
      @Path(value = "/events/{deviceId}")
      public fun listenToEvents(@PathParam("id") id: String): SseEventSource

    }
    ```

### Reactive Support

Reactive support is enabled by the providing the [Reactive Response Type](#generator-options) generator options. The types _must_ be a generic type that is suported by the user's selected JAX-RS framework.

The only significant change to the generated interface is that the return type is always the provided reactive repsonse type parameterized by the response type declared in the RAML definition.

This applies to both server and client interfaces.

??? example "Example Reactive Service Method Generation with `CompletableFuture` (Server)"
   
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    mediaType: [application/json]
    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API {

      @GET
      @Path("items/{id}")
      public fun fetchItem(@PathParam("id") id: String): CompletableFuture<Response>

    }
    ```

??? example "Example Reactive Service Method Generation with `CompletableFuture` (Client)"
   
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    mediaType: [application/json]
    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API  {

      @GET
      @Path("items/{id}")
      public fun fetchItem(@PathParam("id") id: String): CompletableFuture<Item>

    }
    ```

### Kotlin Coroutines Support

Some JAX-RS implementations (e.g. [Quarkus](https://quarkus.io)) support Kotlin coroutine methods as JAX-RS resource methods. Sunday generates coroutine service methods when the [Enable Coroutine Support](#generator-options) generator option is enabled. 

When coroutines are enabled service methods are generated as suspendable (e.g. they include the `suspend` modifier).

This applies to both server and client interfaces.

??? example "Example Coroutine Service Method Generation (Server)"
   
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    mediaType: [application/json]
    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API  {

      @GET
      @Path("items/{id}")
      public suspend fun fetchItem(@PathParam("id") id: String): Response

    }
    ```

??? example "Example Coroutine Service Method Generation (Client)"
   
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API
    mediaType: [application/json]
    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item

    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API  {

      @GET
      @Path("items/{id}")
      public suspend fun fetchItem(@PathParam("id") id: String): Item

    }
    ```

### Explicit Security Support

When the [Enable Explicit Security Parameters](#generator-options) generator option is enabled any security parmeters defined in a security scheme associated with a service method are prepended to the service methods standard paraemters.

This feature allows users to explicitly handle (in server stubs) or explicitly specify (in client methods) security parameters via the interface rather than via an interceptor.

??? example "Explicit Security Parameters Generation (Server)"
    __RAML API Definition__
    ```yaml
    #%RAML 1.0
    title: Test API

    securitySchemes:
      bearer:
        type: Pass Through
        describedBy:
          headers:
            Authorization:
              description: JWT token
              type: string
              pattern: "Bearer [0-9a-zA-Z+/-_=]+"

    securedBy: [bearer]

    types:
      
      Item:
        type: object

    /items/{id}:
      get:
        displayName: fetchItem
        responses:
          200:
            body: Item
    ```

    __Generated Service Interface__
    ```kotlin
    @Produces(value = ["application/json"])
    @Consumes(value = ["application/json"])
    public interface API  {

      @GET
      @Path("items/{id}")
      public fun fetchItem(@HeaderParam(value = "Authorization") bearerAuthorization: String, @PathParam("id") id: String): Response

    }
    ```    

## Generator Options

In addition to the [options supported by all Kotlin code generations targets](target-kotlin-common-features.md#generator-options), this target also supports the following options:

__Generation Mode__
:   Choose whether the to target client or server generation.

    | CLI Option        | Gradle Plugin Properties  | Type    | Allowed Values     | Default |
    | ----------------- | ------------------------- | ------- | ------------------ | ------- |
	| `-mode`           | `mode`     				| string  | `client`, `server` | None    |


__Enable Coroutine Support__
:   Generate suspendable service methods for supporting coroutines.

    | CLI Option        | Gradle Plugin Properties  | Type    | Allowed Values    | Default |
    | ----------------- | ------------------------- | ------- | ----------------- | ------- |
	| `-coroutines`     | `coroutines`              | boolean | true, false 	  | false    |


__Reactive Response Type__
:   Specifies the generic result type for reactive service methods.

    | CLI Option        | Gradle Plugin Properties  | Type    | Allowed Values        | Default |
    | ----------------- | ------------------------- | ------- | --------------------- | ------- |
	| `-reactive`       | `reactiveResponseType`    | string  | Any Valid Kotlin Type | None    |

	!!! note
		 This option also acts as a flag that enables reactive method generation.


__Enable Explicit Security Parameters__
:   Includes security parameters in generated service methods.

    | CLI Option                      | Gradle Plugin Properties     | Type    | Allowed Values        | Default |
    | ------------------------------- | ---------------------------- | ------- | --------------------- | ------- |
	| `-explicit-security-parameters` | `explicitSecurityParameters` | boolean | Any Valid Kotlin Type | None    |

---
title: Sunday - Generator - Target - Kotlin - Sunday
---
# Kotlin - Sunday

Details the specific features of the Kotlin/Sunday code generation target.

## Generated Types

Types generated for the Sunday target are the same as those generated for all Kotlin targets (see [Generated Types for Kotlin Targets](target-kotlin-common-features.md#generated-types))

## Generated Services

Services are generated as classes with a service method for each API endpoint. The services use a `RequestFactory` interface dependency that performs the network requests and adapts the results.  

???+ example "Example Generated Service"
	
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

	__Generated Service Class__
	```kotlin
	public class API(
	  public val requestFactory: RequestFactory,
      public val defaultContentTypes: List<MediaType> = listOf(),
      public val defaultAcceptTypes: List<MediaType> = listOf(MediaType.JSON)
	) {

	  public suspend fun fetchItem(id: String): Item = this.requestFactory.result(
        method = Method.Get,
        pathTemplate = "/items/{id}",
        pathParameters = mapOf(
          "id" to id
        )
        acceptTypes = this.defaultAcceptTypes
      )

	}
	```

!!! note Required Coroutines Support

	Kotlin Sunday clients are generated with service methods that are suspendable (i.e. including the `suspend` modifier). The generation of suspendable methods requires that they be called in a coroutine context and thus Kotlin coroutine support.

### Server-Sent Event Methods

Service methods that are marked with either of Sunday's Server-Sent Events annotations are generated to return values that allow subscribing to events.

#### EventSource

Service methods marked with the [EventSource](../raml-extensions#eventsource-server-sent-events) annotation are generated returning an `EventSource`. 

Sunday's `EventSource` is a work-alike to the [EventSource Web API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource).

??? example "Example Server-Sent Events Service Method Generation (EventSource)"
	
	__RAML API Definition__

	```yaml
	#%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	mediaType: [application/json]
	types:

	  Event:
	  	type: object
	  
	  CallEvent:
	    type: Event

	  MessageEvent:
	    type: Event	    

	/events/{deviceId}:
	  get:
	    displayName: listenToEvents
	    (sunday.eventSource): true
	    responses:
	      200:
	      	body:
	      	  text/event-stream:
	            type: (CallEvent | MessageEvent)

	```

	__Generated Service Class__

	```kotlin
	public class API(
	  val requestFactory: RequestFactory,
      val defaultContentTypes: List<MediaType> = listOf(),
      val defaultAcceptTypes: List<MediaType> = listOf(MediaType.JSON)
	) {

	  public suspend fun listenToEvents(deviceId: String): EventSource =
	    this.requestFactory.eventSource(
          method = Method.Get,
          pathTemplate = "/events/{deviceId}",
          pathParameters = mapOf(
            "id" to id
          )
          acceptTypes = [MediaType.EventStream]
        )

	}
	```

#### EventStream

Service methods marked with the [EventStream](../raml-extensions#eventstream-server-sent-events) annotation return a Kotlin `Flow<T>` that is parameterized to the type of event(s) the method produces.

[Learn about Kotlin coroutines Flow](https://kotlinlang.org/docs/flow.html)

??? example "Example Server-Sent Events Service Method Generation (EventStream)"

	__RAML API Definition__

	```yaml
	#%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	mediaType: [application/json]
	types:

	  Event:
	  	type: object
	  
	  CallEvent:
	    type: Event

	  MessageEvent:
	    type: Event	    

	/events/{deviceId}:
	  get:
	    displayName: listenToEvents
	    (sunday.eventStream): discriminated
	    responses:
	      200:
	      	body:
	      	  text/event-stream:
	            type: (CallEvent | MessageEvent)

	```

	__Generated Service Class__

	```kotlin
	class API(
	  val requestFactory: RequestFactory,
      val defaultContentTypes: List<MediaType> = listOf(),
      val defaultAcceptTypes: List<MediaType> = listOf(MediaType.JSON)
	) {

	  fun listenToEvents(deviceId: String): Flow<Event> =
	  	this.requestFactory.eventStream(
          method = Method.Get,
          pathTemplate = "/events/{deviceId}",
          pathParameters = mapOf(
            "id" to id
          )
          acceptTypes = [MediaType.EventStream]
        )

	}
	```

### Request/Response Only

Service methods can be flagged as "request" or "response" only using Sunday's RAML extension annoations.  These flags will generate service methods that return a platform specific request or response instead of the value defined by the RAML API definition.

!!! note
	Platform requests in [Sunday (Kotlin)](../../sunday-kotlin) library are `okhttp3.Request` instances and platform responses are `okhttp3.Response`; this is due to the library being built upon [okhttp](https://square.github.io/okhttp).


#### Request Only

Request only service methods return a platform specific request object _without_ executing the remote request. The user can execute the request as is or alter the request first and then execute it.

??? example "Example Request Only Service Method Generation"
	
	__RAML API Definition__
	```yaml
	#%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	mediaType: [application/json]
	types:
	  
	  Item:
	    type: object

	/items/{id}:
	  get:
	    displayName: fetchItem
	    (sunday.requestOnly): true
	    responses:
	      200:
	        body: Item

	```

	__Generated Service Class__
	```kotlin
	public class API(
	  public val requestFactory: RequestFactory,
      public val defaultContentTypes: List<MediaType> = listOf(),
      public val defaultAcceptTypes: List<MediaType> = listOf(MediaType.JSON)
	) {

	  public suspend fun fetchItem(id: String): Request =
	  	this.requestFactory.request(
          method = Method.Get,
          pathTemplate = "/items/{id}",
          pathParameters = mapOf(
            "id" to id
          )
          acceptTypes = this.defaultAcceptTypes
        )

	}
	```

#### Response Only

Response only service methods return a platform specific response object _after_ executing the remote request. The user can implement custom parsing and handling of the resposne as needed.

??? example "Example Response Only Service Method Generation"
	
	__RAML API Definition__
	```yaml
	#%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	mediaType: [application/json]
	types:
	  
	  Item:
	    type: object

	/items/{id}:
	  get:
	    displayName: fetchItem
	    (sunday.responseOnly): true
	    responses:
	      200:
	        body: Item

	```

	__Generated Service Class__
	```kotlin
	public class API(
	  public val requestFactory: RequestFactory,
      public val defaultContentTypes: List<MediaType> = listOf(),
      public val defaultAcceptTypes: List<MediaType> = listOf(MediaType.JSON)
	) {

	  public suspend fun fetchItem(id: String): Response =
	  	this.requestFactory.response(
          method = Method.Get,
          pathTemplate = "/items/{id}",
          pathParameters = mapOf(
            "id" to id
          )
          acceptTypes = this.defaultAcceptTypes
        )

	}
	```

### Default Media Types

The constructors of the generated client services allow specifying the default support and ordering of content & accept types. The order, and elements, of these default lists determines how Sunday encoding requests and decoding responses.

#### Request Encoding with Content Types

Each client service constructor includes a parameter `defaultContentTypes`. The items in this list controls which encodings Sunday will support for encoding requests and the order controls the preference order for selecting the specific request encoding.

The items provided in `defaultContentTypes` are matched to the encodings supported by the `requestFactory` to choose which encoding will be used to encode request content. Together this allows complete control over request encoding by configuration.

!!! example "Support JSON & CBOR, Preferring JSON"

	Construct the service supporting both JSON and CBOR; *__preferring JSON__*.

	```kotlin
	val api = API(requestFactory, defaultContentTypes = listOf(MediaType.JSON, MediaType.CBOR))
	```

	If the `requestFactory` supports __JSON__, then __JSON__ will be the default encoding used to encode requests; otherwise the CBOR will be used.

!!! example "Support JSON & CBOR, Preferring CBOR"

	Construct the service supporting both JSON and CBOR; *__preferring CBOR__*.

	```kotlin
	val api = API(requestFactory, defaultContentTypes = listOf(MediaType.CBOR, MediaType.JSON))
	```

	If the `requestFactory` supports __CBOR__, then __CBOR__ will be the default encoding used to encode requests; otherwise the JSON will be used.

#### Response Encoding with Accept Types

Sunday will include an `Accept` header equivalent, in elements and order, to that provided in the service constructor's `defaultAcceptTypes` parameter. When the server supports content negotiation using the `Accept` header it will encode responses using the first supproted media type given.


!!! example "Accept JSON & CBOR, Preferring CBOR"

	Construct the service supporting both JSON and CBOR as response encodings; *__preferring CBOR__*.

	```kotlin
	val api = API(requestFactory, defaultAcceptTypes = listOf(MediaType.CBOR, MediaType.JSON))
	```


## Generator Options

In addition to the [options supported by all Kotlin code generations targets](../target-kotlin-common-features#generator-options), this target also supports the following options:

None
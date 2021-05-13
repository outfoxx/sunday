---
title: Sunday - Generator - Targets - Swift - Sunday
---
# Swift - Sunday

Details the specific features of the Swift/Sunday code generation target.

## Generated Types

Types generated for the Sunday target are the same as those generated for all Swift targets (see [Generated Types for Swift Targets](target-swift-common-features.md#generated-types))

## Generated Services

Services are generated as classes with a service method for each API endpoint. The services use a `RequestFactory` protocol dependency that performs the network requests and adapts the results.  

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

  	__Generated Swift/Sunday Service__
  	```swift
  	public class API {

        public let requestFactory: RequestFactory
        public let defaultContentTypes: [MediaType]
        public let defaultAcceptTypes: [MediaType]

        public init(
          requestFactory: RequestFactory,
          defaultContentTypes: [MediaType] = [.json],
          defaultAcceptTypes: [MediaType] = [.json]
        ) {
          self.requestFactory = requestFactory
          self.defaultContentTypes = defaultContentTypes
          self.defaultAcceptTypes = defaultAcceptTypes
        }


        func fetchItem(id: String) -> RequestResultPublisher<Item> {
          return self.requestFactory.result(
            method: .get,
            pathTemplate: "/items/{id}",
            pathParameters: [
            	"id": id
            ],
            queryParameters: nil,
            body: nil as Empty?,
            contentTypes: nil,
            acceptTypes: self.defaultAcceptTypes,
            headers: nil
          )
        }

  	}
  	```

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
    ```swift
    public class API {

      public let requestFactory: RequestFactory
      public let defaultContentTypes: [MediaType]
      public let defaultAcceptTypes: [MediaType]

      public init(
        requestFactory: RequestFactory,
        defaultContentTypes: [MediaType] = [],
        defaultAcceptTypes: [MediaType] = [.json]
      ) {
        self.requestFactory = requestFactory
        self.defaultContentTypes = defaultContentTypes
        self.defaultAcceptTypes = defaultAcceptTypes
      }

      func listenToEvents(deviceId: String) -> EventSource {
        return self.requestFactory.eventSource(
          method: .get,
          pathTemplate: "/events/{deviceId}",
          pathParameters: [
            "deviceId": deviceId
          ],
          queryParameters: nil,
          body: nil as Empty?,
          contentTypes: nil,
          acceptTypes: [.eventStream],
          headers: nil
        )}

    }
    ```

#### EventStream

Service methods marked with the [EventStream](../raml-extensions#eventstream-server-sent-events) annotation return a Combine `AnyPublisher<T, E>` that is parameterized to the type of event(s) the method produces.

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
    ```swift
    public class API {

      public let requestFactory: RequestFactory
      public let defaultContentTypes: [MediaType]
      public let defaultAcceptTypes: [MediaType]

      public init(
        requestFactory: RequestFactory,
        defaultContentTypes: [MediaType] = [],
        defaultAcceptTypes: [MediaType] = [.json]
      ) {
        self.requestFactory = requestFactory
        self.defaultContentTypes = defaultContentTypes
        self.defaultAcceptTypes = defaultAcceptTypes
      }

      func listenToEvents(deviceId: String) -> RequestEventPublisher<Event> {
        return self.requestFactory.eventStream(
          method: .get,
          pathTemplate: "/events/{deviceId}",
          pathParameters: [
            "deviceId": deviceId
          ],
          queryParameters: nil,
          body: nil as Empty?,
          contentTypes: nil,
          acceptTypes: [.eventStream],
          headers: nil,
          eventTypes: [
            "phone": Phone.self, 
            "tablet": Tablet.self
          ]
        )}

    }
    ```

### Request/Response Only

Service methods can be flagged as "request" or "response" only using Sunday's RAML extension annoations.  These flags will generate service methods that return a platform specific request or response instead of the value defined by the RAML API definition.

!!! note
    Platform requests in [Sunday (Swift)](../../sunday-swift) library are `Foundation.URL` instances and platform responses are the tuple `(response: HTTPURLResponse, data: Data?)`.


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
    ```swift
    public class API {

      public let requestFactory: RequestFactory
      public let defaultContentTypes: [MediaType]
      public let defaultAcceptTypes: [MediaType]

      public init(
        requestFactory: RequestFactory,
        defaultContentTypes: [MediaType] = [],
        defaultAcceptTypes: [MediaType] = [.json]
      ) {
        self.requestFactory = requestFactory
        self.defaultContentTypes = defaultContentTypes
        self.defaultAcceptTypes = defaultAcceptTypes
      }

      func fetchItem(id: String) -> AnyPublisher<URLRequest, Error> {
        return self.requestFactory.request(
          method: .get,
          pathTemplate: "/items/{id}",
          pathParameters: [
            "id": id
          ],
          queryParameters: nil,
          body: nil as Empty?,
          contentTypes: nil,
          acceptTypes: self.defaultAcceptTypes,
          headers: nil
        )
      }

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
    ```swift
    public class API {

      public let requestFactory: RequestFactory
      public let defaultContentTypes: [MediaType]
      public let defaultAcceptTypes: [MediaType]

      public init(
        requestFactory: RequestFactory,
        defaultContentTypes: [MediaType] = [],
        defaultAcceptTypes: [MediaType] = [.json]
      ) {
        self.requestFactory = requestFactory
        self.defaultContentTypes = defaultContentTypes
        self.defaultAcceptTypes = defaultAcceptTypes
      }
      
      func fetchItem(id: String) -> AnyPublisher<(response: HTTPURLResponse, data: Data?), Error> {
        return self.requestFactory.response(
          method: .get,
          pathTemplate: "/items/{id}",
          pathParameters: [
            "id": id
          ],
          queryParameters: nil,
          body: nil as Empty?,
          contentTypes: nil,
          acceptTypes: self.defaultAcceptTypes,
          headers: nil
        )
      }

    }
    ```

### Default Media Types

The initializers of the generated client services allow specifying the default support and ordering of content & accept types. The order, and elements, of these default lists determines how Sunday encoding requests and decoding responses.

#### Request Encoding with Content Types

Each client service initializer includes a parameter `defaultContentTypes`. The items in this list controls which encodings Sunday will support for encoding requests and the order controls the preference order for selecting the specific request encoding.

The items provided in `defaultContentTypes` are matched to the encodings supported by the `requestFactory` to choose which encoding will be used to encode request content. Together this allows complete control over request encoding by configuration.

!!! example "Support JSON & CBOR, Preferring JSON"

    Construct the service supporting both JSON and CBOR; *__preferring JSON__*.

    ```swift
    val api = API(requestFactory: requestFactory, defaultContentTypes: [.json, .cbor])
    ```

    If the `requestFactory` supports __JSON__, then __JSON__ will be the default encoding used to encode requests; otherwise the CBOR will be used.

!!! example "Support JSON & CBOR, Preferring CBOR"

    Construct the service supporting both JSON and CBOR; *__preferring CBOR__*.

    ```swift
    val api = API(requestFactory: requestFactory, defaultContentTypes: [.cbor, .json])
    ```

    If the `requestFactory` supports __CBOR__, then __CBOR__ will be the default encoding used to encode requests; otherwise the JSON will be used.

#### Response Encoding with Accept Types

Sunday will include an `Accept` header equivalent, in elements and order, to that provided in the service initializer's `defaultAcceptTypes` parameter. When the server supports content negotiation using the `Accept` header it will encode responses using the first supproted media type given.


!!! example "Accept JSON & CBOR, Preferring CBOR"

    Construct the service supporting both JSON and CBOR as response encodings; *__preferring CBOR__*.

    ```swift
    val api = API(requestFactory: requestFactory, defaultAcceptTypes: [.cbor, .json])
    ```

## Generator Options

In addition to the [options supported by all Swift code generations targets](../target-swift-common-features#generator-options), this target also supports the following options:

None

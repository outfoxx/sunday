---
title: Sunday - Generator - Targets - TypeScript - Sunday
---
# TypeScript - Sunday

Details the specific features of the TypeScript/Sunday code generation target.

## Generated Types

Types generated for the Sunday target are the same as those generated for all TypeScript targets (see [Generated Types for TypeScript Targets](target-typescript-common-features.md#generated-types))

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

	__Generated TypeScript/Sunday Service__
	```typescript
    export class API {

      constructor(public requestFactory: RequestFactory,
          public defaultContentTypes: Array<MediaType> = [],
          public defaultAcceptTypes: Array<MediaType> = [MediaType.JSON]) {
      }

      fetchItem(id: string): Observable<Item> {
        return this.requestFactory.result(
            {
              method: 'GET',
              pathTemplate: '/items/{id}',
              pathParameters: {
                id
              },
              acceptTypes: [MediaType.],
            },
            fetchItemReturnType
        );
      }

    }


    const fetchItemReturnType: AnyType = [Item];
	```

### Server-Sent Event Methods

Service methods that are marked with either of Sunday's Server-Sent Events annotations are generated to return values that allow subscribing to events.

#### EventSource

Service methods marked with the [EventSource](raml-extensions.md#eventsource-server-sent-events) annotation are generated returning an [EventSource Web API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource). 

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
    ```typescript
    export class API {

      constructor(public requestFactory: RequestFactory,
          public defaultContentTypes: Array<MediaType> = [],
          public defaultAcceptTypes: Array<MediaType> = [MediaType.JSON]) {
      }

      listenToEvents(deviceId: string): EventSource {
        return this.requestFactory.events(
            {
              method: 'GET',
              pathTemplate: '/events/{deviceId}',
              pathParameters: {
              	deviceId,
              },
              acceptTypes: [MediaType.EventStream]
            }
        );
      }

    }
    ```

#### EventStream

Service methods marked with the [EventStream](raml-extensions.md#eventstream-server-sent-events) annotation return a RxJS `Observable<T>` that is parameterized to the type of event(s) the method produces.

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
    ```typescript
    import {Device} from './device';
    import {Phone} from './phone';
    import {Tablet} from './tablet';

    export class API {

      constructor(public requestFactory: RequestFactory,
          public defaultContentTypes: Array<MediaType> = [],
          public defaultAcceptTypes: Array<MediaType> = [MediaType.JSON]) {
      }

      listenToEvents(id: string): Observable<Device> {
        const eventTypes: EventTypes<Device> = {
          'phone' : [Phone], 
          'tablet' : [Tablet]
        };
        return this.requestFactory.events<Device>(
            {
              method: 'GET',
              pathTemplate: '/events/{deviceId}',
              pathParameters: {
              	id,
              },
              acceptTypes: [MediaType.EventStream]
            },
            eventTypes
        );
      }

    }
    ```

### Request/Response Only

Service methods can be flagged as "request" or "response" only using Sunday's RAML extension annoations.  These flags will generate service methods that return a platform specific request or response instead of the value defined by the RAML API definition.

!!! note
    Platform requests in [Sunday (TypeScript)](../sunday-js/index.md) library are `Request` objects and platform responses are `Response` objects both types are from the [Fetch Web API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).


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
    ```typescript
    export class API {

      constructor(public requestFactory: RequestFactory,
          public defaultContentTypes: Array<MediaType> = [],
          public defaultAcceptTypes: Array<MediaType> = [MediaType.JSON]) {
      }

      fetchItem(id: string): Observable<Request> {
        return this.requestFactory.request(
            {
              method: 'GET',
              pathTemplate: '/items/{id}',
              pathParameters: {
                id
              },
              acceptTypes: this.defaultAcceptTypes,
            },
        );
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
    ```typescript
    export class API {

      constructor(public requestFactory: RequestFactory,
          public defaultContentTypes: Array<MediaType> = [],
          public defaultAcceptTypes: Array<MediaType> = [MediaType.JSON]) {
      }

      fetchItem(id: string): Observable<Response> {
        return this.requestFactory.response(
            {
              method: 'GET',
              pathTemplate: '/items/{id}',
              pathParameters: {
                id
              },
              acceptTypes: this.defaultAcceptTypes,
            },
        );
      }

    }
    ```

### Default Media Types

The constructors of the generated client services allow specifying the default support and ordering of content & accept types. The order, and elements, of these default lists determines how Sunday encoding requests and decoding responses.

#### Request Encoding with Content Types

Each client service constructor includes a parameter `defaultContentTypes`. The items in this list controls which encodings Sunday will support for encoding requests and the order controls the preference order for selecting the specific request encoding.

The items provided in `defaultContentTypes` are matched to the encodings supported by the `requestFactory` to choose which encoding will be used to encode request content. Together this allows complete control over request encoding by configuration.

!!! example "Support JSON & CBOR, Preferring JSON"

	Construct the service supporting both JSON and CBOR; *__preferring JSON__*.

	```typescript
	val api = new API(requestFactory, [MediaType.JSON, MediaType.CBOR], defaultAcceptTypes)
	```

	If the `requestFactory` supports __JSON__, then __JSON__ will be the default encoding used to encode requests; otherwise the CBOR will be used.

!!! example "Support JSON & CBOR, Preferring CBOR"

	Construct the service supporting both JSON and CBOR; *__preferring CBOR__*.

	```typescript
	val api = new API(requestFactory, [MediaType.CBOR, MediaType.JSON], defaultAcceptTypes)
	```

	If the `requestFactory` supports __CBOR__, then __CBOR__ will be the default encoding used to encode requests; otherwise the JSON will be used.

#### Response Encoding with Accept Types

Sunday will include an `Accept` header equivalent, in elements and order, to that provided in the service constructor's `defaultAcceptTypes` parameter. When the server supports content negotiation using the `Accept` header it will encode responses using the first supproted media type given.


!!! example "Accept JSON & CBOR, Preferring CBOR"

	Construct the service supporting both JSON and CBOR as response encodings; *__preferring CBOR__*.

	```typescript
	val api = new API(requestFactory, defaultContentTypes, [MediaType.CBOR, MediaType.JSON])
	```

## Generator Options

In addition to the [options supported by all TypeScript code generations targets](target-typescript-common-features.md#generator-options), this target also supports the following options:

None

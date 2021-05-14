---
title: Sunday - Generator - RAML Extensions
---
# RAML Extensions

The Sunday Generator defines numerous RAML extension annotations to control code generation.

## Annotation Definitions

RAML requires that annotations be defined before that can be used. Sunday provides a RAML library with all of the extension annotations.

The latest Sunday extension definition library is always available at:

	https://outfoxx.github.io/sunday-generator/sunday.raml

To use a specific version of the Sunday extension definition library, include the verision in the URL like the following:

	https://outfoxx.github.io/sunday-generator/1.0.0/sunday.raml



!!! example "Usage of Sunday Definitions Library"
	Any RAML file that wishes to reference the Sunday definitions must include a `uses` reference to the library file and prefix all references to Sunday annotations.
	```yaml
	#%RAML 1.0
	title: Test API
	uses:
  	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml 1️⃣
  	/test:
  	  get:
	    (sunday.asynchronous): true 2️⃣
	    response:
	      200:
	        body: object
	```

	1. Import the Sunday definitions library. This example uses the name `sunday` but you are free to choose any name.
	1. Prefix all references to Sunday's annotations using the chosen import name.


## Extensions Reference

A complete reference of the defined annotation extensions is provided below.

!!! note "RAML Examples"
	In an effort to be succinct, the RAML examples provided in this reference are not always complete and may forego some of the required RAML properties that API definitions require.

### General Annotations

Annotations shared by all code generation targets.

---

#### Service Group
##### Marks resources as belonging to a named group.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `group` | `string` | None | Resource |

Allows grouping specific resources (aka endpoints) by name. A distinct service is generated for each named group and then another service is generated for the default unnamed group.

!!! example
    The following RAML would create three service interfaces or classes. One for each named group (e.g. `AccountAPI` and `AdminAPI`) as well
    as one for the unnamed group (e.g. `API`).
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	/auth:
	  post:
	    displayName: login
	/devices:
	  (sunday.group): account
	  get:
	  	displayName: listDevices
	/tenants:
	  (sunday.group): admin
	  get:
	    displayName: listTenants
	```

---

#### Nested Type 
##### Requests that a type be nested inside another type.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `nested` | [Nested](#nested) | None | Type |

Generates the type nested into another specified type with an explicitly specified name.

!!! attention
	This annotation is ignored for any language that does not support nested types. 

!!! Example
	
	Generate an enum `Kind` inside the type `Device`

	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	types:

	  Device:
	    properties:
	      kind: Device-Kind
	      name: string

	  Device-Kind:
	    (nested):
          enclosedIn: Device
          name: Kind
        type: string
        enum: [phone, tablet]    
	```

	Results in the following generated code examples
	=== "Kotlin"
		```kotlin
		public class Device(
		  val kind: Kind,
		  val name: String,
		) {

		  public enum class Kind {
		  	Phone,
		  	Tablet
		  }

		}
		```
	=== "Swift"
		```swift
		public class Device {

          public enum Kind : String {
          	case phone = "phone"
          	case tablet = "tablet"
          }

		  public let kind: Kind
		  public let name: String

		  public init(kind: Kind, name: String) {
		  	self.kind = kind
		  	self.name = name
		  }

		}
		```



---

#### Patchable Type Flag
##### Flags a type as patchable.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `patchable` | `boolean` | None | Type |

A type flagged as patchable has special support for `PATCH` operations. When used in a `PATCH` method, a nested `Patch` type is generated that supports simple JSON patch/merge style updates.

!!! example

	Generate `Update` type with patch support.

	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    types:
      Update:
	    type: object
	    (sunday.patchable): true
	    properties:
	      string: string
	      int: integer
	      bool: boolean
	```

	Results in the following generated code examples
	=== "Kotlin"
		```kotlin

        public class Update(
          public val string: String,
          public val int: Int,
          public val bool: Boolean
        ) {

          public fun patch(source: ObjectNode): Patch = Patch(
            if (source.has("string")) Optional.ofNullable(string) else null,
            if (source.has("int")) Optional.ofNullable(int) else null,
            if (source.has("bool")) Optional.ofNullable(bool) else null
          )

          @JsonInclude(JsonInclude.Include.NON_NULL)
          public data class Patch(
            public val string: Optional<String>? = null,
            public val int: Optional<Int>? = null,
            public val bool: Optional<Boolean>? = null
          )
        }
        
		```
	=== "Swift"
		```swift

        public class Update {

          public let string: String
          public let int: Int
          public let bool: Bool

          public init(
            string: String,
            int: Int,
            bool: Bool
          ) {
            self.string = string
            self.int = int
            self.bool = bool
          }

          func patch(source: [String : Any]) -> Patch {
            return Patch(string: source.keys.contains("string") ? Optional.some(string) : nil,
                  int: source.keys.contains("int") ? Optional.some(int) : nil,
                  bool: source.keys.contains("bool") ? Optional.some(bool) : nil)
          }

          public struct Patch {

            let string: String?
            let int: Int?
            let bool: Bool?

            init(
              string: String?,
              int: Int?,
              bool: Bool?
            ) {
              self.string = string
              self.int = int
              self.bool = bool
            }

          }

        }
		```

---

### Polymorphic Annotations

Annotations for generating polymorphic encoding/decoding of types.

---

#### External Discriminator Property Name
##### Specifies external property to be used as discriminator.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `externalDiscriminator` | `string` | None | Type |


Specifies the property name of a discriminator property in the owning object that discriminators the type flagged as externally discriminated.

!!! note
	This annotation must be used in conjuction with [Externally Discriminated Type Flag](#externally-discriminated-type-flag)


!!! example

    ```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    types:
	  Parent:
	    type: object
	    discriminator: type
	    (sunday.externallyDiscriminated): true
	    properties:
	      type: string

	  Child1:
	    type: Parent
	    properties:
	      value?: string

	  Child2:
	    type: Parent
	    discriminatorValue: child2
	    properties:
	      value?: string

	  Test:
	    type: object
	    properties:
	      value:
	        type: Parent
	        (sunday.externalDiscriminator): objectType
	      objectType: string
    ```

	This allows implementation of the standard JSON structure:
	```json
	{
		"objectType": "child2",
		"object": {
			"value": "test"
		}
	}
	```

---

#### Externally Discriminated Type Flag
##### Flags a type hierarchy as discriminated by an external property.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `externallyDiscriminated` | `boolean` | None | Type |


Polymorphic types that are marked as externally discriminated are allowed to be used as the type of another object's properties where the discriminator is also located in the other object's propereties.

!!! note
	This annotation must be used in conjuction with [External Discriminator Property Name](#external-discriminator-property-name)

!!! example

    ```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    types:
	  Parent:
	    type: object
	    discriminator: type
	    (sunday.externallyDiscriminated): true
	    properties:
	      type: string

	  Child1:
	    type: Parent
	    properties:
	      value?: string

	  Child2:
	    type: Parent
	    discriminatorValue: child2
	    properties:
	      value?: string

	  Test:
	    type: object
	    properties:
	      parent:
	        type: Parent
	        (sunday.externalDiscriminator): parentType
	      parentType: string
    ```

	This allows implementation of the standard JSON structure:
	```json
	{
		"objectType": "child2",
		"object": {
			"value": "test"
		}
	}
	```

---

### Problem Annotations

Annotations for declaring and referencing Problem definitions.


---

#### Problem Base Type URI
##### Specifies the base URI for Problem types
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `problemBaseUri` | `string` | None | API, Overlay |

Problem declarations include a requried `type` property that must be a URI. Sunday's succint declaration method generates the full type property value by resolving it against the base URI specified with this annotation.

Any variables defined in the URI are resolved from the `problemUriParams` values.

###### Relative URIs

If a relative URI value is specified for `problemBaseUri`, it is first resolved against the API's `baseUri`.

!!! note "Fallback"
	If this annotation is not specified the `baseUri` & `baseUriParameters` defined for the API are used instead.



---

#### Problem Base Type URI Parameters
##### Specifies parameter values for the base URI for Problem types
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `problemUriParams` | `object` | None | API, Overlay |

Values for resolving resolving parameters specified in the `problemBaseUri` annotation.

This annotation works similarly to the way `baseUriParameters` works for `baseUri`.

!!! note
	If this annotation is not specified the `baseUriParameters` defined for the API is used instead.


---

#### Problem Declarations
##### Declare a set of reusable problems
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `problemTypes` | [ResponseProblemTypes](#responseproblemtypes) | None | API, Extension, Library, Overlay |

Declares global problem types to be used throughout the API. The value of this annotation is a map of problem type ids to problem type details.

!!! example "Problem Declaration"

	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    
    (sunday.problemTypes):
      id_in_use:
        status: 400
        title: ID Already In Use
        detail: The provided ID is already being used.
        custom: 1️⃣
          suggestedAlternative: string 2️⃣
	```
	
	1. Custom properties can be specified to provide context specific details
	2. Each custom property is a the property name as the key and the type as the value

###### Type URI

Problem details require a `type` property that must be a URI. Sunday generates the a valid type URI by resolving the type id (`id_in_use` in the example above) against the `problemBaseUri` if it's specified. If `problemBaseUri` is not specified then `baseUri` is used instead.


---

#### Problems References
##### Add referenced problems to methdod responses
| Annotation | Type | Modifiers | Method |
| -- | -- | -- | -- |
| `problems` | [ResponseProblemCodes](#responseproblemcodes) | None | Method |

Adds each referenced problem to the range of responses the method is able to generate.

!!! example

	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    
    (sunday.problemTypes):
      device_not_found: 1️⃣
        status: 404
        title: Device Not Found
        detail: The requested device could not be found.

    /devices/{id}:
      get:
        displayName: fetchDevice
        responses:
          200:
            body: Device
      	(sunday.problems): [device_not_found] 2️⃣
	```

	1. Problems are declared via `problemTypes`
	2. Problems are referenced via `problems`

---

### Kotlin Annotations

Annotations for Kotlin code generation.


---

#### Kotlin Package Name
##### Specifies default package name for generated model types and services.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `kotlinPackage` | `string` | `client`, `server` | API, Extension, Library, Overlay, Type |

Specifies the default package name that Kotlin model types and service types are generated into. When more specific package name annotations are not provided (e.g `kotlinModelPackage`) the package name specified by this annotation is used.

###### Modifiers

Two modifiers are supported for targeting only client or server generation.

`kokotlinPackage:client`
:   Specifies the default package name when generating clients.

`kokotlinPackage:server`
:   Specifies the default package name when generating servers.

!!! note "Fallback"
	When a `kotlinPackage:client` or `kotlinPackage:server` annotation is specificed and the generator is in an alternate mode, the generator falls back to using the unmodified `kotlinPackage` annotation.

!!! example "Examples"
	=== "Default"	
		Generate types and services into the `com.example` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
		(sunday.kotlinPackage): com.example
		```
	=== "Client"
		Generate client types and services into the `com.example.client` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.kotlinPackage:client): com.example.client
		```
	=== "Server"
		Generate server types and services into the `com.example.server` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.kotlinPackage:server): com.example.server
		```
	=== "Fallback"
		Generate server types and services into the `com.example.server` package and generate client types and services into the `com.example` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.kotlinPackage): com.example
		(sunday.kotlinPackage:server): com.example.server
		```


---

#### Kotlin Model Package Name
##### Specifies default package name for generated model types.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `kotlinModelPackage` | `string` | `client`, `server` | API, Extension, Library, Overlay, Type |

Specifies the default package name that Kotlin model types are generated into. When this annotation is not specified the value of `kotlinPackage` is used, if specified.

###### Modifiers

Two modifiers are supported for targeting only client or server generation.

`kotlinModelPackage:client`
:   Specifies the default package name when generating clients.

`kotlinModelPackage:server`
:   Specifies the default package name when generating servers.

!!! note "Fallback"
	When a `kotlinModelPackage:client` or `kotlinModelPackage:server` annotation is specificed and the generator is in an alternate mode, the generator falls back to using the unmodified `kotlinModelPackage` annotation.

!!! example "Examples"
	=== "Default"	
		Generate model types into the `com.example.model` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
		(sunday.kotlinModelPackage): com.example.model
		```
	=== "Client"
		Generate client model types into the `com.example.client.model` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		(sunday.kotlinModelPackage:client): com.example.client.model
		```
	=== "Server"
		Generate server model types into the `com.example.server.model` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		(sunday.kotlinModelPackage:server): com.example.server.model
		```
	=== "Fallback"
		Generate server model types into the `com.example.server.model` package and generate client model types into the `com.example.model` package.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.kotlinModelPackage): com.example.model
		(sunday.kotlinModelPackage:server): com.example.server.model
		```
	=== "Mixed Fallback"
		Generate server model types into the `com.example.server.model` package and generate client model and service types into the `com.example` package using both `kotlinModelPackage` and `kotlinPackage` annotations.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.kotlinPackage): com.example
		(sunday.kotlinModelPackage:server): com.example.server.model
		```


---

#### Kotlin Type Override
##### Specifies existing type name.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `kotlinType` | `string` | `client`, `server` | Type |

Specifies the existing type name that should be used for the annotated type.

!!! note "Fallback"
	When a `kotlinType:client` or `kotlinType:server` annotation is specificed and the generator is in an alternate mode, the generator falls back to using the unmodified `kotlinType` annotation.

!!! example "Examples"
	=== "Default"	
		Use the standard `java.net.URI` type for the RAML type defined as `URI`
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
		types:
			URI:
			  type: string
			  (sunday.kotlinType): java.net.URI			
		```
	=== "Client"
		Use the standard `java.net.URI` type for the RAML type defined as `URI` when generating clients.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		types:
			URI:
			  type: string
			  (sunday.kotlinType:client): java.net.URI
		```
	=== "Server"
		Use the standard `java.net.URI` type for the RAML type defined as `URI` when generating servers.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		types:
			URI:
			  type: string
			  (sunday.kotlinType:server): java.net.URI
		```
	=== "Fallback"
		Use the standard `java.net.URI` type for the RAML type defined as `URI` when generating clients and use the standard `java.net.URL` when generating servers.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		types:
			URI:
			  type: string
			  (sunday.kotlinType): java.net.URI
			  (sunday.kotlinType:server): java.net.URL
		```


---

#### Kotlin Implementation Code
##### Specifies code implementing a computed property.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `kotlinImplementation` | [TypeImplementation](#typeimplementation) | `client`, `server` | Property |

Specifies Kotlin code that implements a computed class property.

!!! note "Fallback"
	When a `kotlinImplementation:client` or `kotlinImplementation:server` annotation is specificed and the generator is in an alternate mode, the generator falls back to using the unmodified `kotlinImplementation` annotation.

!!! example "Examples"
	=== "Default"	
		Use standard arithmatic for implementing the computed `xSquared` property.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
		types:
		  MyObject:
		    properties:
		      x: number		    
		      xSquared:
		        type: number
		        (sunday.kotlinImplementation):
		          code: x * x
		```
	=== "Client"
		Use standard arithmatic for implementing the computed `xSquared` property when generating clients. When generating servers a standard property will be generated.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		types:
		  MyObject:
		    properties:
		      x: number		    
		      xSquared:
		        type: number
		        (sunday.kotlinImplementation:client):
		          code: x * x
		```
	=== "Server"
		Use standard arithmatic for implementing the computed `xSquared` property when generating servers. When generating clients a standard property will be generated.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml	
		types:
		  MyObject:
		    properties:
		      x: number		    
		      xSquared:
		        type: number
		        (sunday.kotlinImplementation:server):
		          code: x * x
		```
	=== "Fallback"
		Use standard arithmatic for implementing the computed `xSquared` property when generating clients. Use the mathematic `pow` function when generating servers.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		types:
		  MyObject:
		    properties:
		      x: number		    
		      xSquared:
		        type: number
		        (sunday.kotlinImplementation):
		          code: x.pow(2)
		        (sunday.kotlinImplementation:client):
		          code: x * x
		```

---


### TypeScript Annotations

Annotations for TypeScript code generation.


---

#### TypeScript Module Name
##### Specifies default module name for generated model types and services.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `typeScriptModule` | `string` | None | API, Extension, Library, Overlay, Type |

Specifies the default module name that TypeScript model types and service types are generated into. When more specific module name annotations are not provided (e.g `typeScriptModelModule`) the module name specified by this annotation is used.

!!! example "Example"
	Generate model types and services into the `test/example` module.
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	(sunday.typeScriptModule): test/example
	```


---

#### TypeScript Model Module Name
##### Specifies default module name for generated model types.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `typeScriptModelModule` | `string` | None | API, Extension, Library, Overlay, Type |

Specifies the default module name that TypeScript model types are generated into. When this annotation is not specified the value of `typeScriptModule` is used, if specified.

!!! example "Example"
	=== "Default"
		Generate model types into the `test/example/model` module.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
		(sunday.typeScriptModelModule): test/example/model
		```
	=== "Mixed"
		Generate model types into the `test/example/model` module and generate service types into the `test/example` module using both `typeScriptModelModule` and `typeScriptModule` annotations.
		```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml		
		(sunday.typeScriptModule): test/example
		(sunday.typeScriptModelModule): test/example/model
		```	


---

#### TypeScript Type Override
##### Specifies existing type name.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `typeScriptType` | `string` | None | Type |

Specifies the existing type name that should be used for the annotated type.

!!! example "Example"
	Use the standard JavaScript `URL` type for the RAML type defined as `URL`
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	types:
	  URL:
        type: string
        (sunday.typeScriptType): URL
	```


---

#### TypeScript Implementation Code
##### Specifies code implementing a computed property.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `typeScriptImplementation` | [TypeImplementation](#typeimplementation) | None | Property |

Specifies TypeScript code that implements a computed class property.

!!! example "Example"
	Use standard arithmatic for implementing the computed `xSquared` property.
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	types:
	  MyObject:
	    properties:
	      x: number		    
	      xSquared:
	        type: number
	        (sunday.typeScriptImplementation):
	          code: x * x
	```

---


### Swift Annotations

Annotations for Swift code generation.


---

#### Swift Type Override
##### Specifies existing type name.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `swiftType` | `string` | None | Type |

Specifies the existing type name that should be used for the annotated type.

!!! example "Example"
	Use the standard type `Foundation.URL` type for the RAML type defined as `URL`
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	types:
	  URL:
        type: string
        (sunday.swiftType): Foundation.URL
	```


---

#### Swift Implementation Code
##### Specifies code implementing a computed property.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `swiftImplementation` | [TypeImplementation](#typeimplementation) | None | Property |

Specifies Swift code that implements a computed class property.

!!! example "Example"
	Use standard arithmatic for implementing the computed `xSquared` property.
	```yaml
	%RAML 1.0
	title: Test API
	uses:
	  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	types:
	  MyObject:
	    properties:
	      x: number		    
	      xSquared:
	        type: number
	        (sunday.swiftImplementation):
	          code: x * x
	```

---

### Sunday Annotations

Annotations for generating Sunday clients.


---

#### EventSource (Server-Sent Events)
##### Flag method as returning a raw EventSource
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `eventSource` | `boolean` | None | Method |

Flags the method as returning a raw `EventSource` (or equivalent platform specific type).

!!! note
	Any method using this annotations should also have the `text/event-stream` response content-type.


---

#### EventStream (Server-Sent Events)
##### Flags method as event stream and specifies the type
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `eventStream` | [EventStreamType](#eventstreamtype) | None | Method |

Specifies that the method should return a reactive stream of events and the method of determining what types of events the method will return.

###### Methods

The value of `eventStream` determines how the generator determines what type of events the method will return.

The following methods are supported:

`discriminated`
:   The method body's type should be a RAML union of possible event types. Sunday will use each type's discriminator value to map event type names to concrete value types. If a type has not explicitly set its `discriminatorValue` property the RAML type name will be used.
	
	!!! example
		Generate a `connectEvents` service method that will produce either `Opened` or `Closed` event values. Using the discriminator of each specified type, events with the name `opened` will be mapped to the `Opened` type and events with the name `closed` to the `Closed` type.
	    ```yaml
		%RAML 1.0
		title: Test API
		uses:
		  sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
	    
	    types:
	      Opened:
	      	discriminator: opened
	      	id: string

	      Closed:
	      	discriminator: closed
	      	id: string

	    /events:
	      get:
	      	displayName: connectEvents
	 	    (sunday.eventStream): discriminator
	        responses:
	     	  200:
	     	    body: (Opened | Closed)
	    ```


!!! note
	Any method using this annotations should also have the `text/event-stream` response content-type.


---

#### Request Only Flag
##### Flag method as returning a raw request value
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `requestOnly` | `boolean` | None | Method |

Generates a method that does not call the remote resource endpoint. Instead the method will return a raw request value that can be used to call the resource endpoint independently.



---

#### Response Only Flag
##### Flag method as returning a raw response value
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `responseOnly` | `boolean` | None | Method |

Generates a method that calls the remove resource endpoint but skips all attempts to negotiate content type and deserialize a concreate value. Instead a raw response type is returned that can be used to perform custom handling of the response.

---

### JAX-RS Annotations

Annotations for generating JAX-RS client or server stub interfaces.


---

#### Asynchronous Flag
##### Flags the method as a JAX-RS asynchronous method.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `asynchronous` | `boolean` | None | Method |

Generates a JAX-RS asynchronous method.

For example, in the standard implementation the generated method will have the required `@Context AsyncResponse` parameter added.


---

#### SSE Flag
##### Flags the method as a JAX-RS Server-Sent Events method.
| Annotation | Type | Modifiers | Targets |
| -- | -- | -- | -- |
| `sse` | `boolean` | None | Method |

Generates a JAX-RS Server-Sent Events method.

For example, in the standard implementation the generated method will have the required `@Context Sse` and `@Context SseEventSink` parameters added.

---

### Annotation Type Definitions

---

#### ResponseProblemTypes:
```yaml
type: object
properties:
  /^[a-z0-9_]+$/: ResponseProblemType
```

---

#### ResponseProblemType:
```yaml
type: object
properties:
  status:
    type: integer
    minimum: 100
    maximum: 599
  title:
    type: string
  detail:
    type: string
  custom?: object
additionalProperties: false
```

---

#### ResponseProblemCodes:
```yaml
type: array
items: ResponseProblemCode
```

---

#### ResponseProblemCode:
```yaml
type: string
pattern: '^[a-z0-9_]+$'
```

---

#### TypeImplementation

```yaml
type: object
properties:
  code: string
  parameters:
    type: array
    items:
      type: object
      properties:
        type:
          type: string
          default: Literal
          enum:
            - Type
            - Literal
            - String
        value:
          type: string
```

---

#### Nested
```yaml
type: object
properties:
  enclosedIn: string
  name: string
```

---

#### EventStreamType
```yaml
type: string
enum: [discriminated]
```

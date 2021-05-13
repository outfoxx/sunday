---
title: Sunday - Generator - Targets - Common Kotlin Target Features
---
# Common Kotlin Target Features

The following features are supported by all Kotlin code generation targets.

## Generated Types

### Scalars

RAML built-in scalar types are mapped according to the following table.

| RAML Type | Kotlin Type |
| :-: | :-: |
| any | Any |
| boolean | Boolean |
| number / integer | [Numerics](#numerics) |
| string | String |
| date-only | java.time.LocalDate |
| time-only | java.time.LocalTime |
| datetime-only | java.time.LocalDateTime |
| datetime | java.time.OffsetDateTime |
| file | ByteArray |
| nil | Unit |

#### Numerics

RAML `number` and `integer` types are mapped to one of Kotlins's numeric types based on the `format` facet.
    
|  Format  | Kotlin Type |
| :------: | :--------: |
| `int`    | Int        |
| `int8`   | Byte       |
| `int16`  | Short      |
| `int32`  | Int        |
| `int64`  | Long       |
| `long`   | Long       |
| `float`  | Float      |
| `double` | Double     |

### Objects

For each RAML type that is an `object` or where the root of the inheritance tree is an `object` a Kotlin class or interface is generated.

#### Model Classes

When the generation of model implementations is enabled ([Implement Model Classes - Type Generation Option](#type-generation-options)) a POJO class is generated for each required type that supports copying, equals/hashcode, toString, etc.

??? example "Example Class Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API

    types:
      
      Item:
        type: object
        properties:
          name: String
          value: integer
    ```

    __Generated Kotlin Class__
  	```kotlin
    public class Item(
      public val name: String,
      public val value: Int,
    ) {
      public fun copy(
        name: String? = null,
        value: Int?? = null,
      ) = Test(name ?: this.name, value ?: this.value)

      public override fun hashCode(): Int {
        var result = 1
        result = 31 * result + name.hashCode()
        result = 31 * result + value.hashCode()
        return result
      }

      public override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as Test

        if (name != other.name) return false
        if (value != other.value) return false

        return true
      }

      public override fun toString() = ""${'"'}
      |Test(name='${'$'}name',
      | value='${'$'}value
      ""${'"'}.trimMargin()
    }
  	```

!!! note "Why not data classes?"
    The generator currently does not gnerate Kotlin data classes to ensure easy support for generating type hierarchies. Similar features to those of data classes are generated for each class.

#### Model Interfaces

When the generation of model implementations is disabled ([Implement Model Classes - Type Generation Option](#type-generation-options)) interfaces are generated for each required types.

??? example "Example Interface Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Item:
        type: object
        properties:
          name: String
          value: integer
    ```

    __Generated Kotlin Interface__
  	```kotlin
    public interface Item {
      
      public val name: String
      public val value: Int

    }
  	```

#### Simple Objects

RAML types that are "simple" objects (where no `properties` facet is defined) are mapped to Kotlin maps (i.e. `Map<String, Any>`). They are generated inline and no type aliases are generated.

??? example "Example Simple Object Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Container:
        map: object
    ```

    __Generated Kotlin Class__ (simplified)
    ```kotlin
    public class Container(
      public val map: Map<String, Any>
    )
    ```

#### Pattern Objects

RAML types that only contain pattern properties (i.e. property names defined by regular expressions) are mapped to Kotlin maps with the key as a string and the value as the type specified in the pattern (e.g. `Map<String: Int>`). They are generated inline and no type aliases are generated.

??? example "Example Pattern Object Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      MapOfInts:
        type: object
          //: integer

      Container:
        map: MapOfInts
    ```

    __Generated Kotlin Class__ (simplified)
    ```kotlin
    public class Container(
      public val map: Map<String, Int>
    )
    ```

#### Jackson Annotations

The generator defaults to adding Jackson JSON serialization annotations to the generated classes and interfaces. This enables advanced JSON features (e.g. polymorphism, explicit naming, etc.) to be supported without editing the generated types or registering mixins.

??? example "Example Jackson Annotations Generation (Explicit Names)"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Item:
        type: object
        properties:
          simple-name: string
          value: integer
    ```

    __Generated Kotlin Class__
    ```kotlin
    public class Item(
      @JacksonProperty("simple-name")
      public val simpleName: String
      public val value: Int
    )
    ```

??? example "Example Jackson Annotations Generation (Polymorphism)"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
    
      Device:
        type: object
        discriminator: type
        properties:
          type: string
          name: string
      
      Phone:
        type: Device
        discriminatorValue: phone
        properties:
          hasGPS: boolean
      
      Tablet:
        type: Device
        discriminatorValue: tablet
        properties:
          isKeyboardAttached: boolean
    ```

    __Generated Kotlin Classes__
    ```kotlin
    @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME,
      include = JsonTypeInfo.As.EXISTING_PROPERTY,
      property = "type"
    )
    @JsonSubTypes(value = [
      JsonSubTypes.Type(name = "phone", value = Phone::class),
      JsonSubTypes.Type(name = "tablet", value = Tablet::class)
    ])
    public abstract class Device(
      public val name: String
    ) {
      
      public abstract val type: String

    }
    ```

    ---

    ```kotlin
    public class Phone(
      name: String,
      public val hasGPS: Boolean,
    ) : Device(name) {

      val type: String get() = "phone"

    }
    ```

    ---

    ```kotlin
    public class Tablet(
      type: String,
      public val isKeyboardAttached: Boolean,
    ) : Device() {

      val type: String get() = "tablet"

    }

    ```

!!! note
    The generation of Jackson annotations can be disabled via the [Jackson Annotations - Type Generation Option](#type-generation-options).

!!! danger
    Disabling Jackson will most likely require a custom (de)serialization implementation to support the full feature set.


#### Bean Validation Constraints

The generator defaults to adding Bean Validation Constraints to validate instance data as they are serialized/deserialized.

??? example "Example Validation Constraints Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Item:
        type: object
        properties:
          id:
            type: string
            pattern: [a-zA-Z0-9]+
          value:
            type: integer
            minimum: 5
            maximum: 10
    ```

    __Generated Kotlin Class__
    ```kotlin
    public class Item(
    
      @Pattern("[a-zA-Z0-9]+")
      public val id: String,

      @Min(5) @Max(10)
      public val value: Int,
    )
    ```

!!! note
    The generation of validation constraints can be disabled via the [Validation Constraints - Type Generation Option](#type-generation-options)

### Arrays

RAML array types are mapped to Kotlin's `List` type. They are generated inline and no type aliases are generated.

??? example "Example Array Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
    
      Item:
        type: string

      Items:
        type: array
        items: Item

      Container:
        items: Items
    ```

    __Generated Kotlin Class__ (simplified)
    ```kotlin
    public class Container(
      public val items: List<String>
    )
    ```

### Unions

RAML union types are mapped to the "nearest common ancestor" of all individual types in the union, if one exists, in all other cases the union is mapped to Kotlin's `Any` type.

??? example "Example Union Generation (Common Aancestor)"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Device:
        type: object
        discriminator: type
        properties:
          type: string
          name: string
      
      Phone:
        type: Device
        discriminatorValue: phone
        properties:
          hasGPS: boolean
      
      Tablet:
        type: Device
        discriminatorValue: tablet
        properties:
          isKeyboardAttached: boolean

      UnionOfAllDevices:
        type: (Phone | Tablet)

      Container:
        type: object
        properties:
          device: UnionOfAllDevices

    ```

    1. Inline union of types with a common ancestor.


    __Generated Kotlin Class__ (simplified)
    ```kotlin
    public class Container(
      public val device: Device
    )

    ```

## Generated Problem Types

For each problem defined & referenced using Sunday's [problem annotations](../raml-extensions#problem-annotations), a Kotlin `Exception` class is generated. Generating exceptions allows servers to throw a specific problem and clients to catch specific problems.

Custom properties can be defined for problem types and they are added as simple Kotlin properties on the generated exception class.

??? example "Example Problem Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    uses:
        sunday: https://outfoxx.github.io/sunday-generator/sunday.raml
    types:

    (sunday.problemTypes):
      invalid_id:
        status: 400
        title: Invalid Id
        detail: The id contains one or more invalid characters
        custom:
          offendingId: string
    ```

    __Generated Problem Exception Class__
    ```kotlin
    public class InvalidIdProblem(
      @JsonProperty(value = "offending_id")
      public val offendingId: String,
      instance: URI? = null,
      cause: ThrowableProblem? = null
    ) : AbstractThrowableProblem(TYPE_URI, "Invalid Id", Status.BAD_REQUEST,
        "The id contains one or more invalid characters.", instance, cause) {
      public override fun getCause(): Exceptional? = super.cause
    
      public companion object {
        public const val TYPE: String = "http://example.com/invalid_id"

        public val TYPE_URI: URI = URI(TYPE)
      }
    }
    ```

## Generator Options

In addition to the [options supported by all code generations targets](../target-common-features#generator-options), this target also supports the following options:

__Default Package Name__
:   Specifies the default Kotlin package name for model types and service types.

    | CLI Option        | Gradle Plugin Properties  | Type    | Default |
    | ----------------- | ------------------------- | ------- | ------- |
	| `-pkg`            | `pkgName`     			| string  | None    |


__Default Model Package Name__
:   Specifies the default Kotlin package name for model types.

    | CLI Option        | Gradle Plugin Properties  | Type    | Default |
    | ----------------- | ------------------------- | ------- | ------- |
	| `-model-pkg`      | `modelPkgName`     		| string  | None    |

	!!! note
		If not specified the value specified in `-pkg` is used.


__Default Service Package Name__
:   Specifies the default Kotlin package name for service types.

    | CLI Option        | Gradle Plugin Properties  | Type    | Default |
    | ----------------- | ------------------------- | ------- | ------- |
	| `-service-pkg`    | `servicePkgName`     		| string  | None    |

	!!! note
		If not specified the value specified in `-pkg` is used.

<a name="type-generation-options"></a>

__Type Generation Options__
:   Enable/Disable Kotlin type generation options.

	##### Supported Options
	
	__Implement Model Classes__
	:   Enables or Disables implementation of model classes
	
	__Jackson Annotations__
	:   Enables or Disables generation of Jackson annotations 

	__Validation Constraints__
	:   Enables or Disables generation of bean validation annotations

    | CLI Option                        | Gradle Plugin Properties       | Type    | Default |
    | --------------------------------- | ------------------------------ | ------- | ------- |
	| `-disable implement-model` 		| `disableModelImplementations`  | boolean | enabled |
	| `-disable jackson-annotations`    | `disableJacksonAnnotations`    | boolean | enabled |
	| `-disable validation-constraints` | `disableValidationConstraints` | boolean | enabled |

---
title: Sunday - Generator - Targets - Common Swift Target Features
---
# Common Swift Target Features

The following features are supported by all Swift code generation targets.

## Generated Types

### Scalars

RAML built-in scalar types are mapped according to the following table.

| RAML Type | Swift Type |
| :-: | :-: |
| any | Any |
| boolean | Boolean |
| number / integer | [Numerics](#numerics) |
| string | String |
| date-only | Date |
| time-only | Date |
| datetime-only | Date |
| datetime | Date |
| file | Data |
| nil | Optional.none (aka `nil`) |

#### Numerics

RAML `number` and `integer` types are mapped to one of Swift's numeric types based on the `format` facet.
    
|  Format  | Swift Type |
| :------: | :--------: |
| `int`    | Int        |
| `int8`   | Int8       |
| `int16`  | Int16      |
| `int32`  | Int32      |
| `int64`  | Int64      |
| `long`   | Int64      |
| `float`  | Float      |
| `double` | Double     |

### Objects

For each RAML type that is an `object` or where the root of the inheritance tree is an `object` and contains any defined properties, a Swift class is generated.

Generated classes are immutable and fluent modifier functions are generated for each property defined on the type.

Each generated class implements `Codable` and `CustomDebugStringConvertible`. It's left up to the users to implement other Swift standard protocols like `Equatable` and/or `Hashable`.

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

    __Generated Swift Class__
  	```swift
    public class Item : Codable, CustomDebugStringConvertible {

      public let name: String
      public let value: Int
      public var debugDescription: String {
        return DescriptionBuilder(Test.self)
            .add(name, named: "name")
            .add(value, named: "value")
            .build()
      }

      public init(name: String, value: Int) {
        self.name = name
        self.value = value
      }

      public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.name = try container.decode(String.self, forKey: .name)
        self.value = try container.decode(Int.self, forKey: .value)
      }

      public func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.name, forKey: .name)
        try container.encode(self.value, forKey: .value)
      }

      public func withName(name: String) -> Test {
        return Test(name: name, value: value)
      }

      public func withValue(value: Int) -> Test {
        return Test(name: name, value: value)
      }

      fileprivate enum CodingKeys : String, CodingKey {

        case name = "name"
        case value = "value"

      }

    }
  	```

#### Inherited Objects

RAML types that inherit from a single parent type are generated as a class hierarchy.

As long as each type in the hierarchy only inherits from a single parent type the hierarch can be as complex as the author requires.

When the root of the RAML hierarchy has the `discriminator` facet set, the generated class hierarchy will support polymorphic decoding using a nested reference object named `AnyRef`.


??? example "Example Class Hierarchy Generation"
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

    __Generated Swift Classes__
    ```swift
    public class Device : Codable {

      public var type: String {
        fatalError("abstract type method")
      }
      public let name: String
      public var debugDescription: String {
        return DescriptionBuilder(Device.self)
            .build()
      }

      public init(name: String) {
        self.name = name
      }

      public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.name = try container.decode(String.self, forKey: .name)
      }

      public func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.name, forKey: .name)
      }

      public enum AnyRef : Codable, CustomDebugStringConvertible {

        case phone(Phone)
        case tablet(Tablet)

        public var value: Device {
          switch self {
          case .phone(let value): return value
          case .tablet(let value): return value
          }
        }
        public var debugDescription: String {
          switch self {
          case .phone(let value): return value.debugDescription
          case .tablet(let value): return value.debugDescription
          }
        }

        public init(value: Device) {
          switch value {
          case let value as Phone: self = .phone(value)
          case let value as Tablet: self = .tablet(value)
          default: fatalError("Invalid value type")
          }
        }

        public init(from decoder: Decoder) throws {
          let container = try decoder.container(keyedBy: CodingKeys.self)
          let type = try container.decode(String.self, forKey: CodingKeys.type)
          switch type {
          case "phone": self = .phone(try Phone(from: decoder))
          case "tablet": self = .tablet(try Tablet(from: decoder))
          default:
              throw DecodingError.dataCorruptedError(
                forKey: CodingKeys.type,
                in: container,
                debugDescription: "unsupported value for \"type\""
              )
          }
        }

        public func encode(to encoder: Encoder) throws {
          var container = encoder.container(keyedBy: CodingKeys.self)
          switch self {
          case .phone(let value):
              try container.encode("phone", forKey: .type)
              try value.encode(to: encoder)
          case .tablet(let value):
              try container.encode("tablet", forKey: .type)
              try value.encode(to: encoder)
          }
        }

      }

      fileprivate enum CodingKeys : String, CodingKey {

        case type = "type"

      }

    }
    ```

    ---

    ```swift
    public class Phone : Parent {

      public override var type: String {
        return "phone"
      }
      public let hasGPS: Boolean
      public override var debugDescription: String {
        return DescriptionBuilder(Phone.self)
            .add(type, named: "type")
            .add(name, named: "name")
            .add(hasGPS, named: "hasGPS")
            .build()
      }

      public init(name: String, hasGPS: Boolean) {
        self.hasGPS = hasGPS
        super.init(name: name)
      }

      public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.hasGPS = try container.decode(Boolean.self, forKey: .hasGPS)
        try super.init(from: decoder)
      }

      public override func encode(to encoder: Encoder) throws {
        try super.encode(to: encoder)
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.hasGPS, forKey: .hasGPS)
      }

      public func withName(name: String) -> Phone {
        return Phone(name: name, isKeyboardAttached: isKeyboardAttached)
      }

      public func withHasGPS(hasGPS: Boolean) -> Phone {
        return Phone(name: name, hasGPS: hasGPS)
      }

      fileprivate enum CodingKeys : String, CodingKey {

        case hasGPS = "hasGPS"

      }

    }
    ```

    ---

    ```swift
    public class Tablet : Parent {

      public override var type: String {
        return "tablet"
      }
      public let isKeyboardAttached: Boolean
      public override var debugDescription: String {
        return DescriptionBuilder(Tablet.self)
            .add(type, named: "type")
            .add(name, named: "name")
            .add(isKeyboardAttached, named: "isKeyboardAttached")
            .build()
      }

      public init(name: String, isKeyboardAttached: Boolean) {
        self.isKeyboardAttached = isKeyboardAttached
        super.init(name: name)
      }

      public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.isKeyboardAttached = try container.decode(Boolean.self, forKey: .isKeyboardAttached)
        try super.init(from: decoder)
      }

      public override func encode(to encoder: Encoder) throws {
        try super.encode(to: encoder)
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.isKeyboardAttached, forKey: .isKeyboardAttached)
      }

      public func withName(name: String) -> Tablet {
        return Tablet(name: name, isKeyboardAttached: isKeyboardAttached)
      }

      public func withIsKeyboardAttached(isKeyboardAttached: Boolean) -> Tablet {
        return Tablet(name: name, isKeyboardAttached: isKeyboardAttached)
      }

      fileprivate enum CodingKeys : String, CodingKey {

        case isKeyboardAttached = "isKeyboardAttached"

      }

    }

    ```

    __Use Generated AnyRef to Encode/Decode__
    ```swift
    let decodedDevice = JsonDecoder().decode(Device.AnyRef).value
    let encodedDevice = JsonEncoder().encode(Device.AnyRef(decodedDevice))
    ```


#### Simple Objects

RAML types that are "simple" objects (where no `properties` facet is defined) are mapped to Swift dictionaries (i.e. `[String: Any]`). They are generated inline and no type aliases are generated.

??? example "Example Simple Object Generation"
    __RAML Type Definition__
    ```yaml
    %RAML 1.0
    title: Test API
    types:
      
      Container:
        map: object
    ```

    __Generated Swift Class__ (simplified)
    ```swift
    class Container : Codable {

      let map: [String: Any]

      init(items: [String: Any]) {
        self.map = map
      }

    }
    ```

#### Pattern Objects

RAML types that only contain pattern properties (i.e. property names defined by regular expressions) are mapped to Swift dictionaries with the key as a string and the value as the type specified in the pattern (e.g. `[String: Int]`). They are generated inline and no type aliases are generated.

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

    __Generated Swift Class__ (simplified)
    ```swift
    class Container : Codable {

      let map: [String: Int]

      init(items: [String: Int]) {
        self.map = map
      }

    }
    ```

### Arrays

RAML array types are mapped to Swift's `Array` type. They are generated inline and no type aliases are generated.

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

    __Generated Swift Class__ (simplified)
    ```swift
    class Container : Codable {

      let items: [String]

      init(items: [String]) {
        self.items = items
      }

    }
    ```

### Unions

RAML union types are mapped to the "nearest common ancestor" of all individual types in the union, if one exists, in all other cases the union is mapped to Swit's `Any` type.

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


    __Generated Swift Class__ (simplified)
    ```swift
    public class Container {

      let device: Device

      init(device: Device) {
        self.device = device
      }

    }

    ```

## Generated Problem Types

For each problem defined & referenced using Sunday's Sunday's [problem annotations](raml-extensions.md#problem-annotations), a Swift `Error` class is generated. Generating exceptions allows servers to throw a specific problem and clients to catch specific problems.

Custom properties can be defined for problem types and they are added as simple Swift properties on the generated error class.

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

    __Generated Problem Error Class__
    ```swift
    public class InvalidIdProblem : Problem {

      public static let type: URL = URL(string: "http://example.com/invalid_id")!
      public let offendingId: String
      var description: String {
        return DescriptionBuilder(Self.self)
            .add(type, named: "type")
            .add(title, named: "title")
            .add(status, named: "status")
            .add(detail, named: "detail")
            .add(instance, named: "instance")
            .add(offendingId, named: "offendingId")
            .build()
      }

      init(offendingId: String, instance: URL? = nil) {
        self.offendingId = offendingId
        super.init(type: Self.type, title: "Invalid Id", status: 400,
            detail: "The id contains one or more invalid characters.", instance: instance,
            parameters: nil)
      }

      public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.offendingId = try container.decode(String.self, forKey: CodingKeys.offendingId)
        try super.init(from: decoder)
      }

      public override func encode(to encoder: Encoder) throws {
        try super.encode(to: encoder)
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.offendingId, forKey: CodingKeys.offendingId)
      }

      fileprivate enum CodingKeys : String, CodingKey {

        case offendingId = "offending_id"

      }

    }
    ```

## Generator Options

In addition to the [options supported by all code generations targets](target-common-features.md#generator-options), this target also supports the following options:

__Type Generation Options__
:   Enable/Disable Swift type generation options.

	##### Supported Options
	
	__Add Generated Annotation__
	:   Enables or Disables adding generation annotations to generated types

    | CLI Option                           | Gradle Plugin Properties  | Type    | Default |
    | ------------------------------------ | ------------------------- | ------- | ------- |
	| `-disable add-generation-annotation` |                           | boolean | enabled |

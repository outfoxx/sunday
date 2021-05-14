---
title: Sunday - Generator
---
# Sunday Generator [outfoxx/sunday-generator :fontawesome-brands-git-alt:{ . }](https://github.com/outfoxx/sunday-generator){ .git-repo-lockup }

---

The Sunday Generator is a Kotlin / Java based code generator that generates client APIs targeted for Sunday implementations in a number of languages as well as Kotlin / JAX-RS clients & servers from [RAML](why-raml.md) API definitions.

## Supported Languages & Frameworks

| Language                | Client Frameworks | Server Frameworks |
|:-----------------------:|:-----------------:|:-----------------:|
| Kotlin (JVM)            | [Sunday](kotlin-sunday.md) , [JAX-RS](kotlin-jaxrs.md) | [JAX-RS](kotlin-jaxrs.md) |
| Swift                   | [Sunday](swift-sunday.md) | |
| TypeScript / JavaScript | [Sunday](typescript-sunday.md) | |


## Executing Sunday Generator

The generator has multiple ways it can be executed to make it easily accessible in many build environments.

* [CLI](#cli)
* [Build Plugins](#build-plugins)

### CLI

The generator has a complete command line interface that allows generating code for any target. Being written in Kotlin the generator is first delivered as an executable UberJar (a single jar with all dependencies). Additionally, the CLI can be executed as a docker container for environments wheere docker is avaiable and Java is not already available.

=== "Executable JAR"

    ```console
    java -jar cli-all-{{ generator.latest_release }}.jar <target> <options>
    ```

=== "Docker"

    ```console
    docker run outfoxx/sunday-generator:{{ generator.latest_release }} <target> <options>
    ```

---

#### Targets / Options

Each available target has a specific set of allowed options which are detailed in the following table.

=== "Kotlin / Sunday"

    Generates clients targeting the Kotlin language and the Sunday library.

    !!! info "Target"
        
                kotlin/sunday

    !!! info "Options"

        `-out PATH`  *__required__*
        :   Output directory.

        `-pkg <package-name>`  *__required__*
        :   Default package name.

        `-model-pkg <package-name>`
        :   Default model package name.

            If not specified '-pkg' is used.

        `-service-pkg <package-name>`
        :   Default service package.

            If not specified '-pkg' is used.

        `-service-suffix <suffix>`
        :   Suffix for generated services.

            The suffix to append to generated service types. Defaults to `API`.

        `-media-type <media-type>`
        :   Specifies the order of default media types.

            This option can be given multiple times and the order given is the order of the default media-types.

        `-category [Service|Model]`
        :   Add category of type to output.
            
            This optiona can be given multiple times to add different output categories. Defaults to all types.

        `-problem-base <URI>`
        :   Default problem base URI.

        `-enable`
        :   Enables the given type generation option.

            Available Options:

            * `implement-model`
            * `validation-constraints`
            * `jackson-annotations`
            * `add-generated-annotation`
            * `suppress-public-api-warnings`

            

        `-disable`
        :   Disables the given type generation option.

            Available Options: 

            * `implement-model`
            * `validation-constraints`
            * `jackson-annotations`
            * `add-generated-annotation`
            * `suppress-public-api-warnings`

=== "Kotlin / JAX-RS"

    Generates clients or server stubs targeting the Kotlin language and the JAX-RS library.

    !!! info "Target"
        
                kotlin/jaxrs

    !!! info "Options"

        `-out PATH`  *__required__*
        :   Output directory.

        `-mode [client|server]`  *__required__*
        :   Target 'client' or 'server' for generated services.

        `-pkg <package-name>`  *__required__*
        :   Default package name.

        `-model-pkg <package-name>`
        :   Default model package name.

            If not specified '-pkg' is used.

        `-service-pkg <package-name>`
        :   Default service package.

            If not specified '-pkg' is used.

        `-service-suffix <suffix>`
        :   Suffix for generated services.

            The suffix to append to generated service types. Defaults to `API`.

        `-media-type <media-type>`
        :   Specifies the order of default media types.

            This option can be given multiple times and the order given is the order of the default media-types.

        `-category [Service|Model]`
        :   Add category of type to output.
            
            This optiona can be given multiple times to add different output categories. Defaults to all types.

        `-problem-base <URI>`
        :   Default problem base URI.


        `-enable`
        :   Enables the given type generation option.

            Available Options:
            
            * `implement-model`
            * `validation-constraints`
            * `jackson-annotations`
            * `add-generated-annotation`
            * `suppress-public-api-warnings`

            

        `-disable`
        :   Disables the given type generation option.

            Available Options:

            * `implement-model`
            * `validation-constraints`
            * `jackson-annotations`
            * `add-generated-annotation`
            * `suppress-public-api-warnings`

        `-coroutines`
        :   Generate suspendable service methods for coroutine support.

        `-reactive <type-name>`
        :   Generic result type for reactive service.

            Specifying this option also enables the generation of reactive service methods.

        `-explicit-security-parameters`
        :   Include explicit security parameters in service methods.


=== "Swift / Sunday"

    Generates clients targeting the Swift language and the Sunday library.

    !!! info "Target"
        
                swift/sunday

    !!! info "Options"

        `-out PATH`  *__required__*
        :   Output directory.

        `-service-suffix <suffix>`
        :   Suffix for generated services.

            The suffix to append to generated service types. Defaults to `API`.

        `-media-type <media-type>`
        :   Specifies the order of default media types.

            This option can be given multiple times and the order given is the order of the default media-types.

        `-category [Service|Model]`
        :   Add category of type to output.
            
            This optiona can be given multiple times to add different output categories. Defaults to all types.

        `-problem-base <URI>`
        :   Default problem base URI.

        `-enable`
        :   Enables the given type generation option.

            Available Options:

            * `add-generated-annotation`        

        `-disable`
        :   Disables the given type generation option.

            Available Options: 

            * `add-generated-annotation`


=== "TypeScript / Sunday"

    Generates clients targeting the TypeScript language and the Sunday library.

    !!! info "Target"
        
                typescript/sunday

    !!! info "Options"


        `-out PATH`  *__required__*
        :   Output directory.

        `-service-suffix <suffix>`
        :   Suffix for generated services.

            The suffix to append to generated service types. Defaults to `API`.

        `-media-type <media-type>`
        :   Specifies the order of default media types.

            This option can be given multiple times and the order given is the order of the default media-types.

        `-category [Service|Model]`
        :   Add category of type to output.
            
            This optiona can be given multiple times to add different output categories. Defaults to all types.

        `-problem-base <URI>`
        :   Default problem base URI.

        `-enable`
        :   Enables the given type generation option.

            Available Options:

            * `jackson-decorators`
            * `add-generated-annotation`        

        `-disable`
        :   Disables the given type generation option.

            Available Options: 

            * `jackson-decorators`
            * `add-generated-annotation`


### Build Plugins


In addition to the CLI, the genrator is delivered as build plugins for the following build tools:

#### Gradle

The Gradle plugin provides easy access for generating clients & server stubs for Kotlin/Java targets.

The plugin allow you to create multiple "generations" each with their distinct sets of options to allow generating all of your code in a single project. 

!!! Note
	Tht Gradle plugin _only_ supports generating __Kotlin__ & __Java__ targets.

=== "Groovy DSL"

	```groovy
	plugins {
		id "io.outfoxx.sunday-generator" version "{{ generator.latest_release }}"
	}

    sundayGenerations {

      // Create a generation named "api"

      api {

        // Input RAML files - default: "src/main/raml/*.raml"
        source = file("src/main/raml/api.raml")

        // Other RAML files dependencies - no default
        includes = fileTree("src/main/raml")

        // Output directory - default: "generated/sources/sunday/<generation-name>"
        outputDir = "generated"

        // TargetFramework.JAXRS or TargetFramework.Sunday - no default
        framework = TargetFramework.JAXRS

        // GenerationMode.Client or GenerationMode.Server - no default
        mode = GenerationMode.Client

        // Enable generation of model types - default: true
        generateModel = true

        // Enable generation of service types - default: true
        generateService = true

        // Default package name for model & service types - no default
        pkgName = "com.example.api"

        // Default package name for model types - no default
        modelPkgName = "com.example.api.model"

        // Default package name for service types - no default
        servicePkgName = "com.example.api.services"

        // Suffix to append to generated clients - default: "API"
        serviceSuffix = "Client"

        // Disable generation of bean validation annotations - default: true
        disableValidationConstraints = false

        // Disable generation of Jackson annotations - default: true
        disableJacksonAnnotations = false

        // Disable generation of model implementations - default: true
        disableModelImplementations = false

        // Enable generation of coroutines support in service methods - default: false
        coroutines = false

        // Enable generation of reactive response types in service methods - no default
        reactiveResponseType = "CompletableFuture"

        // Add security parameters to service methods - default: false
        explicitSecurityParameters = false

        // Set order of default media types
        defaultMediaTypes = listOf("application/json")

      }
      
    }
	```

=== "Kotlin DSL"

	```kotlin
	plugins {
		id("io.outfoxx.sunday-generator") version("{{ generator.latest_release }}")
	}

    sundayGenerations {

      // Create a generation named "api"

      val api by creating {

        // Input RAML files - default: "src/main/raml/*.raml"
        source.set(file("src/main/raml/api.raml"))

        // Other RAML files dependencies - no default
        includes.set(fileTree("src/main/raml"))

        // Output directory - default: "generated/sources/sunday/<generation-name>"
        outputDir.set("generated")

        // TargetFramework.JAXRS or TargetFramework.Sunday - no default
        framework.set(TargetFramework.JAXRS)

        // GenerationMode.Client or GenerationMode.Server - no default
        mode.set(GenerationMode.Client)

        // Enable generation of model types - default: true
        generateModel.set(true)

        // Enable generation of service types - default: true
        generateService.set(true)

        // Default package name for model & service types - no default
        pkgName.set("com.example.api")

        // Default package name for model types - no default
        modelPkgName.set("com.example.api.model")

        // Default package name for service types - no default
        servicePkgName.set("com.example.api.services")

        // Suffix to append to generated clients - default: "API"
        serviceSuffix.set("Client")

        // Disable generation of bean validation annotations - default: true
        disableValidationConstraints.set(false)

        // Disable generation of Jackson annotations - default: true
        disableJacksonAnnotations.set(false)

        // Disable generation of model implementations - default: true
        disableModelImplementations.set(false)

        // Enable generation of coroutines support in service methods - default: false
        coroutines.set(false)

        // Enable generation of reactive response types in service methods - no default
        reactiveResponseType.set("CompletableFuture")

        // Add security parameters to service methods - default: false
        explicitSecurityParameters.set(false)

        // Set order of default media types
        defaultMediaTypes.set(listOf("application/json"))

      }

    }
	```
!!! tip
	The above examples may not show all of the options supported by the Gradle generator plugin. For a detailed list of options see check the following documentation pages [Options for All Targets](target-common-features.md#generator-options), [Options for Kotlin Targets](target-kotlin-common-features.md#generator-options),  [Options for the Kotlin/Sunday Target](kotlin-sunday.md#generator-options), and [Options for the Kotlin/JAX-RS Target](kotlin-jaxrs.md#generator-options)

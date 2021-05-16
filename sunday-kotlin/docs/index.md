---
title: Sunday (Kotlin)
---

# Sunday (Kotlin) [outfoxx/sunday-kt :fontawesome-brands-git-alt:](https://github.com/outfoxx/sunday-kt){ .git-repo-lockup }

A Sunday client library implementation written in Kotlin and supporting the JVM and Android.

!!! tip

    While you can use Sunday (Kotlin) to write REST clients manually Sunday is most useful when generating clietns from API definitions.

    [Learn about Sunday Generator](../generator/index.md)

## Installation

Sunday is delivered as a standard Maven library from Maven Central.

=== "Gradle (Kotlin DSL)"

    ```kotlin

    dependencies {
      implementation("io.outfoxx:sunday:{{ sunday.kotlin.latest_version }}")
    }

    ```

=== "Gradle (Groovy DSL)"

    ```groovy

    dependencies {
      implementation 'io.outfoxx:sunday:{{ sunday.kotlin.latest_version }}'
    }

    ```

=== "Maven"

    ```xml
    <dependencies>
        <dependency>
            <groupId>io.outfoxx</groupId>
            <artifactId>sunday</artifactId>
            <version>{{ sunday.kotlin.latest_version }}</version>
        </dependency>
    </dependencies>
    ```

## Usage

## License

    Copyright 2015 Outfox, Inc.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

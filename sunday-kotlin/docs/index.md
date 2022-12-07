---
title: Sunday - Kotlin
---

# Sunday <font size="3">__KOTLIN__</font> [outfoxx/sunday-kt   :fontawesome-brands-github:](https://github.com/outfoxx/sunday-kt){ .git-repo-lockup }

A Sunday client library implementation written in Kotlin and supporting the JVM and Android.

---

!!! tip

    While you can use Sunday (Kotlin) to write REST clients manually, Sunday is most useful when generating clietns from API
    definitions using the Sunday Generator.

    [Learn about Sunday Generator](../generator/index.md)

## Implementations

Sunday Kotlin is modularized to allow for multiple implementations. Currently the following implementations are available:

:fontawesome-brands-java: __JDK HTTP Client__
:   Uses the native [JDK 11 HTTP Client](https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/package-summary.html) to execute HTTP requests.

:fontawesome-regular-square: __Square OkHttp__
:   Uses Square's [OkHttp](https://square.github.io/okhttp/) library (version 4) to execute HTTP requests. OkHttp supports easy configuration of advanced features like
    certificate pinning and configuration interceptors.

## Installation

Sunday is delivered as a standard Maven library from Maven Central.

=== "Gradle (Kotlin DSL)"

    ```kotlin

    dependencies {
      // If using JDK implementation
      implementation("io.outfoxx.sunday:sunday-jdk:{{ client.kotlin.latest_release }}")

      // If using OkHttp implementation
      implementation("io.outfoxx.sunday:sunday-okhttp:{{ client.kotlin.latest_release }}")
    }

    ```

=== "Gradle (Groovy DSL)"

    ```groovy

    dependencies {
      // If using JDK implementation
      implementation 'io.outfoxx.sunday:sunday-jdk:{{ client.kotlin.latest_release }}'

      // If using OkHttp implementation
      implementation 'io.outfoxx.sunday:sunday-okhttp:{{ client.kotlin.latest_release }}'
    }

    ```

=== "Maven"

    ```xml
    <dependencies>
        <dependency>
            <groupId>io.outfoxx.sunday</groupId>

            <!-- If using JDK implementation -->
            <artifactId>sunday-jdk</artifactId>

            <!-- If using OkHttp implementation -->
            <artifactId>sunday-okhttp</artifactId>

            <version>{{ client.kotlin.latest_release }}</version>
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

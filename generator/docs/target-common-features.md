---
title: Sunday - Generator - Targets - Common Features
---
# Common Target Features

The following features are supported by all code generation targets.

## Generator Options

__Output Directory__
:   Specifies the output directory for generated files.

    | CLI Option        | Gradle Plugin Properties             | Type      | Default |
    | ----------------- | ------------------------------------ | --------- | ------- |
	| `-out`            | `outputDir`   					   | directory | None    |


__Output Categories__
:   Selects which categories of output types are generated.

    | CLI Option        | Gradle Plugin Properties             | Type   | Allowed Values     | Default |
    | ----------------- | ------------------------------------ | ------ | ------------------ | ------- |
	| `-category`       | `generateModel`, `generateService`   | flags  | `service`, `model` | None    |

	!!! note
		The option can be specified multiple times, each enabling output of the specified category.


__Service Suffix__
:   Specifies the suffix appended to each generated service name.

    | CLI Option        | Gradle Plugin Property | Type   | Default |
    | ----------------- | ---------------------- | ------ | ------- |
    | `-service-suffix` | `serviceSuffix`        | string | `API`   |


__Default Media Type Ordering__
:   Specifies the order of default media types.

    | CLI Option        | Gradle Plugin Property | Type    | Default |
    | ----------------- | ---------------------- | ------- | ------- |
	| `-media-type`     | `defaultMediaTypes`    | strings | None    |

	!!! note
		The CLI option can be specified multiple times, the order in which they are specified controls
		the default order of media-types.


__Default Problem Base URI__
:   Specifies the default base URI to use for resolving problem type URIs

    | CLI Option        | Gradle Plugin Property | Type   | Default              |
    | ----------------- | ---------------------- | ------ | -------------------- |
	| `-problem-base`   | None				     | string | `http://example.com` |

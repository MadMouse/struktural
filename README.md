# Struktural

## What
Struktural is a suite of tools written in Kotlin designed to make testing of APIs easier in Java/Kotlin.

Struktural is designed to give flexible control over the level of desired testing.

| Features               | Description                                                       |
| ---------------------- | ----------------------------------------------------------------- |
| Assert Json Structure  | A lightweight test to assert presence of fields.                  |
| Assert Json Types      | A middleweight test to assert presence of fields and their types. |
| Assert Json Values     | A heavyweight test to assert presence of fields and their values. <br/>Supports exact values as well as matchers via Hamcrest. |

Struktural provides two interfaces.
1. A native Kotlin interface for running tests. (Interfaces natively with Java)
2. A YAML driven test format. Place tests in a YAML format and don't type any Java/Kotlin at all!
    - There are also plans to build a Maven plugin for this. Initially there will be a helper class to load and run all the YAML tests.

There are two libraries that influenced me and my desire to build this library.
1. Ruby's [Airborne](https://github.com/skyscreamer/JSONassert) library which when combined with RSpec make beautiful and lightweight API testing.
2. Java's [Skyscreamer's JSONAsert](https://github.com/skyscreamer/JSONassert) library which made api testing pain a bit easier.

## Why
Testing APIs in Java/Kotlin often involves verbose methodologies following on of three patterns:
1. Assert each field individually from a raw JSON or Map object.
2. Compare expected JSON from a resource like sample_expected.json with a response
3. Mapping responses to POJOs and peforming `equals` checks or checking field by field.

This is an attempt to make writing integration tests more fun and remove some of the pain often associated with writing API integration tests in Java.
I think this is especially important as the world continues to adopt Service Oriented Architectures.

## Where
Struktural is available on Maven Central. (Or will be very soon)

```xml
<dependency>
    <groupId>com.kennycason</groupId>
    <artifactId>struktural</artifactId>
    <version>1.0.3</version>
</dependency>
```

## How

### Kotlin API

#### Assert Field Structure

```kotlin
val json = """
{
    "name": "kenny",
    "age": 64,
    "job": {
        "id": 123456,
        "title": "Software Engineer"
    }
}
"""
Struktural.assertStructure(json,
        listOf("name",
                "age",
                Pair("job",
                    listOf("id",
                           "title"))))
```

Nested array of objects

```kotlin
val json = """
{
    "languages": [
        {
            "name": "kotlin",
            "coolness": 100
        },
        {
            "name": "java",
            "coolness": 50
        }
    ]
}
"""
Struktural.assertStructure(json,
        listOf(Pair("languages",
                listOf("name",
                       "coolness"))))
```


#### Assert Field Type Structure
```kotlin
val json = """
{
    "name": "kenny",
    "age": 64,
    "shoe_size": 10.5,
    "favorite_number": 2.718281828459045235,
    "long_number": 1223235345342348,
    "random": [1,2,3,4,5,6],
    "job": {
        "id": 123456,
        "title": "Software Engineer"
    }
}
"""
```

Strict number types

```kotlin
Struktural.assertTypes(json,
         listOf(Pair("name", String::class),
                Pair("age", Int::class),
                Pair("shoe_size", Float::class),
                Pair("favorite_number", Double::class),
                Pair("long_number", Long::class),
                Pair("random", Array<Any>::class),
                Pair("job", Object::class),
                Pair("job", listOf(Pair("id", Int::class),
                                   Pair("title", String::class)))))
```

Relaxed number types

```kotlin
Struktural.assertTypes(json,
         listOf(Pair("name", String::class),
                Pair("age", Number::class),
                Pair("shoe_size", Number::class),
                Pair("favorite_number", Number::class),
                Pair("long_number", Number::class),
                Pair("random", Array<Any>::class),
                Pair("job", Object::class),
                Pair("job", listOf(Pair("id", Number::class),
                           Pair("title", String::class)))))

```

Nested array of objects

```kotlin
val json = """
{
    "languages": [
        {
            "name": "kotlin",
            "coolness": 100
        },
        {
            "name": "java",
            "coolness": 50
        }
    ]
}

"""
Struktural.assertTypes(json,
        listOf(Pair("languages",
                listOf(Pair("name", String::class),
                       Pair("coolness", Number::class)))))

```

Nullable values

```kotlin
val json = """
{
    "foo": null
}
"""
Struktural.assertTypes(json,
        listOf(Pair("foo", Nullable(String::class))))
```

#### Assert Field Values

```kotlin
val json = """
{
    "name": "kenny",
    "age": 64,
    "shoe_size": 10.5,
    "favorite_number": 2.718281828459045235,
    "long_number": 1223235345342348,
    "job": {
        "id": 123456,
        "title": "Software Engineer"
    }
}
"""
Struktural.assertValues(json,
         listOf(Pair("name", "kenny"),
                Pair("age", 64),
                Pair("shoe_size", 10.5),
                Pair("favorite_number", 2.718281828459045235),
                Pair("long_number", 1223235345342348),
                Pair("job", listOf(Pair("id", 123456),
                                   Pair("title", "Software Engineer")))))

```

Only match partial

```kotlin
Struktural.assertValues(json,
         listOf(Pair("name", "kenny"),
                Pair("favorite_number", 2.718281828459045235)))
```

Simple array example

```kotlin
val json = """
{
    "numbers": [1,2,3,4,5,6]
}
"""
Struktural.assertValues(json,
                listOf(Pair("numbers", arrayOf(1, 2, 3, 4, 5, 6))))

```

Nested array example

```kotlin
val json = """
{
    "people": [
        {
            "name": "kenny",
            "favorite_language": "kotlin",
            "age": 64
        },
        {
            "name": "martin",
            "favorite_language": "kotlin",
            "age": 92
        }
    ]
}
"""
Struktural.assertValues(json,
        listOf(Pair("people",
                listOf(Pair("favorite_language", "kotlin")))))


```

Using Hamcrest matchers

```kotlin
val json = """
{
    "people": [
        {
            "name": "kenny",
            "favorite_language": "Kotlin",
            "age": 64
        },
        {
            "name": "martin",
            "favorite_language": "Kotlin",
            "age": 92
        }
    ]
}
"""
validator.assert(json,
        listOf(Pair("people",
                listOf(
                        Pair("name", Matchers.notNullValue()),
                        Pair("favorite_language", Matchers.equalToIgnoringCase("kotlin")),
                        Pair("age", Matchers.greaterThan(50))))))
```


### YAML API

In addition to the native Kotlin/Java API Unit tests can also be configured via YAML files.

Example API Test #1
```yaml
---
config:
  base_url: http://api.company.com

tests:
  -
    mode: type
    data:
      request:
        uri: /language/detection
        method: POST
        body: '{"data":[{"id":"1","text":"I am an english comment"}]}'
        headers:
          - 'Content-Type: application/json'

    expects:
      - data:
          id: string
          language:
            name: string
            code: string
            score: int
            is_reliable: bool
```

Example API Test #2
```yaml
---
tests:
  -
    mode: type
    data:
      request:
        uri: https://api.company.com/labels
        method: GET
        params:
          - 'include_inactive=true'
        headers:
          - 'Authorization: Bearer <AUTH_TOKEN>'
          - 'Content-Type: application/json'

    expects:
      - data:
          type: string
          id: string
          attributes:
            account_id: string
            name: string
            color: string
            created_at: string
            created_by: string
            updated_at: string
            updated_by: string
            active: bool
```

The YAML format also provides options for validating json files, resources, as well as a variety of configurations.
The YAML format and description of properties can be found below:
```yaml
---
# config block provides section for global configs
config:
  # base_url is an optional field to remove some verbosity when testing apis.
  # it is prepended to data.request.uri if set.
  base_url: https://api.foobar.com
  port: 8080

tests:
  - # array of tests
    # pick one of three modes for testing
    #   structure = assert fields not missing
    #   type      = assert fields not missing and field types
    #   value     = assert fields not missing and field values
    mode: structure | type | value
    # the data block provides methods for providing data
    data:
      # 1. configuration for url requests
      request:
        uri: /v2/foo/bar
        method: POST
        body: '{"foo":"bar"}'
        params:
          - 'field=value'
          - 'field2=value2'
        headers:
          - 'Authorization: key'

      # 2. configuration for loading json from resource, great for unit tests
      resource: /path/to/resource/food.json
      # 3. configuration for loading file from file system
      file: /path/to/file.json

    expects:
      # note that you must choose ONE of the below formats
      # example for mode: structure
     - name
     - age
     - job:
       - id
       - title

      # example for mode: types
     - name: string
     - age: int
     - job:
         id: int
         title: string

      # example for mode: values
      - name: kenny
      - age: 30
      - job:
          id: 123456
          title: Software Engineer
```

#### Yaml Test Examples

Assert tests in YAML file are valid
```kotlin
Struktural.assertYaml(javaClass.getResourceAsStream("/path/to/resource/my_test.yml"))
```


Test against raw YAML String
```kotlin
val yaml = """
---
tests:
  -
    mode: type
    data:
      resource: /com/kennycason/struktural/json/person_sample_response.json

    expects:
      - name: string
      - age: int
      - job:
          id: int
          title: string
"""

Struktural.assertYaml(yaml)
```

### Kotlin + Struktural + Spek

A small example using JetBrain's [Spek Framework](http://spekframework.org/)

```kotlin
class LanguageClassifierApiTests : Spek( {

    describe("Language Classification API Tests") {

        it("Response structure & types") {
            val json = HttpJsonLoader(
                request = Request(uri = "http://api.service.com/language/detection",
                        method = HttpMethod.POST,
                        body = """{"items":[{"id":"1","text":"I am a happy person"}]}""",
                        headers = listOf<Header>(BasicHeader("Content-Type", "application/json"))))
                .load()
            Struktural.assertTypes(json,
                    listOf(Pair("items", listOf(
                            Pair("id", String::class),
                            Pair("language", listOf(
                                    Pair("name", String::class),
                                    Pair("code", String::class),
                                    Pair("score", Int::class),
                                    Pair("is_reliable", Boolean::class)))))))
        }
    }

})
```


## Notes
- Pass context from test-to-test. Allow a response form one test to drive the next test.
- Better error handling/logging to come.
- I'm looking for ideas on more features. e.g.
     - Maven Plugin to automatically scan resource for yaml test files, or some similar concept to further facility configuring of tests
     - extra validation functions
- Currently the project has a hard dependency on Apache Http Client and Jackson Json parsing. Eventually these *may* be extracted out so that you can choose your library.
- Much of the inernal code will be cleaned up and better organized in time. This was a few day proof-of-concept project.



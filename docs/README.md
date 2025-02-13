# Annotations from comments

JSON schema is partially implemented in this tool. It uses line comments to add annotations for the schema because head comments are frequently used by humans and tools like helm-docs. Multiple annotations can be added to a single line sepparated by semi-colon. For example:

```yaml
nameOverride: "myapp" # @schema maxLength:10;patter:^[a-z]+$
```

This will generate following schema:

```json
"nameOverride": {
    "maxLength": 10,
    "type": "string"
}
```

The following annotations are supported:

* [Validation Keywords for Any Instance Type](#validation-keywords-for-any-instance-type)
    * [Type](#type)
    * [Multiple types](#multiple-types)
    * [Enum](#enum)
* [Strings](#strings)
    * [maxLength](#maxlength)
    * [minLength](#minlength)
    * [pattern](#pattern)
* [Numbers](#numbers)
    * [multipleOf](#multipleof)
    * [maximum](#maximum)
    * [minimum](#minimum)
* [Arrays](#arrays)
    * [maxItems](#maxitems)
    * [minItems](#minitems)
    * [uniqueItems](#uniqueitems)
* [Objects](#objects)
    * [minProperties](#minproperties)
    * [maxProperties](#maxproperties)
    * [required](#required)
* [Meta-Data Annotations](#meta-data-annotations)
    * [title](#title)

## Validation Keywords for Any Instance Type

### Type

The `type` keyword is used to restrict a value to a specific primitive type. There are several possible values for `type`:

* `string`
* `number`
* `integer`
* `boolean`
* `object`
* `array`
* `null`

### Multiple types

Default behaviour returns always a string unless annotation is used. In that case, it returns array of strings. Useful for keys without any value declared for documentation purposes. [section 6.1.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.1.1)

```yaml
# -- (int) replica count
replicaCount: # @schema type:[integer, null]
```

```json
"replicaCount": {
    "type": [
        "integer",
        "null"
    ]
}
```

Another way to use this is to define type when using anchors and aliases in yaml. See discussion [#28](https://github.com/losisin/helm-values-schema-json/issues/28) for more details.

```yaml
app: &app
  settings:
    namespace:
      - *app # @schema type:[string]
```

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "properties": {
        "app": {
            "properties": {
                "settings": {
                    "properties": {
                        "namespace": {
                            "items": {
                                "type": [
                                    "string"
                                ]
                            },
                            "type": "array"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        }
    },
    "type": "object"
}
```

### Enum

Always returns array of strings. Special case is `null` where instead of string, it is treated as valid inpput type.  [section 6.1.2](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.1.2)

```yaml
service: ClusterIP # @schema enum:[ClusterIP, LoadBalancer, null]
```
    
```json
"service": {
    "enum": [
        "ClusterIP",
        "LoadBalancer",
        null
    ],
    "type": "string"
}
```

## Strings

### maxLength

Non-negative integer. [section 6.3.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.3.1)

```yaml
nameOverride: "myapp" # @schema maxLength:10
```

This will generate following schema:

```json
"nameOverride": {
    "maxLength": 10,
    "type": "string"
}
```

### minLength

Non-negative integer. [section 6.3.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.3.2)

```yaml
nameOverride: "myapp" # @schema minLength:3
```

This will generate following schema:

```json
"nameOverride": {
    "minLength": 3,
    "type": "string"
}
```

### pattern

String that is valid regular expression, according to the ECMA-262 regular expression dialect. [section 6.3.3](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.3.3)

```yaml
nameOverride: "myapp" # @schema patter:^[a-z]+$
```

This will generate following schema:

```json
"nameOverride": {
    "type": "string"
}
```

## Numbers

### multipleOf

Number greater than `0`. [section 6.2.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.2.1)

```yaml
replicas: 2 # @schema multipleOf:2
```
 
```json
"replicas": {
    "multipleOf": 2,
    "type": "integer"
}
```

### maximum

Number. [section 6.2.2](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.2.2)

```yaml
replicas: 2 # @schema maximum:10
```
 
```json
"replicas": {
    "maximum": 10,
    "type": "integer"
}
```

### minimum

Number. [section 6.2.4](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.2.4)

```yaml
replicas: 5 # @schema minimum:2
```
 
```json
"replicas": {
    "minimum": 2,
    "type": "integer"
}
```

## Arrays

### maxItems

Non-negative integer. [section 6.4.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.4.1)

```yaml
dummyList: # @schema maxItems:5
  - "item1"
  - "item2"
  - "item3"
```

```json
"dummyList": {
    "items": {
        "type": "string"
    },
    "maxItems": 5,
    "type": "array"
}
```

### minItems

Non-negative integer. [section 6.4.2](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.4.2)

```yaml
dummyList: # @schema minItems:2
  - "item1"
  - "item2"
  - "item3"
```

```json
"dummyList": {
    "items": {
        "type": "string"
    },
    "minItems": 2,
    "type": "array"
}
```

### uniqueItems

Boolean. [section 6.4.3](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.4.3)

```yaml
dummyList: # @schema uniqueItems:true
  - "item1"
  - "item2"
  - "item3"
```

```json
"dummyList": {
    "items": {
        "type": "string"
    },
    "type": "array",
    "uniqueItems": true
}
```

## Objects

### maxProperties

Non-negative integer. [section 6.5.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.5.1)

```yaml
nodeSelector: # @schema maxProperties:10
  kubernetes.io/hostname: "my-node"
```

```json
"nodeSelector": {
    "maxProperties": 10,
    "properties": {
        "kubernetes.io/hostname": {
            "type": "string"
        }
    },
    "type": "object"
}
```

### minProperties

Non-negative integer. [section 6.5.2](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.5.2)

```yaml
nodeSelector: # @schema minProperties:1
  kubernetes.io/hostname: "my-node"
```

```json
"nodeSelector": {
    "minProperties": 1,
    "properties": {
        "kubernetes.io/hostname": {
            "type": "string"
        }
    },
    "type": "object"
}
```

### required

Array of unique strings appened to the parent node. [section 6.5.3](https://json-schema.org/draft/2020-12/json-schema-validation#section-6.5.3)

```yaml
image:
  repository: "nginx" # @schema required:true
  tag: "latest" # @schema required:true
  imagePullPolicy: "IfNotPresent"
```

```json
"image": {
    "properties": {
        "imagePullPolicy": {
            "type": "string"
        },
        "repository": {
            "type": "string"
        },
        "tag": {
            "type": "string"
        }
    },
    "required": [
        "repository",
        "tag"
    ],
    "type": "object"
}
```

## Meta-Data Annotations

### title

String. [section 9.1](https://json-schema.org/draft/2020-12/json-schema-validation#section-9.1)

```yaml
fullnameOverride: bar # @schema title: My title
```

```json
"fullnameOverride": {
    "title": "My title",
    "type": "string"
},
```

### readOnly

Boolean. [section 9.4](https://json-schema.org/draft/2020-12/json-schema-validation#section-9.4)

```yaml
image:
  tag: latest # @schema readOnly: true
```

```json
"image": {
    "properties": {
        "tag": {
            "readOnly": true,
            "type": "string"
        }
    },
    "type": "object"
}
```

# Zod to Json Schema

[![Build](https://img.shields.io/github/workflow/status/stefanterdell/zod-to-json-schema/Tests)](https://github.com/StefanTerdell/zod-to-json-schema)
[![NPM Version](https://img.shields.io/npm/v/zod-to-json-schema.svg)](https://npmjs.org/package/zod-to-json-schema)
[![NPM Downloads](https://img.shields.io/npm/dw/zod-to-json-schema.svg)](https://npmjs.org/package/zod-to-json-schema)

## Summary

Does what it says on the tin; converts [Zod schemas](https://github.com/colinhacks/zod) into [JSON schemas](https://json-schema.org/)!

- Supports all relevant schema types, basic string, number and array length validations and string patterns.
- Resolves recursive and recurring schemas with internal `$ref`s.
- Also able to target Open API 3 (Swagger) specification for paths.

### Usage

```typescript
import { z } from "zod";
import zodToJsonSchema from "zod-to-json-schema";

const mySchema = z.object({
  myString: z.string().min(5),
  myUnion: z.union([z.number(), z.boolean()]),
});

const jsonSchema = zodToJsonSchema(mySchema, "mySchema");
```

### Expected output

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "#/definitions/mySchema",
  "definitions": {
    "mySchema": {
      "type": "object",
      "properties": {
        "myString": {
          "type": "string",
          "minLength": 5
        },
        "myUnion": {
          "type": ["number", "boolean"]
        }
      },
      "additionalProperties": false,
      "required": ["myString", "myUnion"]
    }
  }
}
```

## Options

### Schema name

You can pass a string as the second parameter of the main zodToJsonSchema function. If you do, your schema will end up inside a definitions object property on the root and referenced from there. Alternatively, you can pass the name as the `name` property of the options object (see below).

### Options object

Instead of the schema name (or nothing), you can pass an options object as the second parameter. The following options are available:

| Option                                            | Effect                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **name**?: _string_                               | As described above.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **basePath**?: string[]                           | The base path of the root reference builder. Defaults to ["#"].                                                                                                                                                                                                                                                                                                                                                                                              |
| **$refStrategy**?: "root" \| "relative" \| "none" | The reference builder strategy; <ul><li>**"root"** resolves $refs from the root up, ie: "#/definitions/mySchema".</li><li>**"relative"** uses [relative JSON pointers](https://tools.ietf.org/id/draft-handrews-relative-json-pointer-00.html). _See known issues!_</li><li>**"none"** ignores referencing all together, creating a new schema branch even on "seen" schemas. Recursive references defaults to "any", ie `{}`.</li></ul> Defaults to "root". |
| **effectStrategy**?: "input" \| "any"             | The effects output strategy. Defaults to "input". _See known issues!_                                                                                                                                                                                                                                                                                                                                                                                        |
| **definitionPath**?: "definitions" \| "$defs"     | The name of the definitions property when name is passed. Defaults to "definitions".                                                                                                                                                                                                                                                                                                                                                                         |
| **target**?: "jsonSchema7" \| "openApi3"          | Which spec to target. Defaults to "jsonSchema7"                                                                                                                                                                                                                                                                                                                                                                                                              |

## Known issues

- When using `.transform`, the return type is inferred from the supplied function. In other words, there is no schema for the return type, and there is no way to convert it in runtime. Currently the JSON schema will therefore reflect the input side of the Zod schema and not necessarily the output (the latter aka. `z.infer`). If this causes problems with your schema, consider using the effectStrategy "any", which will allow any type of output.
- JSON Schemas does not support any other key type than strings for objects. When using `z.record` with any other key type, this will be ignored.
- Relative JSON pointers, while published alongside [JSON schema draft 2020-12](https://json-schema.org/specification.html), is not technically a part of it. Currently, most resolvers do not handle them at all.

## Versioning

This package _does not_ follow semantic versioning. The major and minor versions of this package instead reflects feature parity with the [Zod package](http://npmjs.com/package/zod).

I will do my best to keep API-breaking changes to an absolute minimum, but new features may appear as "patches", such as introducing the options pattern in 3.9.1.

## Changelog

https://github.com/StefanTerdell/zod-to-json-schema/blob/master/changelog.md

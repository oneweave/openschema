# OpenSchema Specification

## Attribution

This first draft is structurally inspired by the document style used by the OpenAPI and AsyncAPI specifications.

## Conventions

### Version 0.1.0-draft

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

The OpenSchema Specification is licensed under [The Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0.html).

## Introduction

The OpenSchema Specification is a project used to describe database schemas in a machine-readable format. It is intended to be neutral with respect to storage model and implementation, so it can describe relational, document, key-value, and other structured data systems without assuming one specific database architecture.

An OpenSchema document describes the logical structure of stored data. The document MAY reference other files for additional details or shared components, but it is typically a single primary document that encapsulates one or more logical schemas.

The highest-level modeled entity in the specification is a named [schema](#definitionsSchema) within the root `schemas` map. Inside a schema, named [containers](#definitionsContainer) are declared directly as schema entries. A container is a neutral abstraction for a persisted record set whose physical realization is determined by the target system.

Containers define [fields](#definitionsField). A field MAY use a primitive type such as `string`, `number`, or `boolean`. A field MAY also describe nested object structure or repeated values. Numeric width, precision, and representation SHOULD be expressed using the field `format`. Required fields are declared at the container level, and uniqueness is expressed through container indexes. In addition, a container MAY define nested containers when the target system supports embedded or hierarchical storage patterns.

The OpenSchema Specification does not assume SQL-only or NoSQL-only behavior. Features such as primary keys, embedded objects, repeated values, and nested containers are expressed as optional structural capabilities. Tooling and implementations MAY support all or only part of the model depending on the target database system.

For example:

```yaml
openschema: 0.1.0-draft
info:
  title: Customer Data Model
  version: 1.0.0
schemas:
  customer-data:
    customers:
      primaryKey:
        - customerId
      required:
        - customerId
      indexes:
        - fields:
            - name: email
          unique: true
            
        - fields:
            - name: lastName
              order: asc
            - name: firstName
              order: asc
      fields:
        customerId:
          type: string
        email:
          type: string
        profile:
          type: object
          fields:
            firstName:
              type: string
            lastName:
              type: string
    orders:
      required:
        - id
      fields:
        id:
          type: string
        items:
          type: array
          items:
            type: object
            fields:
              sku:
                type: string
              quantity:
                type: number
                format: int32
```

## Table of Contents
<!-- TOC depthFrom:2 depthTo:4 withLinks:1 updateOnSave:0 orderedList:0 -->

- [Conventions](#conventions)
- [Definitions](#definitions)
  - [Schema](#definitionsSchema)
  - [Container](#definitionsContainer)
  - [Record](#definitionsRecord)
  - [Field](#definitionsField)
  - [Primitive Type](#definitionsPrimitiveType)
  - [Nested Structure](#definitionsNestedStructure)
  - [System](#definitionsSystem)
  - [Index](#definitionsIndex)
  - [Component](#definitionsComponent)
- [Specification](#specification)
  - [Format](#format)
  - [File Structure](#file-structure)
  - [Absolute URLs](#absolute-urls)
  - [Schema](#schema)
    - [OpenSchema Object](#openSchemaObject)
    - [OpenSchema Version String](#openSchemaVersionString)
    - [Identifier](#openSchemaIdString)
    - [Info Object](#infoObject)
    - [Contact Object](#contactObject)
    - [License Object](#licenseObject)
    - [Systems Object](#systemsObject)
    - [System Object](#systemObject)
    - [Schemas Object](#schemasObject)
    - [Schema Object](#schemaObject)
    - [Containers Object](#containersObject)
    - [Container Object](#containerObject)
    - [Index Object](#indexObject)
    - [Index Field Object](#indexFieldObject)
    - [Fields Object](#fieldsObject)
    - [Field Object](#fieldObject)
    - [Tags Object](#tagsObject)
    - [Tag Object](#tagObject)
    - [External Documentation Object](#externalDocumentationObject)
    - [Components Object](#componentsObject)
    - [Reference Object](#referenceObject)
    - [Specification Extensions](#specificationExtensions)

<!-- /TOC -->

## <a name="definitions"></a>Definitions

### <a name="definitionsSchema"></a>Schema

A schema is a logical description of persisted data. A schema groups one or more [containers](#definitionsContainer), their [fields](#definitionsField), and the constraints or metadata needed to understand the structure of stored records.

### <a name="definitionsContainer"></a>Container

A container is a named logical unit inside a [schema](#definitionsSchema). A container MAY correspond to a table, collection, bucket, relation, or similar construct in a concrete database system. A container stores zero or more [records](#definitionsRecord).

### <a name="definitionsRecord"></a>Record

A record is a single stored instance inside a [container](#definitionsContainer). Depending on the target system, a record MAY be represented as a row, document, entity, item, or similar persisted value.

### <a name="definitionsField"></a>Field

A field is a named value within a [record](#definitionsRecord). A field MAY be primitive, structured, repeated, or derived from nested structure. A field definition describes its type and any relevant schema constraints.

### <a name="definitionsPrimitiveType"></a>Primitive Type

A primitive type is a scalar value that does not define child fields. Primitive types include, but are not limited to, `string`, `number`, `boolean`, `date`, `datetime`, `bytes`, and `json`.

### <a name="definitionsNestedStructure"></a>Nested Structure

A nested structure is a field or container definition that contains child fields or child containers. Nested structure MAY be used to model embedded objects, arrays of objects, or hierarchical collections in systems that support them.

### <a name="definitionsSystem"></a>System

A system is a concrete database platform, engine, or deployment target where a schema MAY be stored, validated, generated, or applied.

### <a name="definitionsIndex"></a>Index

An index is a declared access path over one or more fields within a [container](#definitionsContainer). An index MAY enforce uniqueness and MAY restrict its scope to a subset of records. The physical realization of an index is determined by the target system.

### <a name="definitionsComponent"></a>Component

A component is a reusable definition stored under the `components` key of an OpenSchema document. Components MAY define reusable schemas, containers, or fields that can be referenced from elsewhere in the document using a [Reference Object](#referenceObject).

## <a name="specification"></a>Specification

### <a name="format"></a>Format

The files describing a schema in accordance with the OpenSchema Specification are represented as JSON objects and conform to JSON standards. YAML, being a superset of JSON, can be used as well to represent an OpenSchema file.

For example, if a field is said to have an array value, the JSON array representation will be used:

```json
{
  "field": []
}
```

All field names in the specification are case-sensitive.

The schema exposes two types of fields: fixed fields, which have a declared name, and patterned fields, which declare a regex pattern for the field name. Patterned fields can have multiple occurrences as long as each has a unique name.

In order to preserve the ability to round-trip between YAML and JSON formats, YAML version [1.2](https://www.yaml.org/spec/1.2/spec.html) is RECOMMENDED along with these constraints:

- Tags MUST be limited to those allowed by the JSON Schema ruleset.
- Keys used in YAML maps MUST be limited to scalar strings.

### <a name="file-structure"></a>File Structure

An OpenSchema document MAY be made up of a single document or be divided into multiple connected parts at the discretion of the author. In the latter case, [Reference Objects](#referenceObject) are used.

By convention, an OpenSchema file is named `openschema.json` or `openschema.yaml`.

### <a name="absolute-urls"></a>Absolute URLs

Unless specified otherwise, all properties that are absolute URLs are defined by [RFC3986, section 4.3](https://datatracker.ietf.org/doc/html/rfc3986#section-4.3).

### <a name="schema"></a>Schema

#### <a name="openSchemaObject"></a>OpenSchema Object

This is the root document object for the schema specification.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="openSchemaVersion"></a>openschema | [OpenSchema Version String](#openSchemaVersionString) | **REQUIRED.** Specifies the OpenSchema Specification version being used.
<a name="openSchemaId"></a>id | [Identifier](#openSchemaIdString) | Identifier of the schema document.
<a name="openSchemaInfo"></a>info | [Info Object](#infoObject) | **REQUIRED.** Provides metadata about the schema document.
<a name="openSchemaSystems"></a>systems | [Systems Object](#systemsObject) | A map of concrete systems where the schemas described by this document MAY be applied or interpreted.
<a name="openSchemaSchemas"></a>schemas | [Schemas Object](#schemasObject) | **REQUIRED.** The logical schemas described by this document.
<a name="openSchemaComponents"></a>components | [Components Object](#componentsObject) | A container for reusable schema components.

This object MAY be extended with [Specification Extensions](#specificationExtensions).

##### OpenSchema Object Example

```yaml
openschema: 0.1.0-draft
id: urn:oneweave:openschema:customer-data
info:
  title: Customer Data Model
  version: 1.0.0
  description: Logical schema for customer and order persistence.
systems:
  production:
    type: document
    engine: mongodb
    version: '8.0'
schemas:
  customer-data:
    customers:
      required:
        - id
      fields:
        id:
          type: string
```

#### <a name="openSchemaVersionString"></a>OpenSchema Version String

The version string signifies the version of the OpenSchema Specification that the document complies with. The format for this string MUST be `major`.`minor`.`patch`. The `patch` MAY be suffixed by a hyphen followed by pre-release identifiers, and MAY additionally be followed by a plus sign and build metadata identifiers, as defined by [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html). Pre-release and build metadata identifiers MAY contain alphanumeric characters, hyphens, and dots (`major`.`minor`.`patch`-`pre.release`+`build.metadata`).

#### <a name="openSchemaIdString"></a>Identifier

This field represents a unique identifier of the schema document. It MUST conform to the URI format according to [RFC3986](https://tools.ietf.org/html/rfc3986).

It is RECOMMENDED to use a [URN](https://tools.ietf.org/html/rfc8141) to globally and uniquely identify a schema over time.

##### Examples

```yaml
id: urn:oneweave:openschema:customer-data
```

```yaml
id: https://example.com/schemas/customer-data
```

#### <a name="infoObject"></a>Info Object

The object provides metadata about the schema document.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="infoObjectTitle"></a>title | `string` | **REQUIRED.** The title of the schema document.
<a name="infoObjectVersion"></a>version | `string` | **REQUIRED.** The version of the schema document contents defined by this OpenSchema file.
<a name="infoObjectDescription"></a>description | `string` | A short description of the schema. [CommonMark syntax](https://spec.commonmark.org/) MAY be used for rich text representation.
<a name="infoObjectTermsOfService"></a>termsOfService | `string` | A URL to the Terms of Service for the schema or tooling. This MUST be an absolute URL.
<a name="infoObjectContact"></a>contact | [Contact Object](#contactObject) | Contact information for the schema.
<a name="infoObjectLicense"></a>license | [License Object](#licenseObject) | The license information for the schema.
<a name="infoObjectTags"></a>tags | [Tags Object](#tagsObject) | A list of tags for logical grouping and discovery.
<a name="infoObjectExternalDocs"></a>externalDocs | [External Documentation Object](#externalDocumentationObject) | Additional external documentation.

This object MAY be extended with [Specification Extensions](#specificationExtensions).

##### Info Object Example

```yaml
title: Customer Data Model
version: 1.0.0
description: Neutral logical schema for customer and order storage.
contact:
  name: Schema Support
  email: schema@example.com
license:
  name: Apache 2.0
  url: https://www.apache.org/licenses/LICENSE-2.0.html
tags:
  - name: customer
  - name: orders
```

#### <a name="contactObject"></a>Contact Object

Contact information for the schema.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="contactObjectName"></a>name | `string` | The identifying name of the contact person or organization.
<a name="contactObjectUrl"></a>url | `string` | The URL pointing to the contact information. This MUST be an absolute URL.
<a name="contactObjectEmail"></a>email | `string` | The email address of the contact person or organization. MUST be in the format of an email address.

##### Contact Object Example

```yaml
name: Schema Support
url: https://example.com/support
email: schema@example.com
```

#### <a name="licenseObject"></a>License Object

License information for the schema.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="licenseObjectName"></a>name | `string` | **REQUIRED.** The license name used for the schema.
<a name="licenseObjectUrl"></a>url | `string` | A URL to the license used for the schema. This MUST be an absolute URL.

#### <a name="systemsObject"></a>Systems Object

The Systems Object is a map of [System Objects](#systemObject).

##### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
<a name="systemsObjectSystem"></a>`^[A-Za-z0-9_\-]+$` | [System Object](#systemObject) \| [Reference Object](#referenceObject) | A named concrete system where the schema MAY be applied.

#### <a name="systemObject"></a>System Object

An object representing a concrete database platform or target environment.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="systemObjectType"></a>type | `string` | A broad storage classification such as `relational`, `document`, `key-value`, or another implementation-defined value.
<a name="systemObjectEngine"></a>engine | `string` | The engine or product name, if known.
<a name="systemObjectVersion"></a>version | `string` | The engine or platform version.
<a name="systemObjectDescription"></a>description | `string` | A short description of the target system.

##### System Object Example

```yaml
production:
  type: document
  engine: mongodb
  version: '8.0'
  description: Primary MongoDB cluster for customer and order data.
```

#### <a name="schemasObject"></a>Schemas Object

The Schemas Object is a map of named [Schema Objects](#schemaObject).

##### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
`{schemaName}` | [Schema Object](#schemaObject) \| [Reference Object](#referenceObject) | A named logical schema. The map key is the schema identifier within the document and is case-sensitive. For maximum tooling compatibility, schema names SHOULD match `^[A-Za-z0-9_\-]+$`, but documents MUST NOT be considered invalid solely because a schema name uses a different character set.

#### <a name="schemaObject"></a>Schema Object

The Schema Object describes one logical schema. It contains named containers only and does not define schema-local metadata fields.

##### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
`{containerName}` | [Container Object](#containerObject) \| [Reference Object](#referenceObject) | A named logical container declared directly within the schema. Container names are case-sensitive.

##### Schema Object Example

```yaml
customers:
  required:
    - id
  fields:
    id:
      type: string
```

#### <a name="containersObject"></a>Containers Object

An object containing named [Container Objects](#containerObject) used in nested container locations, such as child containers under a container or field.

##### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
<a name="containersObjectContainer"></a>{containerName} | [Container Object](#containerObject) \| [Reference Object](#referenceObject) | A named logical container. The name is case-sensitive.

#### <a name="containerObject"></a>Container Object

An object describing a logical container of persisted records.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="containerObjectSummary"></a>summary | `string` | A short summary of the container.
<a name="containerObjectDescription"></a>description | `string` | A description of the container.
<a name="containerObjectPrimaryKey"></a>primaryKey | [`string`] | An ordered list of field names forming the logical primary key.
<a name="containerObjectIndexes"></a>indexes | [[Index Object](#indexObject)] | A list of secondary index definitions. Each index object MAY describe a single-field or compound index.
<a name="containerObjectRequired"></a>required | [`string`] | A list of required top-level fields.
<a name="containerObjectFields"></a>fields | [Fields Object](#fieldsObject) | **REQUIRED.** The fields stored in each record of the container.
<a name="containerObjectContainers"></a>containers | [Containers Object](#containersObject) | Child containers nested inside this container. Uses the same [Containers Object](#containersObject) type as the `containers` field on [Field Object](#fieldObject). See the note below.
<a name="containerObjectTags"></a>tags | [Tags Object](#tagsObject) | Tags for logical grouping and discovery.

This object MAY be extended with [Specification Extensions](#specificationExtensions).

> **Note (nested containers):** The `containers` field on `Container Object` and the `containers` field on `Field Object` use the same [Containers Object](#containersObject) type. They describe sub-collections owned by the parent: on a container this represents sub-collections owned by the record set itself; on a field this represents sub-collections embedded within a specific field value. Both are subject to the same constraint that the target system MUST support hierarchical or embedded storage for the field to have meaning.

> **Note (primaryKey vs. indexes):** `primaryKey` and `indexes` serve different purposes. `primaryKey` declares the logical identity of a record: each value of the primary key MUST be unique across all records in the container and MUST be present in every record (as though all primary-key fields are implicitly required). Each field name listed in `primaryKey` MUST correspond to a field declared in the container's `fields`. A conformant validator or parser MUST fail with an error when a `primaryKey` entry does not match any declared field. `indexes` declare secondary access paths that optimize queries or enforce uniqueness constraints beyond identity. A secondary index entry MAY be absent, sparse, or non-unique unless `unique: true` is set. Implementations MAY give `primaryKey` special physical storage treatment such as clustering or row ordering that is not applied to `indexes`.

> **Note (required and primaryKey field validation):** Each field name listed in `required` MUST correspond to a field declared in the container's `fields`. A conformant validator or parser MUST fail with an error when a name in `required` does not match any declared field.

> **Note (required and nullable interaction):** A field listed in `required` MUST be present in every record. If that field also declares `nullable: true`, the field key MUST be present but its value MAY be explicitly null. A `required` field without `nullable: true` MUST be present with a non-null value.

##### Container Object Example

```yaml
primaryKey:
  - id
required:
  - id
indexes:
  - fields:
      - name: status
  - fields:
      - name: customerId
        order: asc
      - name: createdAt
        order: desc
fields:
  id:
    type: string
  status:
    type: string
  address:
    type: object
    fields:
      city:
        type: string
      country:
        type: string
containers:
  invoices:
    fields:
      id:
        type: string
```

#### <a name="indexObject"></a>Index Object

An object describing a secondary index for a container.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
fields | [[Index Field Object](#indexFieldObject)] | **REQUIRED.** An ordered list of indexed field descriptors. More than one entry represents a compound index.
unique | `boolean` | Indicates whether the index enforces uniqueness across the indexed field set.
where | `string` | An optional predicate or filter expression for partial indexes when supported by the target system.
name | `string` | An optional implementation-facing index name.
description | `string` | A short description of the index.

##### Index Object Example

```yaml
fields:
  - name: customerId
    order: asc
  - name: createdAt
    order: desc
name: customer-created-at
```

#### <a name="indexFieldObject"></a>Index Field Object

An object describing one field position within an index.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
name | `string` | **REQUIRED.** The field name included in the index. This name MUST correspond to a field declared in the enclosing container's `fields`. A conformant validator or parser MUST fail with an error when this name does not match any declared field.
order | `string` | The index ordering for this field. MUST be `asc` or `desc` when specified. Values outside this set are not defined by this specification and MUST be expressed as [Specification Extensions](#specificationExtensions).

#### <a name="fieldsObject"></a>Fields Object

An object containing all [Field Objects](#fieldObject) defined for a container or structured field.

##### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
<a name="fieldsObjectField"></a>{fieldName} | [Field Object](#fieldObject) \| [Reference Object](#referenceObject) | A named field definition. The name is case-sensitive.

#### <a name="fieldObject"></a>Field Object

An object describing a single field.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
<a name="fieldObjectType"></a>type | `string` | **REQUIRED** unless the field is described by a `$ref`. The field type. Primitive examples include `string`, `number`, `boolean`, `date`, `datetime`, `bytes`, and `json`. Structured examples include `object` and `array`.
<a name="fieldObjectTitle"></a>title | `string` | A human-friendly title for the field.
<a name="fieldObjectDescription"></a>description | `string` | A description of the field.
<a name="fieldObjectNullable"></a>nullable | `boolean` | Indicates whether the field may explicitly contain a null value.
<a name="fieldObjectDefault"></a>default | Any | A default value for the field. This value MUST be type-compatible with the field's declared `type`.
<a name="fieldObjectEnum"></a>enum | [Any] | An enumeration of allowed values. Each value MUST be type-compatible with the field's declared `type`.
<a name="fieldObjectFormat"></a>format | `string` | An implementation hint for more specific typing. For `type: number`, this field SHOULD be used to express numeric representation or precision such as `int32`, `int64`, `float`, `double`, `decimal`, or `decimal128`. For other types, examples include `uuid` and `email`.
<a name="fieldObjectFields"></a>fields | [Fields Object](#fieldsObject) | Child fields. MUST only be present when `type` is `object`. When `type` is `object`, this field SHOULD be declared to describe the embedded structure. When omitted on an object-type field, the embedded object structure is not further constrained by the document (schemaless).
<a name="fieldObjectItems"></a>items | [Field Object](#fieldObject) \| [Reference Object](#referenceObject) | The item definition for repeated values. MUST only be present when `type` is `array`. Array fields are homogeneous: a single item type applies to all elements. Heterogeneous arrays are out of scope for this version of the specification.
<a name="fieldObjectContainers"></a>containers | [Containers Object](#containersObject) | Child containers embedded within this field's value. Uses the same [Containers Object](#containersObject) type as the `containers` field on [Container Object](#containerObject). This field is valid when the target system supports embedded or hierarchical storage. See the note on nested containers under [Container Object](#containerObject).

This object MAY be extended with [Specification Extensions](#specificationExtensions).

##### Field Object Examples

Primitive field:

```yaml
email:
  type: string
  format: email
```

Numeric field:

```yaml
totalCount:
  type: number
  format: int64
```

Structured object field:

```yaml
profile:
  type: object
  fields:
    firstName:
      type: string
    lastName:
      type: string
```

Array field:

```yaml
items:
  type: array
  items:
    type: object
    fields:
      sku:
        type: string
      quantity:
        type: number
        format: int32
```

#### <a name="tagsObject"></a>Tags Object

An ordered list of [Tag Objects](#tagObject) used for logical grouping and discovery of a schema, container, or document. The Tags Object is a named type so that the same list structure can be referenced consistently from multiple objects (Info Object, Container Object) without repeating the definition.

##### Example

```yaml
tags:
  - name: customer
    description: Records related to customer identity and profile data.
  - name: orders
    description: Records related to order lifecycle and line items.
```

Tag names within a single Tags Object SHOULD be unique. Tooling MAY use tag names to group or filter containers and schemas for documentation or code generation purposes.

#### <a name="tagObject"></a>Tag Object

Adds metadata for grouping and discovery.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
name | `string` | **REQUIRED.** The name of the tag.
description | `string` | A description for the tag.

#### <a name="externalDocumentationObject"></a>External Documentation Object

Allows referencing an external resource for extended documentation.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
description | `string` | A description of the target documentation.
url | `string` | **REQUIRED.** The URL for the target documentation. This MUST be an absolute URL.

#### <a name="componentsObject"></a>Components Object

Holds reusable definitions for OpenSchema documents.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
schemas | Map[`string`, [Schema Object](#schemaObject) \| [Reference Object](#referenceObject)] | Reusable schema definitions.
fields | Map[`string`, [Field Object](#fieldObject) \| [Reference Object](#referenceObject)] | Reusable field definitions.
containers | Map[`string`, [Container Object](#containerObject) \| [Reference Object](#referenceObject)] | Reusable container definitions.

> **Note:** The `schemas` key within `components` holds reusable schema fragments intended to be referenced via `$ref`. It is of the same [Schema Object](#schemaObject) type as the values in the root-level `schemas` map, but is intended for reuse via `$ref` rather than direct document-level declaration. It is distinct from the root-level `schemas` field, which declares the named logical schemas that the document describes.

> **Note (v0.1 known gap):** The `components` object does not provide an `indexes` slot. Index definitions are container-local and cannot be shared via `$ref` in this version of the specification.

##### Components Object Example

```yaml
components:
  fields:
    id:
      type: string
      format: uuid
    createdAt:
      type: string
      format: datetime
  containers:
    auditFields:
      fields:
        createdAt:
          $ref: '#/components/fields/createdAt'
        createdBy:
          type: string
schemas:
  customer-data:
    customers:
      fields:
        id:
          $ref: '#/components/fields/id'
        name:
          type: string
```

#### <a name="referenceObject"></a>Reference Object

A simple object that allows referencing another component in the document or an external document.

##### Fixed Fields

Field Name | Type | Description
---|:---:|---
$ref | `string` | **REQUIRED.** A URI reference to the target component.

##### Resolution

`$ref` values are resolved as follows:

- Internal references beginning with `#` are resolved against the root of the current document.
- Relative references are resolved relative to the base URI of the document. The base URI is the value of the root `id` field if present; otherwise it is the URI of the file from which the document was loaded.
- External references MUST use absolute URIs conforming to [RFC3986](https://tools.ietf.org/html/rfc3986).

Circular `$ref` chains MUST NOT be introduced by document authors. Tooling MAY report circular references as an error.

#### <a name="specificationExtensions"></a>Specification Extensions

While the OpenSchema Specification tries to remain storage-model neutral, implementations MAY need database-specific metadata. A field, container, schema, system, or root object MAY therefore be extended with implementation-specific properties beginning with `x-`.

Extensions MUST NOT change the meaning of standard OpenSchema fields. They SHOULD add supplementary behavior, generation hints, or vendor-specific metadata only.

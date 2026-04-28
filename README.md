# OpenSchema

OpenSchema is a specification for defining and validating structured data schemas in a machine-readable format. It is storage-model neutral and can describe relational, document, key-value, and other structured data systems without assuming a specific database architecture.

## Read the specification

The latest draft specification can be found at [spec/openschema.md](./openschema.md) which tracks the latest commit to the main branch in this repository.

**The human-readable markdown file is the source of truth for the specification.**

- [Version 0.1.0-draft](./openschema.md) (latest)

Looking for the JSON Schema file? See [spec/openschema.json](./openschema.json).

Feel like contributing? Check out our [contributor's guide](../CONTRIBUTING.md).

## Examples

Check out the [examples](../examples) directory for example OpenSchema documents:

- [bakery.yaml](../examples/bakery.yaml)
- [car-dealer.yaml](../examples/car-dealer.yaml)
- [music-store.yaml](../examples/music-store.yaml)
- [music-streaming-service.yaml](../examples/music-streaming-service.yaml)
- [university.yaml](../examples/university.yaml)

## Use Cases

OpenSchema can be used for:

- Documentation of database schemas (relational, document, key-value, and more)
- Schema validation and contract enforcement
- Code generation from a neutral schema definition
- Cross-system schema portability and migration planning
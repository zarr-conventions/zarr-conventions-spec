# Zarr Convention Specification

This repo describes the specification for Zarr Conventions. The keywords in this specification follow [RFC2119](https://www.rfc-editor.org/rfc/rfc2119).

## Definition

A **Zarr Convention** is a set of attributes on a Zarr Array or Group which confer special meaning about the data contained within. Conventions are defined via Array or Group `attributes`.
The key feature of Zarr Conventions is that they are _safely ignorable_ by low-level Zarr implementations. Conventions therefore may not change how data are encoded or stored; only how they are interpreted by the end user.

Examples of possible Conventions include:
- Geospatial projection information
- Multiscale arrays
- OME-Zarr

The _Zarr Conventions_ concept is inspired by the highly successful [STAC extensions](https://stac-extensions.github.io/) framework.

## Requirements

These are the requirements which motivated the conventions framework.

- Conventions must be uniquely identifiable.
- Conventions must be definable for both Arrays and Groups.
- Conventions must not require approval of a central authority (e.g. Zarr Steering Council, Zarr Implementation Council); anyone can create a Convention.
- Conventions should be self describing in terms of the specific metadata fields they contain.
- The conventions framework must support composability, allowing one Convention to extend or reference properties from another Convention.
- Convention properties must be accessible at a predictable location in the metadata structure.
- Minimal changes should be necessary for existing conventions to adopt this framework moving forward.

Non-requirements
- This specification addresses only _new data_; conventions do not retroactively apply to existing data written before the creation of the Conventions framework.
- No "implicit" Conventions; Conventions must be declared explicitly following this specification.
- No attempt to define which conventions will safely interoperate with which other conventions.

## Specification

The following applies to elements of a Zarr hierarchy, either Arrays or Groups (henceforth "nodes").

Nodes which conform to this specification MUST contain the following field within their `attributes`:
  - `zarr_conventions` - an array, described below.

### Convention Registration via `zarr_conventions`

The `zarr_conventions` attribute MUST be an array of Convention Metadata Objects (CMO). Each object in the array describes one Convention that applies to the node.

Each Convention Metadata Object MUST contain at least one of the following fields:
 - `uuid` - a UUID (Universally Unique Identifier) that permanently identifies the Convention, independent of URL changes.
 - `schema_url` - a URL which resolves to a JSON schema document which describes the Convention's properties.
 - `spec_url` - a URL which resolves to a document describing the Convention in more detail.

At least one of `schema_url`, `spec_url`, or `uuid` MUST be present. If `uuid` is present, it serves as the primary unique identifier for the Convention. If `uuid` is not present and both `schema_url` and `spec_url` are present, the `schema_url` serves as the identifier. If only `spec_url` is present (and no `uuid`), it serves as the identifier. If the Convention uses versioning, the `schema_url` SHOULD include a reference to the specific version (e.g., `v1`) and the `spec_url` MAY include a reference to the specific version. The Convention SHOULD provide a convenient way to find the specification associated with each version.

Additionally, a Convention Metadata Object SHOULD contain the following fields:
- `name` - a short human-readable name used to represent the Convention in contexts where such a name is desirable (e.g., websites). The name MUST NOT be used by tools to identify the Convention (use the `uuid`, `schema_url`, or `spec_url` instead). Names are not guaranteed to be unique across Conventions. If `name` is not present, tools SHOULD use the identifier (UUID or URL) instead to represent the Convention. If the Convention isolates metadata using [a namespace prefix or a key for nesting](#namespace-prefixes), the name SHOULD match the namespace prefix (e.g., `proj:`) or the key used to nest attributes (e.g., `ome`), depending on which method is used. When using namespace prefixing (in contrast to nesting), it is RECOMMENDED to include the colon in the name (e.g., `proj:` rather than `proj`) to clearly indicate the prefixing approach.
- `schema_url` and `spec_url` - when using `uuid` as the primary identifier, it is RECOMMENDED to also provide `schema_url` and/or `spec_url` for validation and documentation purposes.

The Convention Metadata Object MAY contain the following field:
- `description` - a concise description of the convention.

The Convention Metadata Object MUST NOT contain additional fields beyond those specified above.

#### Convention Identity

Conventions are uniquely identified by one of the following, in order of precedence:
1. `uuid` (if present) - provides a permanent, immutable identifier independent of URL changes
2. `schema_url` (if present and no `uuid`) - enables validation and serves as identifier
3. `spec_url` (if no `uuid` or `schema_url`) - provides human-readable documentation and serves as identifier

The `uuid` provides the most stable identification since it remains constant even if the Convention's hosting location or URLs change. When using a `uuid`, it is RECOMMENDED to also provide `schema_url` and/or `spec_url` for validation and documentation.

#### Versioning

Conventions MAY use a versioning scheme to allow evolution over time. Versioned conventions SHOULD include the version information in the `schema_url` (e.g., `https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json"` to facilitate validation of specific versions. Versioned conventions may include the version in the convention's own properties (not in the Zarr Conventions Metadata object) (`e.g., {'proj:version': "1"}`) for convenient version identification without url parsing. If a convention uses versioning, it MUST clearly define the semantic meaning of version numbers in its specification.

### Convention Properties

Convention properties exist at the root `attributes` level. This enables composability, allowing conventions to extend and reference properties from other conventions.

#### Namespace Prefixes

For conventions which wish to avoid collisions with other conventions, the properties SHOULD either:

- use namespace prefixes on individual attributes (e.g., `proj:`) OR
- nest their properties under a single attribute key (e.g. `ome`)

For example, the following uses a prefix on individual attributes:


```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "2dc8d146-3932-4e08-8542-06aa0e826508",
        "schema_url": "https://raw.githubusercontent.com/zarr-experimental/proj-prefix/refs/tags/v1/schema.json",
        "spec_url": "https://github.com/zarr-experimental/proj-prefix/blob/v1/README.md",
        "name": "proj:",
        "description": "Coordinate reference system information for geospatial data, using prefix namespacing."
      }
    ],
    "proj:code": "EPSG:4326"
  }
}
```

In contrast, the following hypothetical convention nests metadata under a single attribute key:

```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "ef154843-db6c-41c3-8ccf-64294a8fa889",
        "schema_url": "https://raw.githubusercontent.com/zarr-experimental/proj-nested-key/refs/tags/v1/schema.json",
        "spec_url": "https://github.com/zarr-experimental/proj-nested-key/blob/v1/README.md",
        "name": "proj",
        "description": "Coordinate reference system information for geospatial data, using keyed namespacing."
      }
    ],
    "proj": {"code": "EPSG:4326"}
  }
}
```

#### Composability

Conventions can extend objects defined by other conventions by adding namespaced properties. For example, a geospatial projection Convention (`proj:`) can add projection-specific properties to items within a multiscales layout:

```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "schema_url": "https://raw.githubusercontent.com/zarr-conventions/multiscales/refs/tags/v1/schema.json",
        "spec_url": "https://github.com/zarr-conventions/multiscales/blob/v1/README.md",
        "uuid": "d35379db-88df-4056-af3a-620245f8e347",
        "name": "multiscales",
        "description": "Multiscale layout of zarr datasets"
      },
      {
        "schema_url": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json",
        "spec_url": "https://github.com/zarr-experimental/geo-proj/blob/v1/README.md",
        "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f",
        "name": "proj:",
        "description": "Coordinate reference system information for geospatial data"
      }
    ],
    "multiscales": {
      "layout": [
        {
          "asset": "r10m",
          "transform": {
            "scale": [1.0, 1.0],
            "translation": [0.0, 0.0]
          },
          "proj:shape": [1200, 1200],
          "proj:transform": [10.0, 0.0, 500000.0, 0.0, -10.0, 5000000.0]
        },
        {
          "asset": "r20m",
          "derived_from": "r10m",
          "transform": {
            "scale": [2.0, 2.0],
            "translation": [0.0, 0.0]
          },
          "proj:shape": [600, 600],
          "proj:transform": [20.0, 0.0, 500000.0, 0.0, -20.0, 5000000.0]
        }
      ]
    }
  }
}
```

If a Convention wants to allow other conventions to extend its objects, its schema MUST set `additionalProperties: true` on those extensible objects. Since `additionalProperties` defaults to `false` in JSON Schema, conventions that do not explicitly set this to `true` will not support being extended by other conventions. Note that the Convention Metadata Object itself (the object within the `zarr_conventions` array) is not extensible and MUST keep `additionalProperties: false`; only the convention's own properties nested directly under the root `attributes` level can allow additional properties for composability.

## Examples


Recommended Convention (with uuid plus schema_url and spec_url):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f",
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json",
                "spec_url": "https://github.com/zarr-experimental/geo-proj/blob/v1/README.md",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data"
            }
        ],
        "proj:code": "EPSG:4326",
        "proj:spatial_dimensions": ["Y", "X"],
        "proj:transform": [1.0, 0.0, 0.0, 0.0, -1.0, 90.0]
    }
}
```

Minimum conformant hypothetical Convention (with uuid):

```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "zarr_conventions": [
            {
                "uuid": "2dc8d146-3932-4e08-8542-06aa0e826508"
            }
        ]
    }
}
```

Minimum conformant hypothetical Convention (with schema_url):

```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "zarr_conventions": [
            {
                "schema_url": "https://example.com/my-convention/v1/schema.json"
            }
        ]
    }
}
```

Minimum conformant hypothetical Convention (with spec_url only):

```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "zarr_conventions": [
            {
                "spec_url": "https://example.com/my-convention/v1/README.md"
            }
        ]
    }
}
```

Minimum conformant Convention (with uuid):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f"
            }
        ]
    }
}
```

More complete example with projection information:

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f",
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec_url": "https://github.com/zarr-experimental/geo-proj/blob/v1/README.md"
            }
        ],
        "proj:code": "EPSG:4326",
        "proj:transform": [1.0, 0.0, 0.0, 0.0, -1.0, 90.0]
    }
}
```

Example demonstrating composability with multiscales and projection:

```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "zarr_conventions": [
            {
                "uuid": "d35379db-88df-4056-af3a-620245f8e347",
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/multiscales/refs/tags/v1/schema.json",
                "spec_url": "https://github.com/zarr-experimental/multiscales/blob/v1/README.md",
                "name": "multiscales",
                "description": "Multiscale layout of zarr datasets"
            },
            {
                "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f",
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json",
                "spec_url": "https://github.com/zarr-experimental/geo-proj/blob/v1/README.md",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data"
            }
        ],
        "multiscales": {
            "layout": [
                {
                    "asset": "r10m",
                    "transform": {
                        "scale": [1.0, 1.0],
                        "translation": [0.0, 0.0]
                    },
                    "proj:shape": [1200, 1200],
                    "proj:transform": [10.0, 0.0, 500000.0, 0.0, -10.0, 5000000.0]
                },
                {
                    "asset": "r20m",
                    "derived_from": "r10m",
                    "transform": {
                        "scale": [2.0, 2.0],
                        "translation": [0.0, 0.0]
                    },
                    "proj:shape": [600, 600],
                    "proj:transform": [20.0, 0.0, 500000.0, 0.0, -20.0, 5000000.0]
                }
            ]
        },
        "proj:code": "EPSG:32633",
        "proj:spatial_dimensions": ["Y", "X"],
        "proj:bbox": [500000.0, 4900000.0, 600000.0, 5000000.0]
    }
}
```

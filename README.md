# Zarr Convention Specification

This repo describes the specification for Zarr Conventions. The keywords in this specification follow [RFC2119](https://www.rfc-editor.org/rfc/rfc2119).

## Definition

A **Zarr convention** is a set of attributes on a Zarr Array or Group which confer special meaning about the data contained within. Conventions are defined via Array or Group `attributes`.
The key feature of Zarr conventions is that they are _safely ignorable_ by low-level Zarr implementations. Conventions therefore may not change how data are encoded or stored; only how they are interpreted by the end user.

Examples of possible Conventions include:
- Geospatial projection information
- Multiscale arrays
- OME-Zarr

The _zarr conventions_ concept is inspired by the highly successful [STAC extensions](https://stac-extensions.github.io/) framework.

## Requirements

These are the requirements which motivated the conventions framework.

- Conventions must be uniquely identifiable.
- Conventions must be definable for both Arrays and Groups.
- Conventions must not require approval of a central authority (e.g. Zarr Steering Council, Zarr Implementation Council); anyone can create a Convention
- Conventions should be self describing in terms of the specific metadata fields they contain.
- The conventions framework must support composability, allowing one convention to extend or reference properties from another convention.
- Convention properties must be accessible at a predictable location in the metadata structure.

Non-requirements
- This specification addresses only _new data_; conventions do not retroactively apply to existing data written before the creation of the Conventions framework.
- No "implicit" Conventions; Conventions must be declared explicitly following this specification.
- No attempt to define which conventions will safely interoperate with which other conventions.

## Specification

The following applies to elements of a Zarr hierarchy, either Arrays or Groups (henceforth "nodes").

Nodes which conform to this specification MUST contain the following field within their `attributes`:
  - `zarr_conventions` - an array, described below.

### Convention Registration via `zarr_conventions`

The `zarr_conventions` attribute MUST be an array of convention metadata objects. Each object in the array describes one convention that applies to the node.

Each convention metadata object MUST contain at least one of the following fields:
 - `schema_url` - a URL which resolves to a JSON schema document which describes the convention's properties.
 - `spec_url` - a URL which resolves to a document describing the Convention in more detail.
 - `uuid` - a UUID (Universally Unique Identifier) that permanently identifies the convention, independent of URL changes.

At least one of `schema_url`, `spec_url`, or `uuid` MUST be present. If `uuid` is present, it serves as the primary unique identifier for the convention. If `uuid` is not present and both `schema_url` and `spec_url` are present, the `schema_url` serves as the identifier. If only `spec_url` is present (and no `uuid`), it serves as the identifier. If the convention uses versioning, the `schema_url` SHOULD include a reference to the specific version (e.g., `v1`) and the `spec_url` MAY include a reference to the specific version. The convention SHOULD provide a convenient way to find the specification associated with each version.

Additionally, a convention metadata object SHOULD contain the following fields:
- `name` - a short human-readable name used to represent the Convention in contexts where such a name is desirable (e.g., websites). The name MUST NOT be used by tools to identify the Convention (use the `uuid`, `schema_url`, or `spec_url` instead). Names are not guaranteed to be unique across Conventions. If `name` is not present, tools SHOULD use the identifier (UUID or URL) instead to represent the Convention. If the convention isolates metadata using [a namespace prefix or a key for nesting](#namespace-prefixes), the name SHOULD match the namespace prefix (e.g., `proj:`) or the key used to nest attributes (e.g., `ome`), depending on which method is used. When using namespace prefixing (in contrast to nesting), it is RECOMMENDED to include the colon in the name (e.g., `proj:` rather than `proj`) to clearly indicate the prefixing approach.
- `schema_url` and `spec_url` - when using `uuid` as the primary identifier, it is RECOMMENDED to also provide `schema_url` and/or `spec_url` for validation and documentation purposes.

The convention metadata object MAY contain the following field:
- `description` - a concise description of the convention.

The convention metadata object MUST NOT contain additional fields beyond those specified above.

#### Convention Identity

Conventions are uniquely identified by one of the following, in order of precedence:
1. `uuid` (if present) - provides a permanent, immutable identifier independent of URL changes
2. `schema_url` (if present and no `uuid`) - enables validation and serves as identifier
3. `spec_url` (if no `uuid` or `schema_url`) - provides human-readable documentation and serves as identifier

The `uuid` provides the most stable identification since it remains constant even if the convention's hosting location or URLs change. When using a `uuid`, it is RECOMMENDED to also provide `schema_url` and/or `spec_url` for validation and documentation.

#### Versioning

It is RECOMMENDED that conventions use a versioning scheme to allow evolution over time. Versioned conventions SHOULD include the version information in the `schema_url` (e.g., `https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json"` to facilitate validation of specific versions. Versioned conventions MAY also include the version in the convention metadata (`e.g., {'proj:version': "1"}`) for convenient version identification without url parsing. If a convention uses versioning, it MUST clearly define the semantic meaning of version numbers in its specification.

### Convention Properties

Convention properties exist at the root `attributes` level. This enables composability, allowing conventions to extend and reference properties from other conventions.

#### Namespace Prefixes

For conventions which wish to avoid collisions with other conventions, the properties SHOULD either 
- use namespace prefixes on individual attributes (e.g., `proj:`) OR
- nest their properties under a single attribute key (e.g. `ome`)

For example:

- `proj:code` - projection code
- `proj:bbox` - bounding box in projected coordinates


#### Composability

Conventions can extend objects defined by other conventions by adding namespaced properties. For example, a geospatial projection convention (`proj:`) can add projection-specific properties to items within a multiscales layout:

```json
{
  "multiscales": {
    "layout": [
      {
        "group": "r10m",
        "proj:shape": [1200, 1200],
        "proj:transform": [10.0, 0.0, 500000.0, 0.0, -10.0, 5000000.0]
      }
    ]
  }
}
```

If a convention wants to allow other conventions to extend its objects, its schema MUST set `additionalProperties: true` on those extensible objects. Since `additionalProperties` defaults to `false` in JSON Schema, conventions that do not explicitly set this to `true` will not support being extended by other conventions.

## Examples

Minimum conformant Convention (with schema_url):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "schema_url": "https://example.com/my-convention/v1/schema.json"
            }
        ]
    }
}
```

Minimum conformant Convention (with spec_url only):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "spec_url": "https://example.com/my-convention/v1/README.md"
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
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/multiscales/refs/tags/v1/schema.json",
                "name": "multiscales",
                "description": "Multiscale layout of zarr datasets",
                "spec_url": "https://github.com/zarr-experimental/multiscales/blob/v1/README.md"
            },
            {
                "schema_url": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v1/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec_url": "https://github.com/zarr-experimental/geo-proj/blob/v1/README.md"
            }
        ],
        "multiscales": {
            "layout": [
                {
                    "group": "r10m",
                    "proj:shape": [1200, 1200],
                    "proj:transform": [10.0, 0.0, 500000.0, 0.0, -10.0, 5000000.0]
                },
                {
                    "group": "r20m",
                    "from_group": "r10m",
                    "scale": [2.0, 2.0],
                    "proj:shape": [600, 600],
                    "proj:transform": [20.0, 0.0, 500000.0, 0.0, -20.0, 5000000.0]
                }
            ]
        },
        "proj:code": "EPSG:32633",
        "proj:bbox": [500000.0, 4900000.0, 600000.0, 5000000.0]
    }
}
```

# Implementations

- GDAL: ZARR: add a prototype read implementation of the geo/proj attribute conventions: https://github.com/OSGeo/gdal/pull/13215

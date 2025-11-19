# Zarr Convention Specification

This repo describes the specification for Zarr Conventions.

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
 - `schema` - a URL which resolves to a JSON schema document which describes the convention's properties.
 - `spec` - a URL which resolves to a document describing the Convention in more detail.

At least one of `schema` or `spec` MUST be present. If both are present, the `schema` URL serves as the primary unique identifier for the convention. If only `spec` is present, it serves as the unique identifier.

Additionally, a convention metadata object SHOULD contain the following field:
- `name` - a short human-readable name used to represent the Convention in contexts where such a name is desirable (e.g websites). The name MUST NOT be used by tools to identify the Convention (use the `schema` or `spec` URL instead). Names are not guaranteed to be unique across Conventions. If `name` is not present, tools SHOULD use the identifier URL instead to represent the Convention.

The convention metadata object MAY contain the following field:
- `description` - a concise description of the convention.

The convention metadata object MUST NOT contain additional fields beyond those specified above.

#### Convention Identity and Versioning

Conventions are uniquely identified by their identifier URL, which is either the `schema` URL (if present) or the `spec` URL. If both are present, the `schema` URL takes precedence as the primary identifier.

The identifier URL MAY include version information (e.g., via git tags or versioned paths) to allow conventions to evolve over time.

When a convention evolves:
- Breaking changes SHOULD result in a new identifier URL (e.g., incrementing the major version in the URL path)
- Non-breaking changes MAY update the schema/spec at the same URL or use a new URL (e.g., incrementing the minor version)

Tools consuming conventions SHOULD use the identifier URL to determine which convention is being used and which version of that convention applies.

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

Minimum conformant Convention (with schema):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "schema": "https://example.com/my-convention/v0.1.0/schema.json"
            }
        ]
    }
}
```

Minimum conformant Convention (with spec only):

```json
{
    "zarr_format": 3,
    "node_type": "array",
    "attributes": {
        "zarr_conventions": [
            {
                "spec": "https://example.com/my-convention/v0.1.0/README.md"
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
                "schema": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v0.1.0/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec": "https://github.com/zarr-experimental/geo-proj/blob/v0.1.0/README.md"
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
                "schema": "https://raw.githubusercontent.com/zarr-experimental/multiscales/refs/tags/v0.1.0/schema.json",
                "name": "multiscales",
                "description": "Multiscale layout of zarr datasets",
                "spec": "https://github.com/zarr-experimental/multiscales/blob/v0.1.0/README.md"
            },
            {
                "schema": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v0.1.0/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec": "https://github.com/zarr-experimental/geo-proj/blob/v0.1.0/README.md"
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

## Relationship with Extensions

The Zarr V3 spec defines five specific [Array extension points](https://zarr-specs.readthedocs.io/en/latest/v3/core/index.html#extension-points): data type, chunk grid, chunk key encoding, codecs, and storage transformers.
This specification covers situations not addressed by those extension points.

Additionally, the concept of "Generic Extensions" has been proposed in [ZEP-9](https://zarr.dev/zeps/draft/ZEP0009.html), but progress towards acceptance was deliberately paused.
Discussion led to the conclusion that many of the use cases for generic extensions could potentially be implemented in a more light-weight way solely using `attributes`.
This specification is a response to that suggestion.

A user should be able to safely ignore a convention and still be able to interact with the data via a Zarr library, even if some domain-specific context or functionality is missing.
If the data are completely meaningless or unintelligible without the convention, then it should probably be an extension instead.

It is an explicit goal of the Conventions effort to prototype a working process for decentralized, community-driven development of new Zarr-related functionality in a "low stakes" context (constrained just to the `attributes` field).
The lessons learned from this may be applied to the more formal Extensions framework in the future.

# FAQ

- Why isn't `zarr_conventions` a top-level metadata key?
  - Creating a new top-level key would require a new extension point in Zarr, which in turn requires a lengthy ZEP process. 
  - By putting `zarr_conventions` in attributes, we affirm that it is outside of the purview of the Zarr spec itself; these conventions are purely "adopter" metadata.
  - Once the `zarr_conventions` mechanism has matured, we may attempt to promote it to a top-level metadata key in the future.

- How should we handle pre-existing "implicit conventions" like [ome-zarr](https://ome-zarr.readthedocs.io/en/stable/) or Xarray's `_FillValue` encoding?
  - Our goal here is to create a robust conventions mechanism that will serve the community for many years to come. That is incompatible with also supporting implicit conventions.
  - Instead, it is possible to document these conventions as "registered attributes" in the zarr-extensions repo.

- What is the mechanisms for discovery of conventions?
  - The `zarr-conventions` repository provides an [optional](https://github.com/zarr-conventions#share-your-convention) central place for the community to maintain and discover conventions. The [`zarr-conventions/template`](https://github.com/zarr-conventions/template) repo makes it easy to author new conventions. 
  - In the future, we may create a website similar to [STAC extensions](https://stac-extensions.github.io/).

- What if you don't want a convention to be discoverable?
  - Given that the URL should be resolvable, this spec does not support opaque conventions.
 
- Do Zarr implementations need to understand `zarr_conventions`?
  - No; by definition, conventions cannot modify core Zarr behavior (this requires extensions). They sit purely "on top" of the Zarr data model.
  - That said, implementations may choose to interpret the conventions and implement related features. 
  - Or this can be left to higher-level libraries sitting on top of the Zarr implementation (e.g. Xarray.)

- Can conventions be used for arrays or groups?
  - They can be used for either or both.

- Why is schema or spec required?
  - At least one URL is required to serve as the unique identifier for the convention. The schema URL provides validation capabilities, while the spec URL provides human-readable documentation. Having at least one ensures conventions are both identifiable and discoverable.

- Should I provide both schema and spec?
  - It's recommended to provide both when possible. The schema enables validation and tooling support, while the spec provides human-readable documentation. If you can only provide one, choose based on your use case: schema for machine validation, spec for human consumption.

- How do I version my convention?
  - Include version information in your convention's metadata or the schema or spec URL (e.g., using git tags like `/v0.1.0/` or versioned paths). When you make breaking changes, update to a new major version in the URL.

- Why is name recommended?
  - We will use the name to populate the website showing all conventions; tools also will use it for nice metadata representations.

- Why is name not required?
  - Name should only be used for human-oriented displays, it isn't required because it isn't strictly necessary for a convention to be useful.


- Why use schema or spec URLs instead of UUIDs for identification?
  - URLs provide a mechanism for both identification and discovery, simplifying the specification. Schema URLs enable validation, while spec URLs point to documentation. Both naturally encode version information and provide a discoverable path to more information. While this requires conventions to maintain stable URLs, this is consistent with common practices in web standards.

- How does this specification enable composability?
  - Convention properties exist at the root `attributes` level (not isolated in the `zarr_conventions` array), allowing conventions to reference and extend properties from other conventions. This mirrors the proven pattern used by STAC extensions. For example, a projection convention can add projection-specific properties to items within a multiscales layout.
  
- How are property name collisions prevented?
  - Conventions use namespace prefixes (e.g., `proj:`, `ome:`) to prevent collisions. This approach has been successfully used by the STAC community for years without collision issues. Convention authors coordinate on naming within their domain.

- What if my schema or spec URL becomes unavailable?
  - Convention authors should use stable hosting for URLs (e.g., GitHub releases, permanent DOIs). Tools consuming conventions should implement caching strategies and graceful degradation when URLs are temporarily unavailable. The convention properties remain valid even if the identifier URL is unreachable. Convention authors may register a new convention conveying similar information if an identifying URL is irreconcilably lost, and should coordinate with downstream libraries about the change.

# Implementations

- GDAL: ZARR: add a prototype read implementation of the geo/proj attribute conventions: https://github.com/OSGeo/gdal/pull/13215

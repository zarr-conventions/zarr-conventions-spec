# Zarr Convention Specification

This repo describes the specification for Zarr Conventions

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
- Convention metadata attributes must not be able to collide or conflict with metadata attributes of other conventions.
- Conventions must support composability, allowing one convention to extend or reference properties from another convention.
- Convention properties must be accessible at a predictable location in the metadata structure.

Non-requirements
- This specification addresses only _new data_; conventions do not retroactively apply to existing data written before the creation of the Conventions framework.
- No "implicit" Conventions; Conventions must be declared explicitly following this specification.
- No attempt to define which conventions will safely interoperate with which other conventions.

## Specification

The following applies to elements of a Zarr hierarchy, either Arrays or Groups (henceforth "nodes").

Nodes which conform to this specification MUST contain the following fields within their `attributes`:
  - `zarr_conventions_version` - a semver-compatible string indicating the version of _this specification_. The current version is `0.1.0`.
  - `zarr_conventions` - an object, described below.

### Convention Registration via `zarr_conventions`

Each convention is uniquely identified by a [UUID](https://www.rfc-editor.org/rfc/rfc9562.html).
When creating a new convention, the creator MUST use the [UUID4 function](https://www.rfc-editor.org/rfc/rfc9562.html#name-uuid-version-4) to generate a unique identifier for their convention.

The `zarr_conventions` object contains convention registration metadata. Each key MUST be a UUID string identifying a convention, and each value MUST be a "convention metadata object" (described below).

Once created, the convention's UUID MUST NOT change.
(Evolution of the convention instead is managed via `version`, described below.)

Each convention metadata object MUST contain the following field:
 - `version` - a semver-compatible string indicating the version of the convention. New conventions should adopt `0.1.0` as their starting version number.

Additionally, a convention metadata object SHOULD contain the following fields:
- `name` - a short human-readable name used to represent the Convention in contexts where such a name is desirable (e.g websites). The name MUST NOT be used by tools to identify the Convention (use the UUID instead). Names are not guaranteed to be unique across Conventions. If `name` is not present, tools SHOULD use the UUID instead to represent the Convention.
- `schema` - a URL which resolves to a JSON schema document which describes the convention's properties.

The convention metadata object MAY contain the following fields:
- `spec` - a URL which resolves to a document describing the Convention in more detail.
- `description` - a concise description of the convention.

The convention metadata object MUST NOT contain additional fields.

### Convention Properties

Convention properties exist at the root `attributes` level. This enables composability, allowing conventions to extend and reference properties from other conventions.

#### Namespace Prefixes

For conventions which wish to avoid collisions with other conventions, the properties SHOULD either 
- use namespace prefixes on individual attributes (e.g., `proj:`) OR
- nest their properties under a single attribute key (e.g. `ome`)

For example:

- `proj:code` - projection code
- `proj:bbox` - bounding box in projected coordinates

#### Domain-Agnostic Convention Objects

Conventions that define a domain-agnostic object (such as `multiscales`) MAY use an unprefixed name for that object. Properties within such objects follow the convention's internal naming scheme. Those conventions MUST let additional properties be added by other conventions to support composability.

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

Minimum conformant Convention.

```json
{
    "attributes": {
        "zarr_conventions_version": "0.1.0",
        "zarr_conventions": {
            "0396f4cd-47fa-4b09-8c79-9072d90ceed3": {
                "version": "0.1.0"
            }
        }
    }
}
```

More complete example with projection information:

```json
{
    "attributes": {
        "zarr_conventions_version": "0.1.0",
        "zarr_conventions": {
            "f17cb550-5864-4468-aeb7-f3180cfb622f": {
                "version": "0.1.0",
                "schema": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v0.1.0/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec": "https://github.com/zarr-experimental/geo-proj/blob/v0.1.0/README.md"
            }
        },
        "proj:code": "EPSG:4326",
        "proj:transform": [1.0, 0.0, 0.0, 0.0, -1.0, 90.0]
    }
}
```

Example demonstrating composability with multiscales and projection:

```json
{
    "attributes": {
        "zarr_conventions_version": "0.1.0",
        "zarr_conventions": {
            "d35379db-88df-4056-af3a-620245f8e347": {
                "version": "0.1.0",
                "schema": "https://raw.githubusercontent.com/zarr-experimental/multiscales/refs/tags/v0.1.0/schema.json",
                "name": "multiscales",
                "description": "Multiscale layout of zarr datasets",
                "spec": "https://github.com/zarr-experimental/multiscales/blob/v0.1.0/README.md"
            },
            "f17cb550-5864-4468-aeb7-f3180cfb622f": {
                "version": "0.1.0",
                "schema": "https://raw.githubusercontent.com/zarr-experimental/geo-proj/refs/tags/v0.1.0/schema.json",
                "name": "proj:",
                "description": "Coordinate reference system information for geospatial data",
                "spec": "https://github.com/zarr-experimental/geo-proj/blob/v0.1.0/README.md"
            }
        },
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

- Why isn't `zarr_conventions_metadata` a top-level metadata key?
  - Creating a new top-level key would require a new extension point in Zarr, which in turn requires a lengthy ZEP process. 
  - By putting `zarr_conventions_metadata` in attributes, we affirm that it is outside of the purview of the Zarr spec itself; these conventions are purely "user" metadata.
  - Once the `zarr_conventions_metadata` mechanism has matured, we may attempt to promote it to a top-level metadata key in the future.

- How should we handle pre-existing "implicit conventions" like [ome-zarr](https://ome-zarr.readthedocs.io/en/stable/) or Xarray's `_FillValue` encoding?
  - Our goal here is to create a robust conventions mechanism that will serve the community for many years to come. That is incompatible with also supporting implicit conventions.
  - Instead, it is possible to document these conventions as "registered attributes" in the zarr-extensions repo.

- What is the mechanisms for discovery of conventions?
  - The `zarr-conventions` repository provides an [optional](https://github.com/zarr-conventions#share-your-convention) central place for the community to maintain and discover conventions. The [`zarr-conventions/template`](https://github.com/zarr-conventions/template) repo makes it easy to author new conventions. 
  - In the future, we may create a website similar to [STAC extensions](https://stac-extensions.github.io/).

- What if you don't want a convention to be discoverable?
  - This mechanism supports "opaque" conventions. It is not required to publish a schema or a spec. A convention is identified by a UUID.
 
- Do Zarr implementations need to understand `zarr_conventions_metadata`?
  - No; by definitinion, conventions cannot modify core Zarr behavior (this requires extensions). They sit purely "on top" of the Zarr data model.
  - That said, implementations may choose to interpret the conventions and implement related features. 
  - Or this can be left to higher-level libraries sitting on top of the Zarr implementation (e.g. Xarray.)

- Can conventions be used for arrays or groups?
  - They can be used for either or both.

- Why is version required?
  - UUIDs are version-independent; version provides a way for conventions to evole and implementations to track that evolution.

- Do I need to change my UUID when I change the schema?
  - No, UUIDs should be permanent

- Why is schema recommended?
  - It provides a way to validate your convention
  
- Why is schema not required?
  - In order to support conventions that are designed to be private and therefore do not have a publicly resolvable schema.

- Why is name recommended?
  - We use it to populate the website showing all conventions; tools also use it for nice metadata representations.

- Why is name not required?
  - Name should only be used for human-oriented displays, it isn't required because it isn't strictly necessary for a convention to b useful.

- Why do you use a UUID for the key?
    - It provides a decentralized version for guaranteed uniqueness

- How does this specification enable composability?
  - Convention properties exist at the root `attributes` level (not isolated in `configuration` fields), allowing conventions to reference and extend properties from other conventions. This mirrors the proven pattern used by STAC extensions. For example, a projection convention can add projection-specific properties to items within a multiscales layout.
  
- How are property name collisions prevented?
  - Conventions use namespace prefixes (e.g., `proj:`, `ome:`) to prevent collisions. This approach has been successfully used by the STAC community for years without collision issues. Convention authors coordinate on naming within their domain.

# Implementations

- GDAL: ZARR: add a prototype read implementation of the geo/proj attribute conventions: https://github.com/OSGeo/gdal/pull/13215

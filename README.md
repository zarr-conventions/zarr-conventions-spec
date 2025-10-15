# Zarr Convention Specification

This repo describes the specification for Zarr Conventions

## Definition

A **Zarr convention** is a set of attributes on a Zarr Array or Group which confer special meaning to the data contained within. Conventions are defined via Array or Group `attributes`.
The key feature of Zarr conventions is that they are _safely ignorable_ by low-level Zarr implementations. Conventions therefore may not change how data are encoded or stored; only how they are interpreted by the end user.

Examples of possible Conventions include:
- Geospatial projection information
- Multiscale arrays
- OME-Zarr

The _zarr conventions_ concept is inspired by the highly successful [STAC extensions](https://stac-extensions.github.io/) framework.

## Relationship with Extensions

The Zarr V3 spec defines five specific [Array extension points](https://zarr-specs.readthedocs.io/en/latest/v3/core/index.html#extension-points): data type, chunk grid, chunk key encoding, codecs, and storage transformers.
This specification covers situations not addressed by those extension points.

Additionally, the concept of "Generic Extensions" has been proposed in [ZEP-9](), but not yet implemented.
Subsequent discussion led to the conclusion that many of the use cases for generic extensions could potentially be implemented in a more light-weight way solely using `attributes`.
This specification is a response to that suggestion.

A user should be able to safely ignore a convention and still be able to interact with the data via a Zarr library, even if some domain-specific context or functionality is missing.
If the data are completely meaningless or unintelligible without the convention, then it should probably be an extension instead.

It is an explicit goal of the Conventions effort to prototype a working process for decentralized, community-driven development of new Zarr-related functionality in a "low stakes" context (constrained just to the `attributes` field).
The lessons learned from this may be applied to the more formal Extensions framework in the future

## Requirements

These are the requirements which motivated the conventions framework.

- Conventions must be uniquely identifiable.
- Conventions must be definable for both Arrays and Groups.
- Conventions must not require approval of a central authority (e.g. Zarr Steering Council, Zarr Implementation Council); anyone can create a Convention
- Conventions should be self describing in terms of the specific metadata fields they contain.
- Convention metadata must not be able to collide or conflict with other conventions.

Non-requirements
- This specification addresses only _new data_; conventions do not retroactively apply to existing data written before the creation of the Conventions framework.
- No "implicit" Conventions; Conventions must be declared explicitly following this specification.

## Specification

The following applies to elements of a Zarr hierarchy, either Arrays or Groups (henceforth "nodes").

Nodes which conform to this specification MUST contain the following fields within their `attributes`:
  - `zarr_conventions_version` - a semver-compatible string indicating the version of _this specification_. The current version is `0.1.0`.
  - `zarr_conventions_metadata` - an object, described below.

Each convention is uniquely identified by a [UUID](https://www.rfc-editor.org/rfc/rfc9562.html).
When creating a new convention, the creator MUST use the [UUID4 function](https://www.rfc-editor.org/rfc/rfc9562.html#name-uuid-version-4) to generate a unique identifier for their convention.
The `zarr_conventions_metadata` object MUST contain only UUIDs as keys, encoded as strings.
The values are called a "convention metadata object".
Once created, the convention's UUID MUST NOT change.
(Evolution of the convention instead is managed via `version`, described below.)

Each convention metadata object MUST contain the following fields
 - `version` - a semver-compatible string indicating the version of the convention. New conventions should adopt `0.1.0` as their starting version number.
 - `configuration` - an object containing the attributes described by the convention.

Additionally, a convention metadata object SHOULD contain the following fields:
- `name` - a short human-readable name used to represent the Convention in contexts where such a name is desirable (e.g websites). The name MUST NOT be used by tools to identify the Convention (use the UUID instead). Names are not guaranteed to be unique across Conventions. If `name` is not present, tools SHOULD use the UUID instead to represent the Convention. 
- `schema` - a URL which resolves to a JSON schema document which describes the convention's `configuration` object.

The conventions metadata object MAY contain the following fields:
- `spec` - a URL which resolves to a document describing the Convention in more detail.
- `description`: a concise description of the convention.

The conventions metadata object MUST NOT contain additional fields.

## Examples

Minimum comformant Convention.

```json
{
    "attributes": {
        "zarr_conventions_version": "0.1.0",
        "zarr_conventions_metadata": {
            "0396f4cd-47fa-4b09-8c79-9072d90ceed3": {
                "version": "0.1.0",
                "configuration": {}
            }
    }
}

```


More verbose example:

```json
{
    "attributes": {
        "zarr_conventions_version": "0.1.0",
        "zarr_conventions_metadata": {
            "f010a634-3525-416e-9320-8f44b5bc352c": {
                "version": "0.1.0",
                "schema": "https://github.com/EOPF-Explorer/data-model/v0.1.0/attributes/geo/proj/schema.json",
                "name": "geo:proj",
                "description": "Geospatial project information",
                "spec": "https://github.com/EOPF-Explorer/data-model/v0.1.0/README.md",
                "configuration": {
                    "code": "epsg:4326",
                    "transform": [0, 0, 1, 1, 1, 1]
                }
            }
        }
    }
}
```

# FAQ

- Why isn't `zarr_conventions_metadata` a top-level metadata key?
  - Creating a new top-level key would require a new extension point in Zarr, which in turn requires a lengthy ZEP process. 
  - By putting `zarr_conventions_metadata` in attributes, we affirm that it is outside of the purview of the Zarr spec itself; these conventions are purely "user" metadata.
  - Once the `zarr_conventions_metadata` mechanism has matured, we may attempt to promote it to a top-level metadata key in the future.

- How should we handle pre-existing "implicit conventions" like [ome-zarr](https://ome-zarr.readthedocs.io/en/stable/) or Xarray's `_FillValue` encoding?
  - Our goal here is to create a robust conventions mechanism that will serve the community for many years to come. That is incompatible with also supporting implicit conventions.
  - Instead, it is possible to document these conventions as "registered attributes" in the zarr-extensions repo.

- What is the mechanisms for discovery of conventions?
  - The `zarr-conventions` repository provides an [optional] central place for the community to maintain and discover conventions. The `zarr-conventions/template` repo makes it easy to author new conventions. 
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
  - In order to support opaque/provided 

- Why is name recommended?
  - We use it to populate the website showing all conventions; tools also use it for nice metadata representations.

- Why is name not required?

- Why do you use a UUID for the key?
    - It provides a decentralized version for guaranteed uniqueness


- *Happy to see a more decentralized possibility to extend Zarr.* However, why is the mehcanism limited to the configuration field and do not apply to the entire `attributes` object? This limits composability and the possibility for an extension to extend another as it is often done in STAC.

Category of use-cases

# Implementations

- GDAL: ZARR: add a prototype read implementation of the geo/proj attribute conventions: https://github.com/OSGeo/gdal/pull/13215

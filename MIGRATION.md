# Migration Guide: Adopting the Zarr Conventions Framework

This guide helps maintainers of existing Zarr conventions (like OME-Zarr, Xarray encoding, CF conventions, etc.) adopt the Zarr Conventions Framework while maintaining full backward compatibility.

## Table of Contents

- [Overview](#overview)
- [Backward Compatibility Guarantee](#backward-compatibility-guarantee)
- [Migration Phases](#migration-phases)
- [Step-by-Step Guide](#step-by-step-guide)
- [Convention-Specific Examples](#convention-specific-examples)
- [Tool Implementation Patterns](#tool-implementation-patterns)
- [FAQ](#faq)

---

## Overview

The Zarr Conventions Framework provides a standardized way to declare, validate, and discover conventions for Zarr metadata. If you maintain an existing convention, you can adopt the framework through purely **additive changes** that maintain full backward compatibility.

### Key Principles

✅ **Additive Only** - Only add `zarr_conventions` field; no changes to existing attributes  
✅ **Backward Compatible** - Existing data remains valid forever  
✅ **Tool Compatible** - Old tools continue working without modification  
✅ **Opt-in** - Adopt at your own pace with no breaking changes  
✅ **Non-Retroactive** - Framework only applies to new data

---

## Backward Compatibility Guarantee

The conventions framework is designed to ensure:

1. **Existing data never breaks** - Data written before the framework remains valid indefinitely
2. **Existing tools continue working** - Tools can ignore `zarr_conventions` and use structural detection
3. **No rewriting required** - Old data doesn't need to be migrated or updated
4. **Graceful degradation** - Tools work with both old and new data seamlessly

The framework achieves this because:

- `zarr_conventions` is a new field that old tools safely ignore
- All existing attributes remain unchanged in their structure and location
- The framework is "safely ignorable" by definition - it's pure metadata

---

## Migration Phases

We recommend a three-phase approach to adoption:

### Phase 1: Documentation (No Breaking Changes)

**Goal:** Formalize your convention without changing any data or tools

Any of the following steps may be skipped if the described resource (e.g., spec, schema) already exists.

**Steps:**

1. Create formal specification document
2. Create JSON schema for validation
3. Generate UUID for permanent identification
4. Register convention in zarr-conventions organization

**Output:** Convention is documented and discoverable, but no data or tools change yet

---

### Phase 2: Opt-in Adoption (Backward Compatible)

**Goal:** Enable new data to declare convention usage while maintaining compatibility

**Steps:**
1. Update data writers to include `zarr_conventions` field
2. Keep all existing attributes completely unchanged
3. Update tools to optionally validate against schema
4. Maintain structural detection for old data

**Output:** New data benefits from validation and discovery; old data continues working

---

### Phase 3: Enhanced Tooling (Long Term)

**Goal:** Build richer functionality using convention declarations

**Steps:**
1. Build validation tools that use `zarr_conventions`
2. Improve error messages based on schema validation
3. Enable automatic discovery and rendering in UIs
4. Add convention composition with other standards

**Output:** Better tooling, improved user experience, enhanced interoperability

---

## Step-by-Step Guide

### Step 1: Create Specification Document

If your convention does not already have a specification, create a formal specification document that describes your convention. Use the [template repository](https://github.com/zarr-conventions/template) as a starting point.
If your convention does have a specification, add any missing properties required by the Zarr Convention Metadata Object to your convention's specification.

**Your spec should include:**

- UUID (permanent identifier)
- Name (short, human-readable)
- Description (one sentence summary)
- Link to JSON schema
- Detailed field descriptions
- Examples
- Version history (if using a versioning scheme)

**Example structure:**

```markdown
# My Convention

- **UUID**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- **Name**: `my-convention`
- **Schema URL**: `https://example.org/my-convention/v1/schema.json`
- **Spec URL**: `https://example.org/my-convention/v1/README.md`

## Description

Brief description of what this convention provides...

## Properties

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| ...      | ...  | ...         | ...      |
```

---

### Step 2: Generate UUID

Generate a UUID v4 for your convention:

```python
import uuid
convention_uuid = uuid.uuid4()
print(convention_uuid)
```

**Important:** 
- This UUID is permanent - never change it
- It uniquely identifies your convention forever
- Even if URLs change, the UUID remains the same

---

### Step 3: Create JSON Schema

Create a JSON schema that describes your convention's metadata structure. See the [template repository](https://github.com/zarr-conventions/template/blob/main/schema.json) for an example.

---

### Step 4: Register Convention

Register your convention in the [zarr-conventions organization](https://github.com/zarr-conventions):

1. Fork the [conventions registry](https://github.com/zarr-conventions/.github)
2. Add your convention to the list in `profile/README.md`
3. Submit a pull request

---

### Step 5: Update Data Writers

Update your data writing tools to include the `zarr_conventions` field.

**Before (existing convention):**
```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "my_convention": {
      "version": "1.0",
      "field1": "value1"
    }
  }
}
```

**After (with conventions framework):**
```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "schema_url": "https://example.org/my-convention/v1/schema.json",
        "spec_url": "https://example.org/my-convention/v1/README.md",
        "name": "my-convention",
        "description": "My convention for doing X"
      }
    ],
    "my_convention": {
      "version": "1.0",
      "field1": "value1"
    }
  }
}
```

**Implementation example (Python):**

```python
def write_with_convention(group, data, **kwargs):
    # Write data as usual
    group.create_array("data", data=data)
    
    # Add convention metadata (NEW)
    attrs = group.attrs
    attrs["zarr_conventions"] = [
        {
            "uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "schema_url": "https://example.org/my-convention/v1/schema.json",
            "spec_url": "https://example.org/my-convention/v1/README.md",
            "name": "my-convention",
            "description": "My convention for doing X"
        }
    ]
    
    # Write existing convention metadata (UNCHANGED)
    attrs["my_convention"] = {
        "version": "1.0",
        "field1": kwargs.get("field1", "default")
    }
```

---

### Step 6: Update Data Readers

Update your reading tools to optionally validate using `zarr_conventions` while maintaining backward compatibility.

**Implementation pattern:**

```python
def read_with_convention(group):
    # Check for explicit convention declaration (NEW)
    if "zarr_conventions" in group.attrs:
        conventions = group.attrs["zarr_conventions"]
        my_convention = find_convention(conventions, uuid="my-convention-uuid")
        
        if my_convention:
            # Optional: Validate against schema
            if "schema_url" in my_convention:
                validate_metadata(
                    group.attrs.get("my_convention"),
                    my_convention["schema_url"]
                )
    
    # Fall back to structural detection (UNCHANGED - works for old and new data)
    if "my_convention" in group.attrs:
        metadata = group.attrs["my_convention"]
        return parse_convention_metadata(metadata)
    
    raise ValueError("Not a valid convention group")

def find_convention(conventions, uuid=None, name=None):
    """Find convention by UUID or name."""
    for conv in conventions:
        if uuid and conv.get("uuid") == uuid:
            return conv
        if name and conv.get("name") == name:
            return conv
    return None

def validate_metadata(metadata, schema_url):
    """Optionally validate metadata against JSON schema."""
    import jsonschema
    import requests
    
    schema = requests.get(schema_url).json()
    try:
        jsonschema.validate(metadata, schema)
    except jsonschema.ValidationError as e:
        print(f"Validation warning: {e.message}")
```

---

## Convention-Specific Examples

### Example 1: OME-Zarr 0.5

OME-Zarr uses nested metadata under the `ome` key.

**Before conventions framework:**
```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "ome": {
      "version": "0.5",
      "multiscales": [...]
    }
  }
}
```

**After conventions framework:**
```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "schema_url": "https://ngff.openmicroscopy.org/0.5/schema.json",
        "spec_url": "https://ngff.openmicroscopy.org/0.5/",
        "name": "ome",
        "description": "OME-NGFF bioimaging data model version 0.5"
      }
    ],
    "ome": {
      "version": "0.5",
      "multiscales": [...]
    }
  }
}
```

**Migration steps for OME:**
1. Generate UUID for OME-NGFF convention
2. Create/formalize JSON schema from existing spec
3. Update OME-Zarr writers (bioformats2raw, ome-zarr-py, etc.) to add `zarr_conventions`
4. Update readers to optionally validate
5. Document in OME-NGFF specification

---

### Example 2: Xarray Encoding (`_FillValue`)

Xarray uses flat attributes at the root level.

**Before conventions framework:**
```json
{
  "zarr_format": 3,
  "node_type": "array",
  "attributes": {
    "_FillValue": -9999,
  }
}
```

**After conventions framework:**
```json
{
  "zarr_format": 3,
  "node_type": "array",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "schema_url": "https://conventions.zarr.dev/xarray-encoding/v1/schema.json",
        "spec_url": "https://conventions.zarr.dev/xarray-encoding/v1/README.md",
        "name": "xarray-encoding",
        "description": "Xarray encoding conventions for Zarr"
      }
    ],
    "_FillValue": -9999,
  }
}
```

**Migration steps for Xarray:**
1. Document existing encoding conventions formally
2. Create JSON schema for encoding attributes
3. Generate UUID
4. Update `xarray.to_zarr()` to add `zarr_conventions` when writing
5. Maintain current reading logic (already uses structural detection)

---

### Example 3: CF Conventions

CF conventions could be represented similarly.

**After conventions framework:**
```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      {
        "uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "schema_url": "https://cfconventions.org/cf-conventions/zarr/v1/schema.json",
        "spec_url": "https://cfconventions.org/cf-conventions/zarr/v1/",
        "name": "cf",
        "description": "CF conventions for climate and forecast data"
      }
    ],
    "Conventions": "CF-1.11",
    "title": "Example CF dataset",
    "institution": "Example University"
  }
}
```

---

## Tool Implementation Patterns

### Pattern 1: Dual Detection (Recommended)

Support both convention declaration and structural detection:

```python
def read_convention_data(group):
    # Try convention-aware reading first
    if "zarr_conventions" in group.attrs:
        for conv in group.attrs["zarr_conventions"]:
            if conv.get("uuid") == MY_CONVENTION_UUID:
                return read_with_validation(group, conv)
    
    # Fall back to structural detection
    if "my_convention" in group.attrs:
        return read_legacy_format(group)
    
    raise ValueError("Unknown format")
```

### Pattern 2: Validation Helpers

Provide validation utilities for users:

```python
def validate_convention(group, convention_uuid=None):
    """Validate group metadata against convention schema."""
    if "zarr_conventions" not in group.attrs:
        raise ValueError("No conventions declared")
    
    for conv in group.attrs["zarr_conventions"]:
        if convention_uuid is None or conv.get("uuid") == convention_uuid:
            schema_url = conv.get("schema_url")
            if schema_url:
                schema = fetch_schema(schema_url)
                validate_against_schema(group.attrs, schema)
```

### Pattern 3: Convention Discovery

Enable automatic discovery of conventions:

```python
def discover_conventions(group):
    """Discover which conventions are used."""
    if "zarr_conventions" not in group.attrs:
        return []
    
    conventions = []
    for conv in group.attrs["zarr_conventions"]:
        conventions.append({
            "uuid": conv.get("uuid"),
            "name": conv.get("name"),
            "schema_url": conv.get("schema_url"),
            "spec_url": conv.get("spec_url")
        })
    
    return conventions
```

### Pattern 4: Graceful Warnings

Warn but don't fail when conventions are unrecognized:

```python
def read_with_warnings(group):
    """Read data with warnings for unknown conventions."""
    if "zarr_conventions" in group.attrs:
        for conv in group.attrs["zarr_conventions"]:
            if not is_convention_supported(conv):
                warnings.warn(
                    f"Unknown convention: {conv.get('name')} "
                    f"(UUID: {conv.get('uuid')}). "
                    f"See {conv.get('spec_url')} for details."
                )
    
    # Continue reading with available conventions
    return read_data(group)
```

---

## Resources

- [Zarr Conventions Specification](https://github.com/zarr-conventions/zarr-conventions-spec)
- [Template Repository](https://github.com/zarr-conventions/template)
- [Conventions Registry](https://github.com/zarr-conventions)
- [FAQ Document](./FAQ.md)
- [Example Conventions](https://github.com/zarr-conventions#conventions)

---

## Getting Help

- **Questions?** Open a [discussion](https://github.com/orgs/zarr-conventions/discussions)
- **Issues?** File an [issue](https://github.com/zarr-conventions/zarr-conventions-spec/issues)
- **Chat?** Join us on [Zarr Zulip](https://ossci.zulipchat.com/)

---

## Summary Checklist

Use this checklist to track your migration progress:

**Phase 1: Documentation**
- [ ] Create specification document
- [ ] Generate UUID
- [ ] Create JSON schema
- [ ] Register in zarr-conventions
- [ ] Add examples to documentation

**Phase 2: Implementation**
- [ ] Update data writers to include `zarr_conventions`
- [ ] Test backward compatibility with old readers
- [ ] Update readers to optionally validate
- [ ] Maintain structural detection fallback
- [ ] Document changes in release notes

**Phase 3: Enhancement**
- [ ] Add validation utilities
- [ ] Improve error messages using schema
- [ ] Enable convention discovery in tools
- [ ] Document composition with other conventions
- [ ] Gather community feedback

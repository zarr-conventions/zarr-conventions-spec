# FAQ

## Basics

- Why isn't `zarr_conventions` a new key in core Zarr metadata (i.e., outside of `attributes`)?
  - Creating a new top-level key would require a new extension point in Zarr, which in turn requires a lengthy ZEP process. 
  - By putting `zarr_conventions` in attributes, we affirm that it is outside of the purview of the Zarr spec itself; these conventions are purely "user" metadata.
  - Once the `zarr_conventions` mechanism has matured, we may attempt to promote it to a top-level metadata key in the future.

- What's the difference between a Zarr convention and a Zarr extension?
  - Conventions use only the `attributes` field and can be safely ignored by Zarr implementations. The readers of conventions are domain-specific tools that add semantics on top of the core Zarr model. The underlying Zarr data model remains fully readable and usable even without understanding the convention, though domain-specific context may be missing.
  - Extensions modify core Zarr behavior and require implementation support. They connect to defined extension points in the spec (data types, chunk grids, codecs, chunk key encoding, storage transformers) and may change how data is encoded or stored.
  - A good rule of thumb: if the data would be completely meaningless or unintelligible without supporting it, it should be an extension rather than a convention.

- Do Zarr implementations need to understand `zarr_conventions`?
  - No; by definition, conventions cannot modify core Zarr behavior (this requires extensions). They sit purely "on top" of the Zarr data model.
  - That said, implementations may choose to interpret the conventions and implement related features. 
  - Or this can be left to higher-level libraries sitting on top of the Zarr implementation (e.g. Xarray.)

- Can conventions be used for arrays or groups?
  - They can be used for either or both.

## Value and Design Rationale

- What's the value of having a `zarr_conventions` field? Couldn't we just document conventions informally?
  - The `zarr_conventions` field provides significant practical value for both data producers and consumers:
  - **For data consumers**: It enables automated validation and tooling. As one implementor noted: "it lets a Zarr array or group say 'In addition to being valid Zarr, I also do X and Y'." Software can read `zarr_conventions`, fetch the JSON schemas, and validate the data - the only network trip required is fetching the extension schema. This makes it easy to say "we can't read your data because it's not valid" and point to specific validation failures.
  - **For data producers**: It provides a clear, machine-readable way to declare conformance to conventions, making it easier for consumers to understand and use the data correctly.
  - **For the ecosystem**: It prevents gatekeeping while maintaining quality through community adoption. Organizations can publish conventions without approval from a central authority. The community decides (via adoption and use) if a given convention is valuable, rather than requiring approval from a standards body.
  - **For tooling developers**: It enables generic tools that weren't built for specific conventions to still validate and work with them, as long as those conventions provide schemas.
  - Informal documentation alone doesn't provide these machine-readable, validation-friendly capabilities. The `zarr_conventions` field makes conventions both discoverable and verifiable.

- Has this approach been proven to work at scale?
  - Yes. This design is directly inspired by STAC's `stac_extensions` field, which has been battle-tested at scale by NASA, USGS, ESA, Copernicus, and hundreds of other organizations.
  - The STAC extensions ecosystem includes 80+ extensions and has been in production use for years, precisely because it's simple and decentralized.
  - The model has proven successful in handling diverse use cases across the geospatial community without requiring central coordination or approval.
  - STAC converged on schema-identified extensions after real-world testing (see [STAC issue #278](https://github.com/radiantearth/stac-spec/issues/278) and [#1063](https://github.com/radiantearth/stac-spec/issues/1063)).
  - By adopting a proven model, the Zarr conventions framework benefits from significant network effects: much of the geospatial community already understands these patterns, which should accelerate adoption for Zarr.

- Why not require central approval for conventions like the earlier "registered attributes" approach?
  - Central approval creates bottlenecks and gatekeeping that slow innovation. The Zarr Steering Council (or any central body) would become a approval bottleneck, may lack domain expertise to evaluate specialized conventions, and couldn't scale to support rapid community innovation.
  - The STAC experience demonstrates that "the community decides (via adoption) if a given extension is good or not" works better than requiring formal approval.
  - Government agencies, research institutions, and companies can publish conventions relevant to their domains without waiting for approval from "the Zarr people."
  - As an unfunded, oversight-light community, the Zarr governance structure isn't well-positioned to be a gatekeeper for all possible domain-specific conventions.
  - Quality control happens through community adoption, documentation, and shared best practices rather than central authority.
  - This approach enables innovation while the conventions framework itself can evolve based on what patterns prove successful in practice.

- Why does the specification focus on schema-identified conventions?
  - Schema identification (via `schema_url`) enables automated validation, which is crucial for tooling and interoperability.
  - This approach was validated through real-world experience in the STAC community, which converged on schema-identified extensions after testing alternatives.
  - Schemas provide a machine-readable contract between data producers and consumers, making it possible to build generic tools that work across conventions they weren't specifically designed for.
  - The `spec_url` provides human-readable documentation, while the `schema_url` enables machine validation - together they serve both humans and tools.
  - This dual approach has proven successful at scale in the geospatial ecosystem and provides a foundation for a thriving conventions ecosystem.
  
## Convention Identification and Metadata

- Why is one of the `schema_url` and `spec_url` fields required?
  - At least one URL is required to serve as the unique identifier for the convention. The schema_url provides validation capabilities, while the spec_url provides human-readable documentation. Having at least one ensures conventions are both identifiable and discoverable.

- Should I provide _both_ `schema_url` and `spec_url`?
  - It's recommended to provide both when possible. The schema_url enables validation and tooling support, while the spec_url provides human-readable documentation. If you can only provide one, choose based on your use case: schema_url for machine validation, spec_url for human consumption.

- Why use schema_url or spec_url instead of UUIDs for identification?
  - URLs provide a mechanism for both identification and discovery, simplifying the specification. Schema URLs enable validation, while spec URLs point to documentation. Both naturally encode version information and provide a discoverable path to more information. While this requires conventions to maintain stable URLs, this is consistent with common practices in web standards.

- Why is the `name` field recommended?
  - We will use the name to populate the website showing all conventions; tools also will use it for nice metadata representations.

- Why is the `name` field not required?
  - Name should only be used for human-oriented displays, it isn't required because it isn't strictly necessary for a convention to be useful.

## Multiple Conventions and Composability

- Can I use multiple conventions on the same array or group?
  - Yes! Multiple conventions can coexist and even compose. The `zarr_conventions` field is an array specifically to support this.
  - For example, you might have both a `multiscales` convention and a `proj` (geospatial projection) convention on the same group.
  - When conventions are designed to be composable, they should use namespace prefixes (e.g., `proj:`, `ome:`) to avoid property name conflicts.
  - If you're designing a convention that others might want to extend, set `additionalProperties: true` in your JSON schema for the objects you want to be extensible.

- How does this specification enable composability?
  - Convention properties exist at the root `attributes` level (not isolated in the `zarr_conventions` array), allowing conventions to reference and extend properties from other conventions. This mirrors the proven pattern used by STAC extensions. For example, a projection convention can add projection-specific properties to items within a multiscales layout.

- How are property name collisions prevented?
  - Conventions use namespace prefixes (e.g., `proj:`, `ome:`) to prevent collisions. This approach has been successfully used by the STAC community for years without collision issues. Convention authors coordinate on naming within their domain.

- If I declare a convention on a group, does it apply to all arrays and subgroups within it?
  - No, there is no inheritance. Each array and group must declare its own conventions in its own `zarr_conventions` attribute.
  - This design is intentional: it makes the metadata self-contained and explicit at each level, avoids ambiguity about which conventions apply where, and follows the pattern established by STAC.
  - If your convention has requirements that span a hierarchy (e.g., a group structure with specific expected child arrays), document those requirements in your convention specification. Tools reading the data should validate the entire structure as needed.

## Versioning and Changes

- How do I version my convention?
  - Include version information in your convention's metadata or the schema or spec URL (e.g., using git tags like `/v0.1.0/` or versioned paths). When you make breaking changes, update to a new major version in the URL.

- What happens if a convention needs to make breaking changes?
  - Convention versioning should be included in the identifier URL. When making breaking changes, create a new URL with a new major version (e.g., change `/v1.0.0/` to `/v2.0.0/`).
  - Document the breaking changes in your convention's changelog. The old version remains valid - existing data doesn't break.
  - Tools can support multiple versions simultaneously. Consider providing migration guidance in your documentation.
  - Non-breaking changes (adding optional fields, fixing typos in documentation) can be released as minor or patch versions at the same URL or with updated version numbers, depending on your convention's versioning policy.

- Can a convention be deprecated or retired?
  - Yes, but carefully: document the deprecation clearly in your convention's documentation, provide migration guidance to a replacement convention (if any), and consider keeping the URLs available even for deprecated conventions so existing data remains interpretable.
  - Use appropriate HTTP status codes if hosting your own schemas (e.g., 410 Gone with information about the replacement).
  - The convention remains valid for existing data even if deprecated. Tools should handle deprecated conventions gracefully, potentially with warnings.

- What if my schema_url or spec_url becomes unavailable?
  - Convention authors should use stable hosting for URLs (e.g., GitHub releases, permanent DOIs). Tools consuming conventions should implement caching strategies and graceful degradation when URLs are temporarily unavailable. The convention properties remain valid even if the identifier URL is unreachable. Convention authors may register a new convention conveying similar information if an identifying URL is irreconcilably lost, and should coordinate with downstream libraries about the change.

## Discovery and Governance

- What is the mechanism for discovery of conventions?
  - The `zarr-conventions` repository provides an [optional](https://github.com/zarr-conventions#share-your-convention) central place for the community to maintain and discover conventions. The [`zarr-conventions/template`](https://github.com/zarr-conventions/template) repo makes it easy to author new conventions. 
  - In the future, we may create a website similar to [STAC extensions](https://stac-extensions.github.io/).

- What if you don't want a convention to be discoverable?
  - Given that the identifying URL should be resolvable, this spec does not support opaque conventions.

- Why not use the ZEP (Zarr Enhancement Proposal) process for conventions?
  - The conventions framework is intentionally much more lightweight than the ZEP process. ZEPs require approval from the Zarr Steering Council or Implementation Council and involve formal voting, and are appropriate for changes to the core spec or new extension points.
  - Conventions operate entirely within the existing `attributes` field and require no spec changes. Anyone can create a convention without central authority approval.
  - Conventions are a "low stakes" way to prototype community-driven functionality that may inform future extensions. The conventions framework is explicitly designed to enable rapid, decentralized innovation while the ZEP process remains the path for more fundamental changes.
  - This replaces the earlier "registered attributes" mechanism which required steering council approval.

- How does this relate to ZEP-9 (Generic Extensions)?
  - The concept of "Generic Extensions" was proposed in [ZEP-9](https://zarr.dev/zeps/draft/ZEP0009.html), but progress towards acceptance was deliberately paused. Discussion led to the conclusion that many of the use cases for generic extensions could be implemented in a more lightweight way solely using `attributes`.
  - This conventions specification is a direct response to that suggestion. Rather than requiring changes to Zarr implementations, conventions allow the community to experiment with new functionality in a decentralized way.
  - The lessons learned from conventions may inform the more formal Extensions framework in the future.

- Why does this approach look similar to STAC extensions?
  - The Zarr conventions framework is directly inspired by STAC's (SpatioTemporal Asset Catalog) highly successful extensions model. STAC moved all extensions out of the core spec in 2021 to an independent organization, and extensions are identified by URLs to their schemas.
  - In STAC, multiple extensions can compose on the same item, no central approval is needed to create an extension, and a template repository makes it easy to create new extensions.
  - This model has proven successful in the geospatial community for years, with dozens of extensions co-existing without collision issues. The Zarr conventions framework adapts these proven patterns to the Zarr ecosystem.

## Working with Existing Data

- How should we handle pre-existing "implicit conventions" like [ome-zarr](https://ome-zarr.readthedocs.io/en/stable/) or Xarray's `_FillValue` encoding?
  - Our goal here is to create a robust conventions mechanism that will serve the community for many years to come. That is incompatible with also supporting implicit conventions.
  - Instead, these conventions can be documented and shared in the zarr-conventions organization. Data using these conventions remains valid; tools can continue to detect them structurally for backward compatibility.

- I have existing Zarr data with domain-specific metadata. Do I need to rewrite it to follow this specification?
  - No. This specification applies only to new data written after its adoption.
  - For existing data, it can continue to be read as-is, the conventions it follows can be documented and shared in the zarr-conventions organization, and tools reading the data can continue to use structural detection (checking for specific attributes) for backward compatibility.
  - When creating new versions or derivatives of the data, consider adopting explicit convention declarations.
  - The specification is designed for forward compatibility, not retroactive application.

## Creating and Using Conventions

- Should I create a new convention or use an existing one?
  - Use existing if there's already a convention in your domain that meets your needs, even if it's not perfect. Standardization benefits increase with adoption.
  - Extend existing if an existing convention is close but missing features. Design your convention to compose with it using namespace prefixes.
  - Create new if no existing convention covers your domain, existing conventions have fundamentally incompatible approaches, or you're addressing a genuinely novel use case.
  - Check the zarr-conventions organization and community discussions to see what already exists. Consider discussing your needs before creating something new.

- How can I validate that my data conforms to a convention?
  - If a convention provides a `schema_url`, fetch the JSON schema from that URL and validate the attributes against the schema using standard JSON schema validation tools.
  - Many languages have JSON schema validation libraries (Python: jsonschema, JavaScript: ajv, etc.).
  - Generic Zarr tools may eventually provide convention validation features, but this is optional. Domain-specific libraries often have built-in validation for their conventions.
  - The `schema_url` makes it possible for generic tooling to validate conventions it wasn't specifically built for.

- Should I use prefixes (like `proj:code`) or nesting (like `{"proj": {"code": "..."}}`) for my convention?
  - Both approaches work, and the choice depends on your use case.
  - Prefix style (`proj:code`): flatter structure and easier to access, works well for conventions that extend others, similar to STAC approach, but requires consistent prefix usage.
  - Nesting style (`{"proj": {"code": "..."}}`): clear grouping of related properties, can be easier to validate as a unit, used by OME-NGFF, but more nesting to navigate.
  - The specification supports both. Document your choice clearly and be consistent within your convention.

- Does the schema have to be JSON Schema? Can I use other schema languages?
  - The specification currently focuses on JSON Schema because it's widely supported with tooling in many languages, it's the standard for validating JSON documents, and it's consistent with STAC's approach.
  - However, some community members have suggested alternatives like LinkML (which can generate JSON Schema). Future versions of the specification may support additional schema languages.
  - For now, if you use an alternative, also provide a JSON Schema export in the `schema_url` for maximum compatibility.

- Are there tools to help me create and test conventions?
  - The ecosystem is growing. The template repository at [`zarr-conventions/template`](https://github.com/zarr-conventions/template) provides a starting point. Generic JSON Schema validators are also available in most programming languages.
  - Check the zarr-conventions organization for evolving tooling. As the conventions framework matures, expect more tooling for convention creation, validation, and discovery.

## Advanced Topics

- Can conventions help with interoperability between Zarr and other formats like NetCDF or HDF5?
  - Yes! This is a key use case. Since Zarr, NetCDF, and HDF5 implement similar data models, a convention can be designed to work across formats.
  - For example, CF Conventions work on both NetCDF and Zarr via Xarray, and a convention defining coordinate systems could apply to any format with an attributes mechanism.
  - This separation of concerns (storage format vs. domain semantics) enables better modularity.
  - When designing cross-format conventions, document how the convention maps to each format's specifics while keeping the core semantics consistent.

- Can I create a convention for internal use within my organization without making it public?
  - While you technically can, this specification encourages resolvable URLs for discovery and interoperability. However, you could use internal URLs accessible only within your organization, document a convention privately before making it public, or use a publicly resolvable URL even for niche conventions.
  - The main consideration: if others might ever need to read your data, making the convention discoverable helps them understand it. Even internal conventions benefit from documentation.


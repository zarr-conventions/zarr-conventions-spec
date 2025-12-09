# FAQ

## Status

- Is the Zarr Conventions framework ready for use?
  - Yes, the Zarr Conventions framework is ready for use. It was originally designed at the 2025 Zarr Summit and has underwent a round of revisions based on the experiences of early adopters and additional community members. We encourage folks to use the framework.
- How do I share my Zarr Convention?
  - Add your convention to the list at the bottom of this page: https://github.com/zarr-conventions/.github/edit/main/profile/README.md. In the future, we will build a nicer representation of existing conventions.
- How do I find existing Zarr Conventions?
  - Check out the list at https://github.com/zarr-conventions#share-your-convention!
- How do I get started?
  - The two current entry points are the [specification repository](https://github.com/zarr-conventions/zarr-conventions-spec/) and the [template repository](https://github.com/zarr-conventions/template). In early 2026, we will design a more comprehensive getting started guide.
- How do I get help?
  - Open a new discussion topic in https://github.com/orgs/zarr-conventions/discussions, or reach out via the [Zarr Zulip](https://ossci.zulipchat.com/).

## Value and Design Rationale

- What's the value of having a `zarr_conventions` field? Couldn't we just document Conventions informally?
  - The `zarr_conventions` field provides significant practical value for both data producers and consumers:
    - **For data consumers**: It enables automated validation and tooling. As one implementor noted: "it lets a Zarr array or group say 'In addition to being valid Zarr, I also do X and Y'." Software can read `zarr_conventions`, fetch the JSON schemas, and validate the data - the only network trip required is fetching the convention schema. This makes it easy to say "we can't understand your data because it's not valid" and point to specific validation failures.
    - **For data producers**: It provides a clear, machine-readable way to declare conformance to Conventions, making it easier for consumers to understand and use the data correctly.
    - **For the ecosystem**: It prevents gatekeeping while maintaining quality through community adoption. Organizations can publish Conventions without approval from a central authority. The community decides (via adoption and use) if a given Convention is valuable, rather than requiring approval from a standards body.
    - **For tooling developers**: It enables generic tools that weren't built for specific Conventions to still validate and work with them, as long as those Conventions provide schemas.
    - Informal documentation alone doesn't provide these machine-readable, validation-friendly capabilities. The `zarr_conventions` field makes Conventions both discoverable and verifiable.

- Has this approach been proven to work at scale?
  - Yes. This design is directly inspired by STAC's `stac_extensions` field, which has been battle-tested at scale by NASA, USGS, ESA, Copernicus, and hundreds of other organizations.
  - The STAC extensions ecosystem includes 80+ extensions and has been in production use for years, precisely because it's simple and decentralized.
  - The model has proven successful in handling diverse use cases across the geospatial community without requiring central coordination or approval.
  - STAC converged on schema-identified extensions after real-world testing (see [STAC issue #278](https://github.com/radiantearth/stac-spec/issues/278) and [#1063](https://github.com/radiantearth/stac-spec/issues/1063)).
  - By adopting a proven model, the Zarr Conventions framework benefits from significant network effects: much of the geospatial community already understands these patterns, which should accelerate adoption for Zarr.

- Why not require central approval for Conventions like the earlier "registered attributes" approach?
  - Central approval creates bottlenecks and gatekeeping that slow innovation. The Zarr Steering Council (or any central body) would become a approval bottleneck, may lack domain expertise to evaluate specialized Conventions, and couldn't scale to support rapid community innovation.
  - The STAC experience demonstrates that "the community decides (via adoption) if a given extension is good or not" works better than requiring formal approval.
  - Government agencies, research institutions, and companies can publish Conventions relevant to their domains without waiting for approval from "the Zarr people."
  - As an unfunded, oversight-light community, the Zarr governance structure isn't well-positioned to be a gatekeeper for all possible domain-specific Conventions.
  - Quality control happens through community adoption, documentation, and shared best practices rather than central authority.
  - This approach enables innovation while the Conventions framework itself can evolve based on what patterns prove successful in practice.

- Why does the specification focus on schema-identified Conventions?
  - Schema identification (via `schema_url`) enables automated validation, which is crucial for tooling and interoperability.
  - This approach was validated through real-world experience in the STAC community, which converged on schema-identified extensions after testing alternatives.
  - Schemas provide a machine-readable contract between data producers and consumers, making it possible to build generic tools that work across Conventions they weren't specifically designed for.
  - The `spec_url` provides human-readable documentation, while the `schema_url` enables machine validation - together they serve both humans and tools.
  - This dual approach has proven successful at scale in the geospatial ecosystem and provides a foundation for a thriving Conventions ecosystem.
  
- Why isn't `zarr_conventions` a new key in core Zarr metadata (i.e., outside of `attributes`)?
  - Creating a new top-level key would require a new Extension point in Zarr, which in turn requires a lengthy ZEP process. 
  - By putting `zarr_conventions` in attributes, we affirm that it is outside of the purview of the Zarr spec itself; these conventions are purely "user" metadata.
  - Once the `zarr_conventions` mechanism has matured, we may attempt to promote it to a top-level metadata key in the future.

- What's the difference between a Zarr _Convention_ and a Zarr _Extension_?
  - Conventions use only the `attributes` field and can be safely ignored by Zarr implementations. The readers of Conventions are domain-specific tools that add semantics on top of the core Zarr model. The underlying Zarr data model remains fully readable and usable even without understanding the Convention, though domain-specific context may be missing.
  - Extensions modify core Zarr behavior and require implementation support. They connect to defined Extension points in the spec (data types, chunk grids, codecs, chunk key encoding, storage transformers) and may change how data is encoded or stored.
  - A good rule of thumb: if the data would be completely meaningless or unintelligible without supporting it, it should be an Extension rather than a Convention.

- Do Zarr implementations need to understand `zarr_conventions`?
  - No; by definition, Conventions cannot modify core Zarr behavior (this requires Extensions). They sit purely "on top" of the Zarr data model.
  - That said, implementations may choose to interpret the Conventions and implement related features. 
  - Or this can be left to higher-level libraries sitting on top of the Zarr implementation (e.g. Xarray).

- Can Conventions be used for arrays or groups?
  - They can be used for either or both.

## Convention Identification and Metadata

- Why is one of the `uuid`, `schema_url` or `spec_url` fields required?
  - A UUID or at least one URL is required to serve as the unique identifier for the Convention. The UUID provides a consistent, unique identifier isolated from potential domain name issues, the schema_url provides validation capabilities, while the spec_url provides human-readable documentation. Having at least one ensures Conventions are both identifiable and discoverable.

- Should I provide _all_of_ `uuid`, `schema_url` and `spec_url`?
  - It's recommended to provide all three when possible. The `uuid` provides consistency if domains for the spec and schema change, the `schema_url` enables validation and tooling support, while the `spec_url` provides human-readable documentation. If you can only provide one, choose based on your use case: `uuid` for guaranteed consistency, `schema_url` for machine validation, `spec_url` for human consumption.

- When should I use `uuid`, `schema_url`, or `spec_url` for identification?
  - It is RECOMMENDED to use both a UUID and schema_url/spec_url together for maximum robustness:
    - `uuid` provides a permanent, immutable identifier that never changes, even if hosting locations change
    - `schema_url` enables validation and provides a way to fetch the schema for automated tooling
    - `spec_url` provides human-readable documentation
  - Using all three together combines the stability of UUIDs with the practical benefits of URL-based discovery and validation.
  - If you only provide URLs (no UUID), your Convention is still valid but may be harder to track if the hosting location changes.

- Does UUID or schema_url take precedence as the primary identifier?
  - UUID takes precedence over all other identifiers when present. The identifier priority order is:
    1. `uuid` (if present) - permanent, immutable identifier
    2. `schema_url` (if present and no uuid) - enables validation
    3. `spec_url` (if no uuid or schema_url) - provides documentation
  - UUIDs are immutable identifiers of a Convention and remain constant even if schema_url, spec_url, or name changes.

- If `UUID` is used as the identifier, are `schema_url` and `spec_url` recommended?
  - Yes, it is RECOMMENDED to provide both `schema_url` and/or `spec_url` alongside the UUID:
    - `schema_url` enables automated validation and tooling support
    - `spec_url` provides human-readable documentation for developers
    - The UUID provides stability while the URLs provide practical functionality
  - This combination gives you the best of both worlds: permanent identification via UUID and practical utility via URLs.

- How do I register my UUID?
  - In the future, you may register your UUID with Convention registries by making pull requests to registries of interest (e.g., organization-specific registries or community registries).
  - However, registration is not required - UUIDs are designed to be unique without central coordination.
  - The main purpose of registries is information dissemination and convenience, not central authority.

- Who assigns a UUID to my Convention?
  - You, as the Convention author, assign the UUID yourself. No central authority is required to assign a UUID.
  - To generate a UUID and assign it to your Convention, execute the following in Python:
    ```python
    import uuid
    uuid_for_my_convention = uuid.uuid4()
    print(uuid_for_my_convention)
    ```
  - Once assigned, a UUID should never change. The UUID permanently identifies your Convention regardless of name changes, URL changes, or hosting location changes.

- What type of UUID should be used?
  - UUID v4 (a random 128-bit number) is RECOMMENDED if there are no other considerations.
  - Any type of UUID is acceptable, but UUID v4 is the simplest and most commonly used.
  - UUIDs must conform to [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html).

- Why is the `name` field recommended?
  - We will use the name to populate the website showing all Conventions; tools also will use it for nice metadata representations.

- Why is the `name` field not required?
  - Name should only be used for human-oriented displays, it isn't required because it isn't strictly necessary for a Convention to be useful.

## Multiple Conventions and Composability

- Can I use multiple Conventions on the same array or group?
  - Yes! Multiple Conventions can coexist and even compose. The `zarr_conventions` field is an array specifically to support this.
  - For example, you might have both a `multiscales` Convention and a `proj` (geospatial projection) Convention on the same group.
  - When Conventions are designed to be composable, they should use namespace prefixes (e.g., `proj:`, `ome:`) to avoid property name conflicts.
  - If you're designing a Convention that others might want to extend, set `additionalProperties: true` in your JSON schema for the objects you want to be extensible.

- How does this specification enable composability?
  - Convention properties exist at the root `attributes` level (not isolated in the `zarr_Conventions` array), allowing Conventions to reference and extend properties from other Conventions. This mirrors the proven pattern used by STAC extensions. For example, a projection Convention can add projection-specific properties to items within a multiscales layout.

- How are property name collisions prevented?
  - Conventions use namespace prefixes (e.g., `proj:`, `ome:`) to prevent collisions. This approach has been successfully used by the STAC community for years without collision issues. Convention authors coordinate on naming within their domain.

- If I declare a Convention on a group, does it apply to all arrays and subgroups within it?
  - No, there is no inheritance. Each array and group must declare its own Conventions in its own `zarr_conventions` attribute.
  - This design is intentional: it makes the metadata self-contained and explicit at each level, avoids ambiguity about which conventions apply where, and follows the pattern established by STAC.
  - If your Convention has requirements that span a hierarchy (e.g., a group structure with specific expected child arrays), document those requirements in your Convention specification. Tools reading the data should validate the entire structure as needed.

## Versioning and Changes

- How do I version my Convention?
  - The Zarr Conventions framework does not require versioning or prescribe semantics for versioning schemes. However, if you choose to version your Convention, it is RECOMMENDED to:
    - Include version information in the identifier URL (e.g., using git tags like `/v1/` or versioned paths like `/v1.0.0/`)
    - Define a version field within your Convention's attributes (e.g., `proj:version`) so libraries don't need to parse the URL
    - Clearly document the semantic meaning of your version numbers in your Convention specification
  - If using semantic versioning or similar schemes, consider whether you need MAJOR.MINOR.PATCH granularity or if MAJOR versions alone (v1, v2, etc.) would suffice for your use case.
  - If using semantic versioning or similar schemes, consider whether you need the version in the Convention's attributes (e.g., `proj:version`) so libraries don't need to parse the URL or whether you need a single source of truth for the version via the URL

- What happens if a Convention needs to make breaking changes?
  - If your Convention uses versioning, breaking changes SHOULD result in a new identifier URL with an updated major version (e.g., change `/v1/` to `/v2/`).
  - Document the breaking changes in your Convention's changelog and provide migration guidance.
  - The old version remains valid - existing data doesn't break. Tools can support multiple versions simultaneously.
  - If you don't use versioning, you can use a new identifier URL (effectively a new Convention UUID) for incompatible changes, or extend your schema to handle both old and new structures where possible.

- Do I need to use version numbers in my Convention?
  - No. Version numbers are optional. JSON schemas can often be extended in backwards-compatible ways without explicit versioning:
    - Adding optional fields doesn't break old data
    - Old implementations can fail gracefully on unknown fields
    - New implementations can support both old and new structures
  - However, versioning can be useful when:
    - You need to track non-structural changes (e.g., how a field should be interpreted)
    - You want to explicitly communicate compatibility expectations
    - Your Convention follows an established versioning practice in your domain
  - If you do use version numbers, define clear rules for what different versions mean (breaking vs. non-breaking changes).

- What if the version in the URL and a version field in my Convention metadata disagree?
  - If you include version information in both places, you MUST ensure they stay synchronized. Define in your Convention specification which takes precedence if they disagree (typically the URL).
  - Consider whether you need version information in both places - having it only in the URL may be simpler and less error-prone.

- Can a Convention be deprecated or retired?
  - Yes, but carefully: document the deprecation clearly in your Convention's documentation, provide migration guidance to a replacement Convention (if any), and consider keeping the URLs available even for deprecated Conventions so existing data remains interpretable.
  - Use appropriate HTTP status codes if hosting your own schemas (e.g., 410 Gone with information about the replacement).
  - The Convention remains valid for existing data even if deprecated. Tools should handle deprecated Conventions gracefully, potentially with warnings.

- What if my `schema_url` or `spec_url` becomes unavailable?
  - Convention authors should use stable hosting for URLs (e.g., GitHub releases, permanent DOIs). Tools consuming Conventions should implement caching strategies and graceful degradation when URLs are temporarily unavailable. The Convention properties remain valid even if the identifier URL is unreachable. Convention authors may register a new Convention conveying similar information if an identifying URL is irreconcilably lost, and should coordinate with downstream libraries about the change.

## Discovery and Governance

- What is the mechanism for discovery of Conventions?
  - The `zarr-conventions` repository provides an [optional](https://github.com/zarr-conventions#share-your-convention) central place for the community to maintain and discover Conventions. The [`zarr-conventions/template`](https://github.com/zarr-conventions/template) repo makes it easy to author new Conventions. 
  - In the future, we may create a website similar to [STAC extensions](https://stac-extensions.github.io/).

- What if you don't want a Convention to be discoverable?
  - You may use a UUID without a `schema_url` or `spec_url`, and/or use an internally resolvable `schema_url` and `spec_url`.

- Why not use the ZEP (Zarr Enhancement Proposal) process for Conventions?
  - The Conventions framework is intentionally much more lightweight than the ZEP process. ZEPs require approval from the Zarr Steering Council or Implementation Council and involve formal voting, and are appropriate for changes to the core spec or new extension points.
  - Conventions operate entirely within the existing `attributes` field and require no spec changes. Anyone can create a Convention without central authority approval.
  - Conventions are a "low stakes" way to prototype community-driven functionality that may inform future extensions. The Conventions framework is explicitly designed to enable rapid, decentralized innovation while the ZEP process remains the path for more fundamental changes.
  - This replaces the earlier "registered attributes" mechanism which required steering council approval.
  - Nevertheless, this framework will likely be used to update [ZEP-4](https://zarr.dev/zeps/draft/ZEP0004.html).

- How does this relate to ZEP-9 (Generic Extensions)?
  - The concept of "Generic Extensions" was proposed in [ZEP-9](https://zarr.dev/zeps/draft/ZEP0009.html), but progress towards acceptance was deliberately paused. Discussion led to the conclusion that many of the use cases for generic extensions could be implemented in a more lightweight way solely using `attributes`.
  - This Conventions specification is a direct response to that suggestion. Rather than requiring changes to Zarr implementations, Conventions allow the community to experiment with new functionality in a decentralized way.
  - The lessons learned from Conventions may inform the more formal Extensions framework in the future.

- Why does this approach look similar to STAC extensions?
  - The Zarr Conventions framework is directly inspired by STAC's (SpatioTemporal Asset Catalog) highly successful extensions model. STAC moved all extensions out of the core spec in 2021 to an independent organization, and extensions are identified by URLs to their schemas.
  - In STAC, multiple extensions can compose on the same item, no central approval is needed to create an extension, and a template repository makes it easy to create new extensions.
  - This model has proven successful in the geospatial community for years, with dozens of extensions co-existing without collision issues. The Zarr Conventions framework adapts these proven patterns to the Zarr ecosystem.

## Working with Existing Data

- How should we handle pre-existing "implicit Conventions" like [ome-zarr](https://ome-zarr.readthedocs.io/en/stable/) or Xarray's `_FillValue` encoding?
  - Our goal here is to create a robust Conventions mechanism that will serve the community for many years to come. That is incompatible with also supporting implicit Conventions.
  - Instead, these Conventions can be documented and shared in the zarr-conventions organization. Data using these Conventions remains valid; tools can continue to detect them structurally for backward compatibility.

- I have existing Zarr data with domain-specific metadata. Do I need to rewrite it to follow this specification?
  - No. This specification applies only to new data written after its adoption.
  - For existing data, it can continue to be read as-is, the Conventions it follows can be documented and shared in the zarr-conventions organization, and tools reading the data can continue to use structural detection (checking for specific attributes) for backward compatibility.
  - When creating new versions or derivatives of the data, consider adopting explicit Convention declarations.
  - The specification is designed for forward compatibility, not retroactive application.

## Creating and Using Conventions

- Should I create a new Convention or use an existing one?
  - Use existing if there's already a Convention in your domain that meets your needs, even if it's not perfect. Standardization benefits increase with adoption.
  - Extend existing if an existing Convention is close but missing features. Design your Convention to compose with it using namespace prefixes.
  - Create new if no existing Convention covers your domain, existing Conventions have fundamentally incompatible approaches, or you're addressing a genuinely novel use case.
  - Check the zarr-conventions organization and community discussions to see what already exists. Consider discussing your needs before creating something new.

- How can I validate that my data conforms to a Convention?
  - If a Convention provides a `schema_url`, fetch the JSON schema from that URL and validate the attributes against the schema using standard JSON schema validation tools.
  - Many languages have JSON schema validation libraries (Python: jsonschema, JavaScript: ajv, etc.).
  - Generic Zarr tools may eventually provide Convention validation features, but this is optional. Domain-specific libraries often have built-in validation for their Conventions.
  - The `schema_url` makes it possible for generic tooling to validate Conventions it wasn't specifically built for.

- When should schema validation occur?
  - Schema validation can occur at different points in the data lifecycle, based on lessons from STAC:
    - **Data providers** almost always validate their metadata against all Convention schemas before publishing. This ensures data quality at the source.
    - **Ingestion pipelines** usually validate before adding data to databases or catalogs. This provides a consistency check and catches issues early.
    - **Consumers** often use validation for debugging when data doesn't work as expected. If a tool breaks, validating against the Convention schemas is a good first debugging step to identify specification violations.
    - **UIs and tools** may use schema URLs to add functionality dynamically (e.g., a UI might detect a `proj` Convention and add a "Projection" section).
  - Validation is typically an explicit operation (not automatic) since it may require network calls to fetch schemas.
  - Readers are NOT expected to validate on every read - the `schema_url` is provided for optional validation when needed.
  - Writers should ensure their output conforms to the schema, but are not required to perform validation using the `schema_url` itself (though it can be useful for testing).

- Should I use prefixes (like `proj:code`) or nesting (like `{"proj": {"code": "..."}}`) for my Convention?
  - Both approaches work, and the choice depends on your use case.
  - Prefix style (`proj:code`): flatter structure and easier to access, works well for Conventions that extend others, similar to STAC approach, but requires consistent prefix usage.
  - Nesting style (`{"proj": {"code": "..."}}`): clear grouping of related properties, can be easier to validate as a unit, used by OME-NGFF, but more nesting to navigate.
  - The specification supports both. Document your choice clearly and be consistent within your Convention.

- Does the schema have to be JSON Schema? Can I use other schema languages?
  - The specification currently focuses on JSON Schema because it's widely supported with tooling in many languages, it's the standard for validating JSON documents, and it's consistent with STAC's approach.
  - However, some community members have suggested alternatives like LinkML (which can generate JSON Schema). Future versions of the specification may support additional schema languages.
  - For now, if you use an alternative, also provide a JSON Schema export in the `schema_url` for maximum compatibility.

- Are there tools to help me create and test Conventions?
  - The ecosystem is growing. The template repository at [`zarr-conventions/template`](https://github.com/zarr-conventions/template) provides a starting point. Generic JSON Schema validators are also available in most programming languages.
  - Check the zarr-conventions organization for evolving tooling. As the Conventions framework matures, expect more tooling for Convention creation, validation, and discovery.

## Advanced Topics

- Can Conventions help with interoperability between Zarr and other formats like NetCDF or HDF5?
  - Yes! This is a key use case. Since Zarr, NetCDF, and HDF5 implement similar data models, a Convention can be designed to work across formats.
  - For example, CF Conventions work on both NetCDF and Zarr via Xarray, and a Convention defining coordinate systems could apply to any format with an attributes mechanism.
  - This separation of concerns (storage format vs. domain semantics) enables better modularity.
  - When designing cross-format Conventions, document how the Convention maps to each format's specifics while keeping the core semantics consistent.

- Can I create a Convention for internal use within my organization without making it public?
  - While you technically can, this specification encourages resolvable URLs for discovery and interoperability. However, you could use internal URLs accessible only within your organization, document a Convention privately before making it public, or use a publicly resolvable URL even for niche Conventions.
  - The main consideration: if others might ever need to read your data, making the Convention discoverable helps them understand it. Even internal Conventions benefit from documentation.

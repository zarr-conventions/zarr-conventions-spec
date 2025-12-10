# Zarr Conventions Framework Roadmap

This document outlines the development and release roadmap for the Zarr conventions framework, a lightweight, decentralized approach to defining and sharing domain-specific metadata conventions for Zarr arrays and groups.

## Vision

Enable the Zarr community to rapidly innovate and share domain-specific conventions without central gatekeeping, inspired by the proven success of STAC extensions. Create a thriving ecosystem where conventions can compose and interoperate, supported by robust tooling and clear documentation.

## Current Status (November 2025)

### Completed
- ✅ Core specification drafted (README.md)
- ✅ JSON Schema for validating convention metadata (zarr-conventions-schema)
- ✅ Comprehensive FAQ document addressing common questions
- ✅ Decision to adopt STAC-like model over encapsulated approach
- ✅ `zarr-conventions` GitHub organization established
- ✅ Template repository created for convention authors
- ✅ Decision to retire `zarr-experimental` organization in favor of `zarr-conventions`
- ✅ Coordinate with new convention developers (e.g., dggs, generic multiscales)

### In Progress
- 🔄 Blog post draft describing the framework and design decisions
- 🔄 Migration plan from registered attributes approach
- 🔄 Coordination with existing convention authors (e.g., OME, CF)

---

## Phase 1: Finalization and Documentation (Q4 2025)

**Goal:** Stabilize the specification and prepare comprehensive documentation for early adopters.

### Specification Refinement

- [x] Review and incorporate community feedback from draft spec

### Testing with new conventions

- [x] Develop new conventions for geo:proj, multiscales, and dggs following this framework
- [x] Iterate on convention spec based on early adopters

### Community Engagement with existing conventions

- [x] Publish specification via GitHub
- [ ] Share specification on Zarr Zulip
- [ ] Gather feedback from key stakeholders (e.g., OME, GeoZarr, Xarray, GDAL)
- [ ] Address feedback and iterate on specification

### Documentation

- [ ] Complete and publish blog post explaining the framework
- [ ] Create migration guide for existing conventions (e.g., OME-NGFF, Xarray, CF)
- [ ] Create best-practices document with:
  - Quickstart guide for convention authors
  - Best practices for schema design
  - Composability patterns and examples
  - Testing and validation guidance

### JSON Schema

- [ ] Validate schema against all examples in specification
- [ ] Add schema versioning strategy
- [ ] Publish schema at stable URL (e.g., zarr-conventions.github.io)
- [ ] Create schema changelog

**Milestone:** Version 1 of specification ready for early adopters

---

## Phase 2: Tooling and Validation (Q1 2026)

**Goal:** Provide robust tooling to make convention creation, validation, and consumption easy.

### Publicizing framework

- [ ] Update ZEP 4 to formalize the new conventions system
- [ ] Remove prior registered attributes concept

### Python Validator

- [ ] Development of Python validator library
  - Read `zarr_conventions` from attributes
  - Fetch schemas from `schema_url`
  - Validate attributes against schemas
  - Handle multiple conventions on same node
  - Support schema caching
  - Provide clear validation error messages
- [ ] Add command-line interface for validation
- [ ] Publish to PyPI
- [ ] Documentation and usage examples
- [ ] Integration tests with example conventions

### Convention Template Improvements

- [ ] Enhance template repository with:
  - GitHub Actions for schema validation
  - Example test suite
  - Documentation template
  - Versioning best practices
  - Release workflow
- [ ] Create template variants:
  - Minimal convention template
  - Complex composable convention template
  - Cross-format convention template

### Language Support

- [ ] JavaScript/TypeScript validator
- [ ] Python validator
- [ ] Julia validator
- [ ] Rust validator

**Milestone:** Production-ready validation tooling available in Python and JavaScript

---

## Phase 3: Expanding Adoption (Q2 2026)

**Goal:** Work to expand adoption and gather real-world feedback.

### Implementation Support

- [ ] Work with library maintainers to add convention support:
  - [ ] Xarray
  - [ ] GDAL
  - [ ] rioxarray
  - [ ] zarrita.js
  - [ ] openlayers
  - [ ] zarr-python
  - [ ] tensorstore
  - [ ] OME tools
- [ ] Document integration patterns for library authors
- [ ] Create reference implementations

### Testing and Feedback

- [ ] Real-world data creation with conventions
- [ ] Cross-tool interoperability testing
- [ ] Performance benchmarking of validation
- [ ] Gather feedback on specification clarity, tooling usability, composability in practice, version management

### Community Resources

- [ ] Create example datasets repository with small reference datasets using various conventions and test fixtures for validation tools
- [ ] Start convention registry/catalog

**Milestone:** 5+ working conventions demonstrating framework capabilities

---

## Phase 4: Discovery and Ecosystem Growth (Q3 2026)

**Goal:** Build discovery mechanisms and grow the conventions ecosystem.

### Discovery Website

- [ ] Create zarr-conventions.github.io website (similar to stac-extensions.github.io)
  - [ ] List all registered conventions
  - [ ] Search and filter functionality
  - [ ] Display convention metadata (name, description, URLs)
  - [ ] Show convention relationships and compositions
  - [ ] Link to schemas and documentation
  - [ ] Show adoption/usage statistics
- [ ] Automated convention submission process
  - [ ] GitHub-based submission via PR
  - [ ] Automated validation of convention metadata
  - [ ] CI checks for schema validity
- [ ] Convention "health" indicators
  - [ ] Schema availability
  - [ ] Documentation completeness
  - [ ] Recent updates/maintenance


### Community Governance

- [ ] Clear escalation paths for issues
- [ ] Contribution guidelines refinement
- [ ] Maintainer onboarding process

---

## Success Metrics

We will track the following metrics to measure the success of the conventions framework:

### Adoption Metrics
- Number of published conventions
- Number of organizations using conventions
- Number of Zarr datasets using explicit conventions
- Number of tools supporting convention validation

### Health Metrics
- Convention documentation completeness
- Schema availability and uptime
- Community engagement (forum posts, GitHub activity)
- Time to create and publish new convention

### Impact Metrics
- Reduced attribute naming collisions
- Improved interoperability between tools
- Faster adoption of new domain-specific features
- Reduction in support requests about metadata interpretation

---

## Open Questions and Risks

### Technical Risks
- **Schema hosting stability**: Conventions depend on URLs remaining available
  - Mitigation: Encourage use of stable hosting (GitHub releases, DOIs), develop caching strategies
  
- **Namespace collisions**: Multiple conventions might use same attribute names
  - Mitigation: Strong documentation on prefixing, community coordination, collision detection tools

- **Validation performance**: Schema validation might be expensive at scale
  - Mitigation: Caching, lazy validation, performance benchmarks

### Community Risks
- **Fragmentation**: Too many similar conventions could confuse users
  - Mitigation: Discovery tools, community review, consolidation discussions

- **Adoption barriers**: Libraries may be slow to add convention support
  - Mitigation: Reference implementations, clear integration guides, direct outreach

- **Maintenance burden**: Conventions may become orphaned
  - Mitigation: Clear ownership model, community takeover process, archival strategy

---

## How to Contribute

We welcome contributions to all aspects of the conventions framework:

- **Specification feedback**: Comment on GitHub issues or discourse
- **Tooling development**: Contribute to validators and tools
- **Convention creation**: Build and share conventions for your domain
- **Documentation**: Improve guides, tutorials, and examples
- **Testing**: Try the framework and report issues

See the [zarr-conventions organization](https://github.com/zarr-conventions) for repositories and contribution guidelines.

## Timeline Summary

| Phase | Timeline | Key Deliverables |
|-------|----------|------------------|
| Phase 1: Finalization | Q4 2025 | Stable spec v1, comprehensive docs |
| Phase 2: Tooling | Q1 2026 | Python/JS validators, enhanced templates |
| Phase 3: Early Adoption | Q2 2026 | 5+ pilot conventions, library integrations |
| Phase 4: Discovery | Q3 2026 | Website, rich ecosystem |
| Phase 5: Maturation | 2026 | Widespread adoption |

---

*This roadmap is a living document and will be updated as the project evolves. Last updated: November 2025*

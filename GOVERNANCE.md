# Governance

## Purpose

This document describes how decisions are made for the Zarr Conventions framework. It covers the maintenance of the framework specification itself, the supporting tooling, and the `zarr-conventions` GitHub organization. It does not govern individual Conventions, which are authored and maintained independently by their own authors.

## Principles

The Zarr Conventions framework is intentionally decentralized:

- **Anyone can author a Convention.** No CDG approval is required to publish or use a new Convention.
- **Adoption is determined by the community.** A Convention's success is measured by uptake among data publishers and tool implementers, not by any central body's endorsement.
- **The CDG governs the framework, not the conventions.** This document describes how the CDG maintains the framework's specification, supporting tooling, and organization. The rationale for this decentralized model is documented in [FAQ.md](FAQ.md).

## Roles

### Core Developers Group (CDG)

The CDG maintains the conventions framework specification, the supporting tooling, and the `zarr-conventions` GitHub organization. The current members of the CDG, who together drafted the framework at the 2025 Zarr Summit, are:

- Alistair Miles ([@alimanfoo](https://github.com/alimanfoo))
- Emmanuel Mathot ([@emmanuelmathot](https://github.com/emmanuelmathot))
- John Kirkham [@jakirkham](https://github.com/jakirkham)
- Joe Hamman ([@jhamman](https://github.com/jhamman))
- Justus Magin ([@keewis](https://github.com/keewis))
- Max Jones ([@maxrjones](https://github.com/maxrjones))
- Ryan Abernathey ([@rabernat](https://github.com/rabernat))
- Even Rouault ([@rouault](https://github.com/rouault))

CDG members may approve and merge pull requests.

### Contributors

Anyone who opens an issue, files a pull request, or otherwise participates in the framework's development is a contributor. Contribution does not require CDG membership.

## Decision-making

### Pull requests

All changes to repositories in the `zarr-conventions` organization are made via pull request. Pull requests require at least one approval from a CDG member before being merged.

For changes to the framework specification (the normative documents and JSON schema in this repository), CDG members SHOULD seek approval from a second CDG member before merging. Documentation, examples, and tooling changes MAY be merged with a single approval at the merging member's discretion.

### Substantive changes

A "substantive change" is any change that modifies the framework specification's normative requirements, changes the JSON schema in a way that affects validation behavior, or establishes a new policy or process for the framework or organization.

Substantive changes SHOULD be discussed openly for at least 7 days before merging, either in the pull request itself or in a linked issue or discussion. CDG members SHOULD respond on the record to substantive objections raised during this period before merging.

### Disagreement

Where CDG members disagree on a pull request, the merge is held while discussion continues. If consensus cannot be reached after the 7-day open-discussion period, any CDG member MAY call for a vote. Votes are open for 14 days. The outcome is decided by a majority of the CDG members who cast a vote. Emeritus members do not vote. Disagreements about whether a particular change is substantive are resolved through the same mechanism.

## CDG membership

### Adding members

Any CDG member may nominate a new member by opening a pull request to this document. Nominations are accepted by lazy consensus: the nomination remains open for at least 7 days, and the nominee is added if no CDG member objects. If any CDG member objects, the nomination proceeds to a vote under the disagreement process above. New members receive the same permissions as existing members.

Nominees are typically contributors who have demonstrated sustained engagement with the framework, though no quantitative criteria are enforced. The CDG values cross-organizational representation and SHOULD consider this when evaluating nominations.

### Stepping down and emeritus status

CDG members may step down at any time by notifying the CDG and removing themselves from the organization owner roster. A member who has had no activity (commits, pull request reviews, or substantive discussion participation in `zarr-conventions` repositories) for 12 months MAY be moved to emeritus status by lazy consensus of the active CDG.

Emeritus members lose merge permissions but retain standing in the project. An emeritus member MAY return to active status by notifying the CDG.

### Removal

In exceptional cases, such as violations of community standards, a CDG member MAY be removed by a 2/3 vote of the active CDG. The vote is open for 14 days. This mechanism is intended for serious conduct issues, not for resolving technical disagreements. Zarr conventions operates within the broader Zarr Project ecosystem and follow the [Zarr Code of Conduct](https://github.com/zarr-developers/.github/blob/main/CODE_OF_CONDUCT.md).

## Relationship to broader Zarr governance

The Zarr Conventions framework operates within the broader Zarr Project ecosystem. The CDG coordinates with the maintainers of the core Zarr specification when framework changes interact with that specification.

[ZEP0011](https://github.com/zarr-developers/zeps/issues/69), currently in draft, proposes a three-tier governance structure under which conventions projects would be governed by Core Developers Groups. If ZEP0011 is accepted, this document MAY be revised to align with the accepted template developed after ZEP0011, and the framework MAY register as an Affiliated Software Project. The decision to seek Affiliated status is independent of internal governance and is not addressed by this document.

## Convention lifecycle

Individual Conventions are authored and maintained outside the `zarr-conventions` organization by their own authors. The CDG does not approve, reject, or rank Conventions; the community decides which Conventions are useful through adoption.

The `zarr-conventions` organization provides an optional central location for Convention discovery, as described in [FAQ.md](FAQ.md). Inclusion in this discovery surface is a curatorial decision made by the CDG and is not an endorsement of correctness or quality.

A process for handling orphaned Conventions, whose authors are no longer responsive, is not yet established. The CDG intends to develop one and will document it via amendment to this document.

## Amendments

This document is amended via pull request following the substantive-change process above.

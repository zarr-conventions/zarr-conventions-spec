# Governance

## Purpose

This document describes how decisions are made for the Zarr Conventions framework. It covers the maintenance of the framework specification itself, the framework repositories listed under [Pull requests](#pull-requests), and the `zarr-conventions` GitHub organization. It does not govern individual Conventions or independently maintained tooling, which are authored and maintained by their own authors.

## Principles

The Zarr Conventions framework is intentionally decentralized:

- **Anyone can author a Convention.** No CDG approval is required to publish or use a new Convention.
- **Adoption is determined by the community.** A Convention's success is measured by uptake among data publishers and tool implementers, not by any central body's endorsement.
- **The CDG governs the framework, not the conventions.** This document describes how the CDG maintains the framework's specification, the framework repositories, and the organization. The rationale for this decentralized model is documented in [FAQ.md](FAQ.md).

## Roles

### Core Developers Group (CDG)

The CDG maintains the conventions framework specification, the other framework repositories listed under [Pull requests](#pull-requests), and the `zarr-conventions` GitHub organization. The current members of the CDG, who together drafted the framework at the 2025 Zarr Summit, are:

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

Discussion is encouraged to happen openly on GitHub. Conversations may also take place in other venues, including the [Zarr Conventions Zulip channel](https://ossci.zulipchat.com/#narrow/channel/604429-Zarr-Conventions), community meetings, and private discussions. Decisions, including votes, are recorded on GitHub in the relevant pull request, issue, or discussion.

### Pull requests

All changes to the framework repositories in the `zarr-conventions` organization are made via pull request. The framework repositories currently under CDG control are:

- [zarr-conventions-spec](https://github.com/zarr-conventions/zarr-conventions-spec) — the framework specification, JSON schema, and supporting documentation
- [.github](https://github.com/zarr-conventions/.github) — organization-level documentation, including the Convention discovery table

For changes to the framework specification (the normative documents and JSON schema in `zarr-conventions-spec`), approvals SHOULD be obtained from two CDG members other than the pull request's author before merging.

All other changes to framework repositories — including documentation, examples, and tooling — require approval from a CDG member or authorship by a CDG member. A pull request's author MAY merge their own pull request once these requirements are met; CDG members merging their own pull requests SHOULD use their judgment as to whether to seek review from another CDG member first.

These requirements do not apply to other repositories hosted in the organization, which are maintained by their own authors under their own review practices. This includes Convention repositories, as described under [Convention lifecycle](#convention-lifecycle), and independently maintained tooling such as [zarr-cm](https://github.com/zarr-conventions/zarr-cm). New repositories in the organization, including those created by CDG members, default to independent maintenance; a repository becomes a framework repository only by being added to the list above by amendment to this document.

### Substantive changes

A "substantive change" is any change that modifies the framework specification's normative requirements, changes the JSON schema in a way that affects validation behavior, or establishes a new policy or process for the framework or organization.

Substantive changes SHOULD be discussed openly for at least 7 days before merging, either in the pull request itself or in a linked issue or discussion. CDG members SHOULD respond on the record to substantive objections raised during this period before merging.

### Disagreement

Where CDG members disagree on a pull request, the merge is held while discussion continues. If consensus cannot be reached after the 7-day open-discussion period, any CDG member MAY call for a vote. Votes are cast on GitHub in the pull request, issue, or discussion where the disagreement arose and are open for 14 days. The outcome is decided by a majority of the CDG members who cast a vote. Emeritus members do not vote. Disagreements about whether a particular change is substantive are resolved through the same mechanism.

## CDG membership

### Adding members

Any CDG member may nominate a new member by opening a pull request to this document. Nominations are accepted by lazy consensus: the nomination remains open for at least 7 days, and the nominee is added if no CDG member objects. If any CDG member objects, the nomination proceeds to a vote under the disagreement process above. New members receive the same permissions as existing members.

Nominees are typically contributors who have demonstrated sustained engagement with the framework, though no quantitative criteria are enforced. The CDG values cross-organizational representation and SHOULD consider this when evaluating nominations.

### Stepping down and emeritus status

CDG members may step down at any time by notifying the CDG and removing themselves from the organization owner roster. A member who has had no activity (commits, pull request reviews, or substantive discussion participation in `zarr-conventions` repositories) for 12 months MAY be moved to emeritus status by lazy consensus of the active CDG.

Emeritus members lose merge permissions but retain standing in the project. An emeritus member MAY return to active status by notifying the CDG.

### Removal

In exceptional cases, such as violations of community standards, a CDG member MAY be removed by a 2/3 vote of the active CDG. The vote is open for 14 days. Given the sensitive nature of conduct issues, discussion of a proposed removal MAY take place in a private venue and the vote MAY be conducted privately; the outcome is recorded on GitHub. This mechanism is intended for serious conduct issues, not for resolving technical disagreements. Zarr Conventions operates within the broader Zarr Project ecosystem and follows the [Zarr Code of Conduct](https://github.com/zarr-developers/.github/blob/main/CODE_OF_CONDUCT.md). Violations may be reported to the Zarr Steering Council as instructed in the [Zarr Code of Conduct](https://github.com/zarr-developers/.github/blob/main/CODE_OF_CONDUCT.md).

## Relationship to broader Zarr governance

The Zarr Conventions framework operates within the broader Zarr Project ecosystem. The CDG coordinates with the maintainers of the core Zarr specification when framework changes interact with that specification.

[ZEP0011](https://github.com/zarr-developers/zeps/issues/69), currently in draft, proposes a three-tier governance structure under which conventions projects would be governed by Core Developers Groups. If ZEP0011 is accepted, this document MAY be revised to align with the accepted template developed after ZEP0011, and the framework MAY register as an Affiliated Software Project. The decision to seek Affiliated status is independent of internal governance and is not addressed by this document.

## Convention lifecycle

Individual Conventions are authored and maintained by their own authors, independently of the CDG. A Convention's repository may be hosted anywhere, including within the `zarr-conventions` organization. Hosting within the organization does not transfer editorial control to the CDG: the Convention's authors remain its maintainers and decide what is merged into its repository. The CDG does not approve, reject, or rank Conventions; the community decides which Conventions are useful through adoption.

The `zarr-conventions` organization provides an optional central location for Convention discovery, as described in [FAQ.md](FAQ.md). Guidance for authoring and sharing a new Convention, including a template repository, is available in the [organization README](https://github.com/zarr-conventions#define-your-own-convention). Convention authors who would like their Convention's repository hosted within the organization may request this by opening an issue in the [organization repository](https://github.com/zarr-conventions/.github). Inclusion in the discovery surface and hosting within the organization are curatorial decisions made by the CDG and are not an endorsement of correctness or quality.

A process for handling orphaned Conventions, whose authors are no longer responsive, is not yet established. The CDG intends to develop one and will document it via amendment to this document.

## Amendments

This document is amended via pull request following the substantive-change process above.

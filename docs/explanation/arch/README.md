# Architecture Decision Records

## Overview

An Architectural Decision (AD) is a software design choice that addresses a
functional or non-functional requirement that is architecturally significant. An
Architecturally Significant Requirement (ASR) is a requirement that has a
measurable effect on a software system’s architecture and quality. An
Architectural Decision Record (ADR) captures a single AD, such as often done
when writing personal notes or meeting minutes; the collection of ADRs created
and maintained in a project constitute its decision log. All these are within
the topic of Architectural Knowledge Management (AKM).

## Adding an ADR

To propose a new ADR, create an initial draft in the `explanations/arch/drafts`
directory by copying the [template](./drafts/adr-template.md). The draft may be
merged as "accepted" if the new ADR is simple enough, otherwise there may be
additional PRs to edit and refine the ADR. Once the ADR PR is merged and
accepted, create a final PR to move the ADR out of drafts and assign it a
number. Along with every accepted ADR added, the following files must be updated

## Information

The documents in this directory record:

- Decisions made.
- Alternatives considered.
- Reasoning to support the decision.

Read [ADR-1 Use ADRs](adr-1-use-adrs.md) for more information.

See [Decision Records](DECISIONS.md) for a chronological listing of all
decisions.

Abbreviations:

    AD: architecture decision

    ADL: architecture decision log

    ADR: architecture decision record

    AKM: architecture knowledge management

    ASR: architecturally-significant requirement

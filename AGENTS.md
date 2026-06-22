# Agent Guidance

This repository contains the Pontmore protocol specifications. Pontmore is a Nostr-native protocol family for agent identity, capability discovery, escrow declaration, and swap lifecycle coordination.

## Scope

Work in this repository should stay focused on protocol specification text. Do not add application implementation plans, product UX, database schemas, deployment notes, or SDK instructions unless the user explicitly asks for protocol text that requires them.

## Terminology

In protocol text, `Agent` means a Pontmore protocol participant that publishes capabilities and can conduct swaps.

When referring to automation working on this repository, use explicit terms such as `AI agent`, `coding assistant`, or `repository automation`.

## Spec Structure

The active PIP series contains the required PIPs for the minimum interoperable core plus one recommended discovery layer:

- `PIP-00-agent-definition.md`: public agent capability and discovery record
- `PIP-01-escrow-descriptor.md`: public escrow declaration referenced by agents and swaps
- `PIP-02-swap-state-machine.md`: request, transition, evidence, dispute, note, and snapshot event lifecycle
- `PIP-03-dispute-policy.md`: dispute classes, timeout classes, evidence boundary, and resolution modes
- `PIP-04-agent-short-code.md`: user-friendly agent identification, verification, and discovery using NIP-05 and NIP-19 (recommended)

Read `README.md` first, then read only the PIPs directly relevant to the requested change.

## Implementation

When reading these specs to build or review an implementation in another repository:

1. Start with `README.md` to understand the protocol definition and active PIP set.
2. Read all required PIPs in order: `PIP-00`, `PIP-01`, `PIP-02`, then `PIP-03`.
3. Treat Nostr identity, public agent definitions, escrow descriptors, swap events, and dispute policy as the protocol surface.
4. Model operator accounts, dashboards, indexes, moderation tools, and private databases as implementation overlays, not canonical protocol state.
5. Keep public protocol facts separate from private operator judgments, internal notes, KYC data, payment instructions, screenshots, and local account records.
6. If an implementation needs local conveniences such as sessions, API keys, indexes, queues, or webhooks, derive them from the protocol instead of redefining the protocol around them.

Implementation plans should identify which PIP each protocol behavior comes from. If the specs are silent, call that out as an implementation assumption instead of presenting it as Pontmore behavior.

## Contribution

- Keep each change atomic by protocol component.
- Keep one PIP as the primary source of truth for each rule.
- Update related PIPs only where consistency requires it.
- Update `README.md` when adding, removing, or renumbering a PIP.
- Preserve the distinction between canonical public protocol state and operator-layer overlays.
- Prefer documenting draft conventions or open questions over implying false finality.

## Final Checks

Before finishing, verify that:

- cross-links point to real files
- PIP numbers and filenames agree
- required/recommended/optional labels match the intended implementation burden
- new normative statements do not contradict related PIPs
- no product-specific assumptions have leaked into protocol text

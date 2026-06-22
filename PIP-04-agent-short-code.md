# PIP-04: Agent Short Code

## Status

- Status: Draft
- Implementation: Recommended
- Scope: user-friendly agent identification, verification, and discovery using NIP-05 and NIP-19
- Related:
  - [PIP-00-agent-definition.md](./PIP-00-agent-definition.md)
  - [PIP-02-swap-state-machine.md](./PIP-02-swap-state-machine.md)

## Purpose

This document defines the agent short code: a compact, human-friendly identifier that lets users discover, verify, and share Pontmore agents without handling raw pubkeys.

The short code reuses two existing Nostr standards:

- NIP-05 for DNS-backed mapping from a memorable handle to a canonical hex pubkey
- NIP-19 for bech32-encoded shareable artifacts that bundle the pubkey with relay hints

The short code is a discovery and display surface. It does not replace the pubkey as canonical agent identity.

## Core Rule

- **the Nostr pubkey is the canonical agent identity**
- **the short code is a verified alias for discovery, display, and sharing**

Clients MUST always follow the pubkey, not the short code. If a short code ever resolves to a different pubkey, clients MUST NOT silently replace a previously established agent reference. The short code is an entry point, not a persistent identity.

## Short Code Format

A short code is a NIP-05 internet identifier:

```text
<handle>@<domain>
```

- `<handle>` is the NIP-05 local part
- `<domain>` is the NIP-05 domain that serves `.well-known/nostr.json`

The full NIP-05 character set is permitted. For agent short codes, handles SHOULD be:

- lowercase
- 3 to 16 characters
- limited to `a-z`, `0-9`, and `-`
- free of leading or trailing hyphens

These constraints are recommendations for memorability and consistent display, not a hard restriction on the NIP-05 charset. Clients MUST accept any valid NIP-05 identifier.

### Domain Root

A short code MAY use the NIP-05 root form `_@<domain>`, in which case clients SHOULD display it as just `<domain>`.

## Declaration

The short code is declared through two cooperating events. No new event kind is introduced.

### Kind 0 Profile Event

The agent's kind 0 profile event SHOULD include a `nip05` field set to the short code:

```json
{
  "name": "K45 Sud",
  "about": "Pontmore swap agent serving East Africa",
  "nip05": "k45sud@pontmore.example"
}
```

This is standard NIP-05. The kind 0 event is the primary verification anchor for the short code.

### Agent Definition Event

The agent definition event defined in [PIP-00](./PIP-00-agent-definition.md) SHOULD reference the short code in two places:

1. A `nip05` tag on the event, for relay filtering and indexing:

   ```text
   ["nip05", "k45sud@pontmore.example"]
   ```

2. A `short_code` field inside the versioned `content` JSON:

   ```json
   "short_code": "k45sud@pontmore.example"
   ```

The `short_code` content field is the canonical declaration on the agent definition. The `nip05` tag is an index and discovery aid. Clients MUST NOT treat a `nip05` tag as authoritative unless the same value is present in `content.short_code`.

## Resolution

To resolve a short code to a canonical agent, a client SHOULD:

1. Split the short code into `<handle>` and `<domain>`.
2. Fetch `https://<domain>/.well-known/nostr.json?name=<handle>`.
3. Read the hex pubkey from `names[<handle>]`.
4. Fetch the kind 0 profile event for that pubkey.
5. Verify that the kind 0 `nip05` field matches the original short code.
6. Fetch the agent definition event (kind `30360`) for that pubkey.
7. Verify that `content.short_code` matches the original short code.

If any verification step fails, the client MUST treat the short code as unverified and SHOULD warn the user before proceeding.

The `.well-known/nostr.json` response MAY include a `relays` map as described in NIP-05. Clients SHOULD use the listed relays as hints for fetching the agent's profile, agent definition, and relay list.

### Multiple Agent Profiles

An identity may publish multiple agent definitions with different `d` tags, as described in [PIP-00](./PIP-00-agent-definition.md). A short code resolves to one pubkey. The short code does not select a specific `d` tag.

When a resolved pubkey has multiple agent definitions, clients SHOULD default to the `agent` profile unless the user or a deeper reference selects a more specific profile.

## Sharing

For sharing an agent outside of a typed short code, clients SHOULD use NIP-19 `nprofile` encoding. An `nprofile` bundles the agent pubkey with relay hints, letting recipients locate the agent without re-resolving the short code:

```text
nprofile1qqsrhuxx8l9ex335q7he0f09aej04zpazpl0ne2cgukyawd24mayt8gpp4mhxue69uhhytnc9e3k7mgpz4mhxue69uhkg6nzv9ejuumpv34kytnrdaksjlyr9p
```

Clients MAY also share the short code directly. The two forms serve different contexts:

- **short code**: human-typed, human-read, DNS-verified entry point
- **nprofile**: copy-paste and QR-code shareable artifact with relay hints

Neither form replaces the hex pubkey as the canonical identifier used in protocol events.

## Display

Clients that support short codes SHOULD:

- display the short code alongside or instead of the raw `npub` when one is verified
- continue to display the `npub` or `nprofile` when no verified short code exists
- show the short code in a visually distinct way from free-text names so users can tell it is a verified identifier

Clients MUST NOT present an unverified short code as if it were confirmed by DNS lookup.

## Use in Swap Events

Swap request events defined in [PIP-02](./PIP-02-swap-state-machine.md) identify participants by pubkey. The short code is not used inside swap content as a participant identifier.

A short code MAY appear in a human-readable note event (kind `7304`) or in client UI as a display label for an agent referenced by pubkey. The swap state machine itself MUST use pubkeys for all normative participant references.

## Verification Boundary

Short code verification is a discovery-time check, not a protocol-state fact. Specifically:

- verification confirms that a handle and domain currently map to a pubkey
- verification does not certify trustworthiness, operator status, or escrow solvency
- a verified short code does not override the agent definition, escrow descriptor, or dispute policy

Trust and capability assertions remain the responsibility of [PIP-00](./PIP-00-agent-definition.md), [PIP-01-escrow-descriptor.md](./PIP-01-escrow-descriptor.md), and [PIP-03-dispute-policy.md](./PIP-03-dispute-policy.md).

## Open Questions

- Whether a Pontmore-specific well-known path or response shape is needed beyond standard NIP-05, or whether plain NIP-05 is sufficient for all agent discovery cases.
- Whether short codes should support a Pontmore-native suffix or tag to distinguish agent handles from general Nostr handles in mixed directories.

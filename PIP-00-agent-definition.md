# PIP-00: Agent Definition

## Status

- Status: Draft
- Implementation: Required
- Scope: agent discovery and public capability declaration
- Related:
  - [PIP-01-escrow-descriptor.md](./PIP-01-escrow-descriptor.md)

## Purpose

This document defines the public agent definition event used for discovery and capability declaration.

## Event Type

- kind: `30360`
- addressable
- `d` tag: stable identifier, usually `agent`

## Discovery Inputs

An agent is discoverable through:

- a Nostr pubkey
- a normal Nostr profile event
- a relay list event
- an agent definition event

## Required Tags

- `d`
- `t`, `agent`
- `relay`
- `a`
  - reference to the default escrow descriptor event

## Recommended Tags

- `f`
  - fiat currency supported by this profile
- `r`
  - operator or service URLs
- `client`
  - optional handler hint for compatible clients

## Content Schema

`content` is JSON and MUST be versioned.

Minimum expected fields:

- `version`
- `name`
- `about`
- `capabilities`
- `pricing_policy`
- `escrow`
- `updated_at`

## Capability Surface

The agent definition should describe:

- supported swap types
- supported fiat currencies
- supported payment channels
- supported settlement networks
- regions or markets served
- min and max limits
- pricing or margin policy references

## Multiple Profiles

An identity may publish multiple agent definitions with different `d` tags for different contexts.

Suggested examples:

- `agent`
- `agent:ke`
- `agent:staging`

Clients should treat `agent` as the default public profile unless a more specific profile is requested.

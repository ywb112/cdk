# DeltaCrafting License System (Design Scaffold)

This repository is intended to host the **public, signed license-revocation manifest** for the DeltaCrafting desktop client.

## Security rules

- Never commit the license-signing private key, GitHub tokens, passwords, full license keys, or personal data.
- The private signing key must be generated and stored locally by the owner. The client contains only the matching public verification key.
- The client must verify the manifest's digital signature before trusting any data in it.
- A missing, invalid, stale, or unreachable manifest must fail closed if strict online startup verification is enabled.
- GitHub is static file hosting here, not a private database or a full license API. It cannot reliably record every activation or provide instantaneous revocation.
- A public revocation manifest should contain only random license identifiers (or their hashes), never the full license key or device details.

## Planned layout

- `licenses/manifest.json`: public revocation state, once finalized.
- `licenses/manifest.sig`: detached signature for the exact manifest bytes, once signing is implemented.
- `docs/PROTOCOL.md`: protocol and implementation notes.

## Current status

**Design scaffold only.** No production license has been issued. The manifest signature, issuer tool, and DeltaCrafting client integration have not yet been implemented or tested. Do not point a released client at this repository until signature verification and fail-closed behavior have been implemented and tested.

## Proposed license types

- `monthly`: 30 days from the defined activation/issue time.
- `yearly`: 365 days from the defined activation/issue time.
- `permanent`: no scheduled expiration.

For the first release, licenses should be bound to one device. The issuer's private ledger should track issued license IDs and device bindings locally.

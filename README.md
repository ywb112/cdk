# DeltaCrafting License System

This public repository hosts the signed revocation manifest used by the DeltaCrafting desktop client.

## Security rules

- Never commit the license-signing private key, GitHub tokens, passwords, full license keys, or personal data.
- The issuer private key stays on the owner's computer. The client contains only the matching public verification key.
- The client verifies the ECDSA/SHA-256 signature before trusting any manifest data.
- The public manifest contains only random license IDs that have been revoked, never full tokens, device codes, names, or contact details.
- GitHub is static file hosting, not a licensing API. It cannot reliably record activations or guarantee instantaneous revocation.

## Files

- `licenses/manifest.signed.json`: one-file envelope containing base64-encoded exact payload bytes and their signature.
- `docs/PROTOCOL.md`: protocol, issuer workflow, and limitations.

## Current status

The repository now has an initial signed manifest with an empty revocation list. The local source modification includes client-side online checks and issuer commands, but the Windows build and end-to-end behavior still need to be tested before distribution. No production licenses have been issued.

## License plans

- `monthly`: 30 days from issue time.
- `yearly`: 365 days from issue time.
- `permanent`: no scheduled expiration.

Licenses are cryptographically bound to one device. The issuer's private ledger is stored locally.

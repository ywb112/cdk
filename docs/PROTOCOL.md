# License protocol (implementation plan)

## Goals

- No paid VPS for the first version.
- One license is bound to one device.
- Supported plans: monthly (30 days), yearly (365 days), permanent.
- Client checks online on every startup; if it cannot fetch and verify current revocation state, it must not open the protected main UI.
- Owner can revoke a license by publishing a newly signed manifest.
- Keep license issuing private: the issuer private key never enters this repository or the client.

## Important architecture constraint

A static public GitHub repository is suitable for distributing a signed revocation manifest, but it is not a trusted transactional licensing server. It cannot securely enforce first-activation races, prove that a client activated, or reliably log last-seen timestamps. Since the repository is public, it must not contain license keys, user names, raw device IDs, or private activation records.

## Proposed flow

1. Owner generates an asymmetric signing key pair locally.
2. Client embeds only the public verification key and the manifest URL.
3. Owner issues a license locally. The local issuer ledger stores the license ID, key hash, plan, issue/expiry times, and device binding.
4. Activation binds the license to the device and produces a signed license receipt. For a truly offline first activation, activation cannot be remotely confirmed; this design instead requires internet access and a separate owner-side workflow or future API for secure first-activation enforcement.
5. At startup, client downloads the manifest bytes, verifies the detached signature against the embedded public key, checks freshness/version, and denies entry if network access or verification fails.
6. Client checks whether the license ID appears in the revocation list and validates the signed receipt, expiry, and device binding.
7. To revoke, owner updates the private issuer ledger, generates a new manifest, signs it locally, and publishes both manifest and signature.
8. Optional running-session checks repeat the manifest validation every few minutes; revocation takes effect when a newer manifest is fetched and verified.

## Public manifest shape (illustrative only)

```json
{
  "schemaVersion": 1,
  "manifestVersion": 1,
  "generatedAtUtc": "2026-01-01T00:00:00Z",
  "revokedLicenseIds": []
}
```

The manifest must be serialized canonically or signed as exact bytes. The signature must cover the exact downloaded manifest bytes. Do not accept unsigned content or silently fall back to cached authorization when strict online mode is selected.

## Publishing and caching

- Publish `licenses/manifest.json` and `licenses/manifest.sig` together in a single commit.
- Client must use HTTPS and verify the signature before parsing/trusting content.
- Track `manifestVersion` to reject rollback to an older manifest where possible.
- GitHub CDN caching can delay revocation propagation; this is not instant remote control.
- If GitHub is unavailable, strict fail-closed behavior means the software will not start, even for a valid permanent license.

## Still to implement and test

- Key-pair generation and safe private-key storage.
- Canonical manifest serialization and detached signature generation.
- Local issuer app and protected issuer ledger.
- Device fingerprint strategy and reset workflow.
- Client startup gate, online fetch, signature verification, expiry and revocation checks.
- Update-source decoupling from upstream project.
- Windows build and end-to-end tests.

This document describes a design, not a completed or tested licensing implementation.

# DeltaCrafting online license protocol

## Goals

- Avoid a paid VPS for the first version by using this public GitHub repository as static manifest hosting.
- One license is bound to one device.
- Plans: monthly (30 days), yearly (365 days), and permanent.
- Client checks online at activation and every app startup. While running, it rechecks every five minutes.
- If the network is unavailable or the signed manifest cannot be fetched or verified, the protected app is denied/closed (strict fail-closed behavior).
- Owner can revoke a license by publishing a newer signed manifest.
- The private signing key never enters this repository or the client.

## Public manifest format

`licenses/manifest.signed.json` is an envelope with:
- `payloadBase64`: base64 of the exact UTF-8 JSON payload bytes.
- `signatureBase64`: ECDSA/SHA-256 signature in RFC 3279 DER sequence format over those exact payload bytes.

The decoded payload has:
- `schemaVersion`: currently 1.
- `manifestVersion`: monotonically increasing integer.
- `generatedAtUtc`: UTC timestamp.
- `revokedLicenseIds`: array of random 32-hex license IDs.

The client verifies the signature using its embedded public key before parsing or trusting the payload. It stores the highest seen manifest version locally and rejects lower versions.

## License tokens

The owner signs device-bound license tokens locally. A token contains a random license ID (the `nonce`), product, device code, plan, issue time and optional expiry. The public manifest contains only the license ID when revoked; never publish the full token or device code.

## Issuing and revoking

- Keep the private key at `%LocalAppData%\\DeltaCrafterLicense\\issuer-private-key.pem` on the owner's PC.
- Issue a card with `dotnet run --project .\\tools\\DeltaCrafter.LicenseIssuer -- <device-code> <monthly|yearly|permanent>`.
- The issuer prints a 32-hex license ID. Record that ID; do not use the full card token as a revocation ID.
- Revoke with `dotnet run --project .\\tools\\DeltaCrafter.LicenseIssuer -- revoke <license-id>`.
- The tool writes a new signed manifest to `%LocalAppData%\\DeltaCrafterLicense\\manifest.signed.json`.
- Upload that file to this repository, replacing `licenses/manifest.signed.json`.
- Keep the local revocation ledger and signed manifest backed up. If local state is lost, download the latest manifest from GitHub before generating a new one so the version number does not go backwards.

## Update source

The app's manual update query is configured for the owner's fork `ywb112/DeltaCrafting`; automatic update checks at startup have been disabled. A GitHub Release must be published there before the manual update check can find a version.

## Limitations

- GitHub static hosting is not a transactional licensing server. It cannot securely arbitrate two simultaneous first activations or reliably report actual activations/last-seen times.
- CDN propagation may delay revocation. It takes effect when the client successfully retrieves the newer signed manifest.
- Strict fail-closed means a GitHub outage also blocks startup, including for permanent cards.
- This is not absolute anti-tamper protection; a sufficiently modified client can bypass local checks. Test the Windows build before distribution.

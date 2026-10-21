# Encrypted short-code records

Short-code clients read `licenses/cards/<record-id>.json`.

- A record ID is a domain-separated SHA-256 hash of a random 80-bit redemption code.
- Each record contains only `version`, `nonce`, `ciphertext`, and `tag` (AES-256-GCM).
- The redemption code and signing private key are never uploaded. Device details and the signed license token are inside the encrypted payload.
- Decryption is only a lookup step. The client still verifies the original ECDSA signature, machine binding, expiry and the signed revocation manifest.
- Issuer receipts, GitHub credentials, license ledgers and private keys must stay on the owner's computer.
- Old long license tokens remain compatible.

This is static file hosting. It does not count activations, bind an unbound code to a device, or guarantee immediate CDN propagation. Short codes are issued for a device in advance.

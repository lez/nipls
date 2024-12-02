# ANIP-02-3c565f3dfeef3

## Signer NIP-44 Custom Salt

A signer is a software as described in NIP-07/46/55.

HKDF is NIP-44's key derivation function and it always
uses "nip44-v2" as salt.

This NIP allow using a salt string other than the above with
[up to 32 bytes](https://github.com/nostr-protocol/nips/pull/940#issuecomment-1868027273)
for any of the NIP-44 signer methods that uses that key
derivation function internally.

### Update to NIP-44 "encrypt" and "decrypt" methods

The custom salt string is the third argument of the "encrypt"
and "decrypt" methods.

```js
window.nostr.nip44.encrypt(pubkey, plaintext, "a custom salt")
window.nostr.nip44.decrypt(pubkey, ciphertext, "a custom salt")
```

### Use Case

Each user device may elect a different salt that it keeps locally stored
to encrypt data that the other devices won't be able to decrypt.

# ANIP-02-3c565f3dfeef3

## Signer NIP-44 Custom Salt

A signer is a software as described in NIP-07/46/55.

HKDF is NIP-44's key derivation function and it always
uses "nip44-v2" as salt.

This NIP allow using a string other than the above
of up to 32 bytes as salt for any of the NIP-44 signer
methods that uses that key derivation function internally.

> [!IMPORTANT]
> Honest signers must REJECT a method call
> if the involved salt string starts with "!".

> [!NOTE]
> All data generated with a salt prefix with "!" is meant to
> never leave the signer, i.e. to not be handled over to an app.

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

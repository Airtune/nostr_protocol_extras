# Generate storage private key from wallet private key

Generate `storage_private_key_hex` as defined in [wallet_to_nostr_keys.md].

# Create wallet contacts event

```json
{
  ...
  "kind": 10033,
  "content": <encrypted wallet contacts>,
  ...
}
```

Events in range `10000 <= n < 20000` are replaceable meaning only the latest event for the `pubkey` and `kind` combination is expected to be stored by relays.

## Unencrypted wallet contacts JSON

`c` is the chain type:

* "evm" for EVM-like chains
* "btc" for Bitcoin
* "ban" for Banano
* "xno" for XNO

`i` is an optional chain_id for blockchains with unique chain identifiers:
https://chainlist.org

`a` is the address.

`alt` is a human readable text from the user to explain what contact address it is.

```json
[
  {
    "c": "evm",
    "i": 1,
    "a": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "alt": "Vitalik"
  },
  {
    "c": "evm",
    "i": 137,
    "a": "0xDb46d1Dc155634FbC732f92E853b10B288AD5a1d",
    "alt": "Lens Protocol Contract (Polygon)"
  },
  {
    "c": "ban",
    "a": "ban_1burnbabyburndiscoinferno111111111111111111111111111aj49sw3w",
    "alt": "Burn address (Banano)"
  },
  {
    "c": "xno",
    "a": "nano_1111111111111111111111111111111111111111111111111111hifc8npp",
    "alt": "Burn address (Nano)"
  }
]
```

## Encrypted wallet contacts

Now that you have `storage_private_key_hex` calculated and the unencrypted wallet contacts JSON, you encrypt the it as if you're sending an encrypted DM to yourself:

```js
// https://github.com/nbd-wtf/nostr-tools/blob/de72172583a3059b010791e5719b47405b7a6a29/keys.ts#L8C17-L8C29
const storage_public_key_hex = getPublicKey(storage_private_key_hex);
// https://github.com/nbd-wtf/nostr-tools/blob/de72172583a3059b010791e5719b47405b7a6a29/nip44.ts#L9C19-L9C19
const encryption_key = sha256(secp256k1.getSharedSecret(storage_private_key_hex, '02' + storage_public_key_hex).subarray(1, 33));
const wallet_contacts_json = JSON.stringify(wallet_contacts);
// https://github.com/nbd-wtf/nostr-tools/blob/de72172583a3059b010791e5719b47405b7a6a29/nip44.ts#L12
const encrypted_wallet_contacts = encrypt(encryption_key, wallet_contacts_json);
```

The client can then decrypt the contacts:
```js
const wallet_contacts_json = decrypt(encryption_key, encrypted_wallet_contacts);
const wallet_contacts = JSON.parse(wallet_contacts_json);
```

# Authentication

The client should be authenticated with NIP-42 by the relay before the relay shares kind 10033 events so clients only can fetch their own contacts events.

https://github.com/nostr-protocol/nips/blob/master/42.md


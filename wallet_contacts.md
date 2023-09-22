# Generate storage private key from wallet private key

A storage private key is derived from the wallet private key. The storage private key is used to sign events on Nostr for storing contact info for the wallet. The wallet private key is used in uppercase hex form, e.g.: `"07923CD2DE62E375F4B4463C877A8E5F746C06843A2C3DD6D5C859D72BA41540"`.

`"_nostr_storage"` is appended to the upper case hex private key and the SHA-256 digest is used as the storage private key on Nostr.

E.g.:
```
storage_private_key = sha256Digest("07923CD2DE62E375F4B4463C877A8E5F746C06843A2C3DD6D5C859D72BA41540_nostr_storage")
```

For a wallet with multiple accounts derived from a seed, use the first account from the seed.

## Javascript example

Convert wallet private key to storage private key bytes
```js
async function get_storage_private_key(wallet_private_key_hex) {
  if (typeof(wallet_private_key_hex) !== 'string' || !/^[a-fA-F0-9]{64}$/.test(wallet_private_key_hex)) {
    return null;
  }
  const uppercase_hex = wallet_private_key_hex.toUpperCase();
  // bring your own sha256 or use the one below in the extras section
  const storage_private_key_bytes = await sha256Digest(`${uppercase_hex}_nostr_storage`);
  // bring your own bytesToHex or use the one below in the extras section
  return bytesToHex(storage_private_key_bytes);
}
```

Expected value to make sure the hashing is set up correct

```js
// private key index 0 used for the whole wallet derived from the seed
const wallet_private_key_hex = "0000000000000000000000000000000000000000000000000000000000000000";
const storage_private_key_hex = await get_storage_private_key(wallet_private_key_hex);
// => "0b90b0d209b82b79f830e00e66f824441f82f8b7f5120170dda41ba8b1af4710"
if (storage_private_key_hex === '0b90b0d209b82b79f830e00e66f824441f82f8b7f5120170dda41ba8b1af4710') {
  console.log("success");
} else {
  console.error("error generating storage secret");
}
```

## Javascript Extras

### `sha256Digest(message)`

Using SubtleCrypto in browsers for the SHA-256 digest
https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest#converting_a_digest_to_a_hex_string

```js
async function sha256Digest(message) {
  const encoder = new TextEncoder();
  const data = encoder.encode(message);
  const hash = await crypto.subtle.digest("SHA-256", data);
  return hash;
}
```

### `bytesToHex(arrayBuffer)`
```js
function bytesToHex(arrayBuffer)
{
  return [...new Uint8Array(arrayBuffer)].map((v) => v.toString(16).padStart(2, '0')).join('');
}
```

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


# Generate Nostr private key from wallet private key

TODO: Generate message signature to derive private key from instead of hashing private key.

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

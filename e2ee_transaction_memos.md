# E2EE Transaction Memos for $XNO and Banano

Using a custom kind 1119 picked from the range `1000 <= n < 10000` for a regular stored event as defined in:
https://github.com/nostr-protocol/nips/blob/master/01.md#kinds

# Event

```json
{
  ...
  "kind": 1119,
  "pubkey": <shared secret memo public key>,
  "content": <encrypted memo json>,
  ...
}
```

By generating a shared secret between the two wallet accounts, a shared memo account keypair is derived for Nostr.

# Authentication

To fetch events, the relay should authenticate the user and verify that they have access to the shared memo account.

# E2EE Transaction Memo

## Unencrypted transaction memo JSON

```json
{
  "h": <transaction hash>,
  "c": <chain type>,
  "i": <chain identifier>,
  "alt": <memo>
}
```

TODO: Somehow separate memos from sender and receiver. Maybe using a wallet signature? It's fair to assume the sender is the one attaching a memo.

The chain type can be:

* "evm" for EVM-like blockchains
* "btc" for Bitcoin
* "xno" for $XNO
* "ban" for Banano

The chain identifiers for EVM chains:

http://chainlist.org

# Derive key memo keys from shared secret


## ECDH shared secret on secp256k to Nostr memo private key and public key

```js
const shared_secret_hex = sha256(secp256k1.getSharedSecret(secp256k1_private_key, '02' + secp256k1_public_key).subarray(1, 33));
```

## Banano shared secret 

```js
// https://github.com/BananoCoin/bananojs/blob/9fb91c566cd3bf653fd5c37de2f52ba1c2f2b357/app/scripts/camo-util.js#L68
const shared_secret_hex = bananojs.getSharedSecret(banano_private_key, banano_public_key);
```

## Memo keypair

```js
const memo_encryption_key = sha256(`${shared_secret_hex}_encryption_key`);
const memo_private_key    = sha256(`${shared_secret_hex}_memo`);
// https://github.com/nbd-wtf/nostr-tools/blob/de72172583a3059b010791e5719b47405b7a6a29/keys.ts#L8C17-L8C29
const memo_public_key = getPublicKey(memo_private_key);
```

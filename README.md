# noble-ed25519

[ed25519](https://en.wikipedia.org/wiki/EdDSA), an elliptic curve that could be used for assymetric encryption and EDDSA signature scheme.

Supports [ristretto255](https://ristretto.group).

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies, one small file
- Easily auditable TypeScript/JS code
- Uses es2019 bigint. Supported in Chrome, Firefox, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [bls12-381](https://github.com/paulmillr/noble-bls12-381),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160),
  [secretbox-aes-gcm](https://github.com/paulmillr/noble-secretbox-aes-gcm)

## Usage

> npm install noble-ed25519

```js
import * as ed25519 from "noble-ed25519";

const PRIVATE_KEY = 0xa665a45920422f9d417e4867efn;
const HASH_MESSSAGE = new Uint8Array([99, 100, 101, 102, 103]);

(async () => {
  const publicKey = await ed25519.getPublicKey(PRIVATE_KEY);
  const signature = await ed25519.sign(HASH_MESSAGE, PRIVATE_KEY);
  const isMessageSigned = await ed25519.verify(signature, HASH_MESSAGE, publicKey);
})();
```

## API

- [`getPublicKey(privateKey)`](#getpublickeyprivatekey)
- [`sign(hash, privateKey)`](#signhash-privatekey)
- [`verify(signature, hash, publicKey)`](#verifysignature-hash-publickey)
- [Helpers & Point](#helpers--point)

##### `getPublicKey(privateKey)`
```typescript
function getPublicKey(privateKey: Uint8Array): Promise<Uint8Array>;
function getPublicKey(privateKey: string): Promise<string>;
function getPublicKey(privateKey: bigint): Promise<Point>;
```
- `privateKey: Uint8Array | string | bigint` will be used to generate public key.
  Public key is generated by executing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
- Returns:
    * `Promise<Uint8Array>` if `Uint8Array` was passed
    * `Promise<string>` if hex `string` was passed
    * `Promise<Point(x, y)>` instance if `bigint` was passed
    * Uses **promises**, because ed25519 uses SHA internally; and we're using built-in browser `window.crypto`, which returns `Promise`.

##### `sign(hash, privateKey)`
```typescript
function sign(hash: Uint8Array, privateKey: Uint8Array | bigint): Promise<Uint8Array>;
function sign(hash: string, privateKey: string | bigint): Promise<string>;
```
- `hash: Uint8Array` - message hash which would be signed
- `privateKey: Uint8Array | bigint` - private key which will sign the hash
- Returns EdDSA signature. You can consume it with `SignResult.fromHex()` method:
    - `SignResult.fromHex(ed25519.sign(hash, privateKey))`

##### `verify(signature, hash, publicKey)`
```typescript
function verify(
  signature: Uint8Array | string | SignResult,
  hash: Uint8Array | string,
  publicKey: string | Point | Uint8Array
): Promise<boolean>
```
- `signature: Uint8Array` - object returned by the `sign` function
- `hash: string | Uint8Array` - message hash that needs to be verified
- `publicKey: string | Uint8Array | Point` - e.g. that was generated from `privateKey` by `getPublicKey`
- Returns `Promise<boolean>`: `Promise<true>` if `signature == hash`; otherwise `Promise<false>`

##### Helpers & Point

```typescript
// 𝔽p
ed25519.P // 2 ^ 255 - 19

// Subgroup order
ed25519.PRIME_ORDER // 2 ^ 252 - 27742317777372353535851937790883648493

// Elliptic curve point
ed25519.Point {
  static fromY(y: bigint);
  static fromHex(hash: string);
  constructor(x: bigint, y: bigint);
  toHex(): string; // Compact representation of a Point
  toX25519(): bigint; // To montgomery
  encode(): Uint8Array;
  add(other: Point): Point;
  subtract(other: Point): Point;
  multiply(scalar: bigint): Point;
}
ed25519.SignResult {
  constructor(r: bigint, s: bigint);
  toHex(): string;
}

// Base point
ed25519.BASE_POINT // new ed25519.Point(x, y) where
// x = 15112221349535400772501151409588531511454012693041857206046113283949847762202n;
// y = 46316835694926478169428394003475163141307993866256225615783033603165251855960n;

// Example usage:
ed25519.BASE_POINT.multiply(65537n);
```

There are additional `ristretto255` helpers in `ristretto255.js` file.

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. Overall they are quite rare; for our particular usage they're unimportant. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Try LibreSSL & similar low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.

## License

MIT (c) Paul Miller [(https://paulmillr.com)](https://paulmillr.com), see LICENSE file.

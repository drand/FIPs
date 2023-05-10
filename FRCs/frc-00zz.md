## Simple Summary
Timelock encryption is a method of encrypting a message now that can't be decrypted until a specific point in the future has passed. The drand team have launched a scheme that uses BLS12-381 signatures in an innovative way using [identity based encryption](https://crypto.stanford.edu/~dabo/papers/bfibe.pdf) to achieve practical Timelock encryption using the drand network.
As the Filecoin network already supports BLS12-381, it could trivially implement built-in FVM actors for enabling timelock encryption (providing the [FIP for switching to the new drand network](https://github.com/filecoin-project/FIPs/pull/652) is accepted).


## Abstract
On 2023-03-01, drand launched the first practical Timelock encryption scheme to drand mainnet after undergoing a security audit and developing libraries and tooling to make it easier.
Timelock encryption would enable users to do sealed-bid auctions, create more secure implementations of [RANDAO](https://github.com/randao/randao), schedule social media network messages, prevent [MEV](https://coinmarketcap.com/alexandria/glossary/miner-extractable-value-mev), and much, much more!
The drand timelock scheme uses [identity based encryption](https://crypto.stanford.edu/~dabo/papers/bfibe.pdf) which exploits the pairing capabilities of BLS12-381 to use a future round number as a public key to encrypt messages which will be decryptable only once that future round has been emitted by the drand network by using the round signature as a private key.

We propose a precompile for FVM that would enable users to timelock encrypt arbitrary data and attempt decryption of previously timelock encrypted data. This would enable users to timelock encrypt some part of the data inside their smart contract, while also enabling users to pass ciphertexts directly into smart contracts for decryption there at a later time. 

## Change Motivation
To enable timelock encryption and decryption using primitives that are already available in the FVM internals.

## Specification

The following interfaces should be fulfilled in FEVM and native actors respectively:

```solidity
interface Timelock {
    function decrypt(bytes message) returns (bytes, bool)
}
```

```rust
trait Timelock {
    fn decrypt(message: Vec<u8>) -> Result<Vec<u8>>
}
```

The interfaces only contain a decrypt method, as due to the public nature of blockchains and the pool of transactions submitted to them, it does not make sense to ask the network to encrypt a plaintext on your behalf as everybody in the network would be able to see the plaintext. For encryption, we suggest users follow the methods specified in [the timelock encryption paper](https://eprint.iacr.org/2023/189.pdf) released by the team (and others), or use one of the supported libraries. 
In short, users can take raw message bytes and encrypt them using the drand block number corresponding to the decryption time as a public key. The block number is hashed to a point on the G2 group of the BLS12-381 curve. That point is multiplied by a new point derived from the message and mapped onto the target group Gt, and the message is xor'd with the resulting point on Gt. 
For messages larger than 128bits, we recommend using symmetric encryption such as [AGE](https://age-encryption.org/) to encrypt the message, and performing timelock encryption on just the symmetric key as the pairing operation is expensive and decryption would cost a large amount of gas on-chain.
The above interfaces assume that details such as the drand network chain hash and public key are transparent for the user - should the Filecoin network switch to an alternative drand network in future, users may need to interact with different actor addresses to timelock encrypt/decrypt for the respective drand network. We assume this is in line with standard practice.

To decrypt a message, a storage provider takes the drand randomness for the given round (our AGE-powered libraries bundle this round number in the ciphertext itself), and uses it as a private key to decrypt the ciphertext as per [the timelock encryption paper](https://eprint.iacr.org/2023/189.pdf).
Whether users store ciphertext to round number tuples or the network supports ciphertexts in the AGE/[armor](https://datatracker.ietf.org/doc/html/rfc4880#section-6.2) format transparently is a matter for community debate.

Users can store timelock encrypted payloads in smart contracts using either the `Vec<u8>` or `&[u8]` types in rust, or a `bytes` type in Solidity.

## Test Cases

- ciphertexts should be decryptable once the relevant time has been reached
- ciphertexts should not be decryptable before the relevant time has been reached
- decryption should be compatible with the existing implementations [tlock](https://github.com/drand/tlock) and [tlock-js](https://github.com/drand/tlock-js)
- users cannot encrypt to a time before introduction of the new scheme (i.e. when [FIP-0063](https://github.com/filecoin-project/FIPs/pull/652)) was accepted and deployed to the Filecoin network), as the signatures before this time will not conform with the scheme for timelock encryption.

## Backwards compatibility

There are no backwards compatibility considerations, as this feature has not been exposed on the FVM until now.

## Security considerations

As we recommend using hybrid encryption with timelock, users unfamiliar with cryptographic best practices may choose to use weak or poorly implemented schemes to encrypt their data. We suggest using battle-tested [AEAD](https://en.wikipedia.org/wiki/Authenticated_encryption) schemes such as [AES-GCM](https://www.rfc-editor.org/rfc/rfc7714) or [ChaCha20Poly1305](https://www.rfc-editor.org/rfc/rfc7539) and that symmetric keys are chosen using [secure, private random number generators](https://datatracker.ietf.org/doc/html/rfc4086#section-7.2.2). Additionally, users should be careful to avoid [nonce reuse](https://cwe.mitre.org/data/definitions/323.html) in schemes that are vulnerable to it, and prefer schemes that are resistant to nonce reuse (e.g. [AES-GCM-SIV](https://en.wikipedia.org/wiki/AES-GCM-SIV)) to limit user error.

## Incentive considerations

N/A

## Product considerations

In principle, timelock encryption could be possible on Ethereum, but given the lack of [precompiles for BLS operations](https://eips.ethereum.org/EIPS/eip-2537), it's prohibitively expensive in practice, and we're unaware of anybody offering a contract to do it.
At [Real World Crypto 2023](https://rwc.iacr.org), dfinity presented a talk on threshold signatures that suggested it would be possible to implement timelock encryption in userspace of the Internet Computer. Again, while they do offer some BLS12-381 operations out-of-the-box, there is no precompile for timelock encryption specifically.
To that end, FVM could be the first blockchain ecosystem providing timelock encryption as a first-class functionality which could confer adoption benefits.


## Implementation

There are already multiple implementations of our timelock encryption scheme which combines timelock encryption of a key used to perform [AGE](https://github.com/FiloSottile/age) symmetric encryption. Most relevant to FVM (at this time), Thibault Meunier from Cloudflare has a [rust implementation](https://github.com/thibmeu/tlock-rs) based on an MVP developed by [Timofey](https://github.com/timoftime/tlock-rs) from Chainsafe. It is complete and functional, and has been developed in consultation with the drand team. Unlike the team's [typescript](https://github.com/drand/tlock-js) and [go](https://github.com/drand/tlock) implementations however, it has not been formally security audited.

## Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).

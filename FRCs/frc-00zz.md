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
    function encrypt(bytes message, uint64 roundNumber) returns (bytes)
    function decrypt(bytes message) returns (bytes, bool)
}
```

```rust
trait Timelock {
    fn encrypt(message: Vec<u8>, roundNumber: u64) -> Result<Vec<u8>>
    fn decrypt(message: Vec<u8>) -> Result<Vec<u8>>
}
```

The `encrypt` functions take raw message bytes and a round number which, along with the public key of the drand network, are passed into the function specified in [the timelock encryption paper](https://eprint.iacr.org/2023/189.pdf) released by the team. In short, the round number is hashed to a point on the G2 group of the BLS12-381 curve. That point is multiplied by a new point derived from the message and mapped onto the target group Gt, and the message is xor'd with the resulting point on Gt.
For larger messages originating outside the system, we recommend using a symmetric cipher such as [age](https://age-encryption.org/) to encrypt the message, and performing timelock encryption on just the symmetric key, in order to save gas fees, as the pairing operation is comparatively expensive.
The above interfaces assume that details such as the drand network chain hash and public key are transparent for the user - should the Filecoin network switch to an alternative drand network in future, users may need to interact with different actor addresses to timelock encrypt/decrypt for the respective drand network. We assume this is in line with standard practice.

## Test Cases

- plaintexts encrypted by the network should be decryptable once the relevant time has been reached
- ciphertexts generated outside the network should be decryptable once the relevant time has been reached
- plaintexts encrypted by the network should not be decryptable before the relevant time has been reached
- ciphertexts should be compatible with the existing implementations [tlock](https://github.com/drand/tlock) and [tlock-js](https://github.com/drand/tlock-js)
- encryption using the points at 0 and infinity should be disallowed
- users cannot encrypt to a time before introduction of the new scheme (i.e. when [FIP-0063](https://github.com/filecoin-project/FIPs/pull/652)) was accepted and deployed to the Filecoin network), as the signatures before this time will not conform with the scheme for timelock encryption.

## Backwards compatibility

There are no backwards compatibility considerations, as this feature has not been exposed on the FVM until now.

## Security considerations

Are there implications for the miners seeing data before they timelock encrypt it?
How do we manage hybrid encryption to ensure users don't do it wrong?

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

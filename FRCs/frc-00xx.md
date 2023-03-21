## Simple summary

Presently, drand randomness can be retrieved on the FVM via the `prevrandao` opcode. This comes with limitations that make it unsuitable for many applications requiring randomness on the FVM. 
In this FRC, we propose changes that would enable retrieving arbitrary drand randomness beacons from arbitrary drand networks, allowing users to have randomness that conforms to their security model and enables timelock encryption and decryption on-chain.

## Abstract
//TODO: detail something about the `getRandomness()` function available to native actors
Presently drand randomness can be retrieved inside smart contracts on the FVM via the `prevrandao` opcode. This opcode has been provided for EVM compatibility and evaluates to the drand random value from the previous tipset's header truncated to 256bits. This poses multiple concerns:

- all network participants know the randomness in advance of the next tipset being mined
- users are limited to the randomness from a single League of Entropy network
- drand emits 1024bits of entropy, much of which is lost

### all network participants know the randomness in advance of the next tipset being mined
For some applications, such as lotteries or games, knowing the randomness in advance of the next block can cause players to submit or not submit a transaction, as they can precompute the outcome given the current state of the chain.
Similarly, block producers can decide whether or not to include transactions in a block upon doing the same precomputation, if they have an incentive to do so.
To that end, smart contract developers should be able to access the drand randomness for the current block, to be filled in at execution time in order to ensure an unbiased outcome. Additionally, this lowers the time available for block producers to precompute the state of the world to gain an edge, lest they risk missing a block and getting slashed.

### users are limited to the randomness from a single League of Entropy network
The security assumptions of the League of Entropy network rely on trusting there were never a threshold number of bad actors in the League of Entropy (at any time since inception). While this is a high bar for most people, for a variety of technical, security, political or social reasons, it may not be an appropriate users deploying to FVM. Being open source software, anybody can run their own drand network consisting of participants who fulfil their trust assumptions. For example, interational government agencies might want to run cross-country drand networks, Cryptosat may want to run a satellite-only drand network, or other partners may want to run a TPM-backed drand network.

### drand emits 1024bits of entropy, much of which is lost
//TODO: add some use cases that might need this entropy
While 256bits is plenty of entropy for most use cases, drand provides more!

### suggested changes
//TODO: precompiles for BLS12-381 public keys, encryption with them and verification of sigs them
//TODO: precompiles for BLS12-381 pairing operation

### timelock encryption
Coupled with [FIP-xxx](link-to-network-switch-FIP), these changes would trivially enable timelock encryption and decryption on the Filecoin network - users could timelock encrypt messages by encrypting them with the public key of a supported network, and they could decrypt messages by either performing the pairing operations themselves or interacting with a native actor the drand team plan to implement (when the required functionality becomes available).

## Change motivation

## Specification

## Backwards compatibility

## Security considerations

## Incentive considerations

## Product considerations

## Implementation

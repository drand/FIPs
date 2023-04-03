## Simple Summary

Presently, drand randomness can be retrieved on the FVM via the `prevrandao` opcode. A limitation of this is that users and miners know this randomness in advance of the next block being mined, allowing users to submit or not submit transactions, and miners to include or not include transactions, based on precomputation of future actor state.
This FRC proposes an FVM function for retrieving the randomness for the current block to eliminate the drawbacks of pre-knowledge of the randomness in FVM actors.

## Abstract

Randomness is important on the FVM for a variety of uses cases such as lotteries, games and public selection processes.
The randomness included in the Filecoin block headers comes from [drand](https://drand.love), a threshold network providing public, verifiable, unbiasable and unpredictable random numbers at a rate (presently) consistent with Filecoin block creation, i.e. every 30 seconds. 
When applying transactions to a tipset to mine a block, miners pull the latest drand round from the drand network, verify it, and include it in the block header. Additionally, they use this randomness as an input to WinningPoSt and their election proof generation to show that they are eligible to mine the block in question.

At present, authors of actors on the FVM can only retrieve randomness from prior blocks using the `prevrandao` opcode. Due to blocks by their nature being public, this randomess is available to everybody following the Filecoin blockchain (henceforth 'viewers') 30 seconds ahead of time (i.e. the time period between blocks), enabling viewers of the network to precompute future contract state before the next block has been mined. 
For applications such as lotteries, viewers could probabilistically (or definitively) precompute the outcome or win conditions of a such a smart contract, and submit high priority transactions to achieve favourable outcomes.
Additionally, malicious miners could choose to include or not include transactions in the next block if they gain some benefit from doing so (commonly referred to as [MEV](https://coinmarketcap.com/alexandria/glossary/miner-extractable-value-mev)).

All current implementations of Filecoin surveyed retrieve randomness before mining the next block (and thus executing updates to FVM actors), and to that end we propose a new built-in function that provides actors with the randomness from the block currently being mined. 

## Change Motivation


## Specification

We propose the following function guideline specification (pulled in part from [ref-fvm](https://github.com/filecoin-project/ref-fvm)):
```rust
fn get_current_randomness() -> Result<[u8; 128]>
```

As existing implementations already include the current block's randomness in their execution process (e.g. [in Lotus](https://github.com/filecoin-project/lotus/blob/43da1084669fa1d91bd0b1bb22b4d459b477ba4b/chain/consensus/compute_state.go#L76)), we're open to minor changes that would be canonical for any given implementation.
The randomness length suggested is 128 bytes - the full amount drand makes available per beacon.

This could also be exposed to solidity developers as a precompiled contract matching the following:
```solidity
interface Drand {
    function current() returns(uint256);
    function round(uint64 roundNumber) returns(uint256);
}
```
Note the above functions return a `uint256`, in line with the `prevrandao` opcode specification, rather than the full 1024 bits of entropy provided by drand. This appears canonical, as Solidity doesn't provide an arbitrary precision integer type out of the box, however if there is an accepted standard for a `BigInt` type, the interface could be replaced or augmented to use it.

## Design Rationale


## Backward Compatibility

There are no backwards compatibility considerations, as this feature has not been exposed on the FVM until now.

## Test Cases

Test cases require validation of BLS12-381 signatures, which are already a primitive within the Filecoin ecosystem. 
Tests for compatibility of randomness before block height 50`000 must also be validated against the drand devnet public key, as a switch was made between networks at this time; these already exist in compliant implementations for validating old tipsets, but should be extended to cover fetching and validating randomness on the FVM.

## Security Considerations

## Incentive Considerations

## Product Considerations

## Implementation

A prior implementation of randomness on the FVM is provided (although disabled) in [ref-fvm](https://github.com/filecoin-project/ref-fvm). It has some limitations however, such as not being able to fetch arbitrary randomness rounds and only providing 32bits of randomness.

## Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).

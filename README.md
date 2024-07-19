# Open Value Sharing (OVS)

Version: 0.0.0

Open Value Sharing (OVS) is an open protocol for automatically sharing monetary value with software dependencies.

OVS enables consumers of software dependencies to automatically pay for their use while minimizing or eliminating the friction involved. Though not restricted to open source software dependencies, one main goal of OVS is to enable a new open source native business model, facilitating automatic revenue streams to open source projects without requiring changes to their licenses.

OVS embraces a number of core ideals:

1. Minimize or eliminate consumer (payer) friction
2. No changes required to existing dependency licenses
3. Voluntary participation from consumers and dependencies
4. Flexibility across platforms, payment mechanisms, sharing heuristics, and other parameters
5. Heuristics over perfection

## Definitions

### Consumer

A software project with software dependencies. The consumer will be using and making payments to its dependencies.

### Dependency

A software project that is used by a consumer. Dependencies are generally installed and incorporated into consumers using tools such as [npm](https://www.npmjs.com/), [PyPI](https://pypi.org/), or [crates.io](https://crates.io/).

### Platform

The infrastructure or environment that hosts or executes the consumers and their dependencies. Platforms generally foster ecosystems or communities such as [Linux](https://en.wikipedia.org/wiki/Linux), [Android](<https://en.wikipedia.org/wiki/Android_(operating_system)>), [Bitcoin](https://bitcoin.org/bitcoin.pdf), [Ethereum](https://ethereum.org/en/), [Solana](https://solana.com/), [ICP](https://internetcomputer.org/), [AO](https://ao.arweave.dev/), [AWS](https://aws.amazon.com/), [Azure](https://azure.microsoft.com/en-us), or [Vercel](https://vercel.com/).

### Asset

The currency or medium of exchange that a consumer will use to pay a dependency. Most likely to be a cryptocurrency, fiat currency, or similar medium of exchange. Examples are [USD](https://en.wikipedia.org/wiki/United_States_dollar), [EUR](https://en.wikipedia.org/wiki/Euro), [USDC](https://www.circle.com/en/usdc), [USDT](https://tether.to/en/), [BTC](https://en.wikipedia.org/wiki/Bitcoin), [ETH](https://en.wikipedia.org/wiki/Ethereum), or [cycles](https://internetcomputer.org/docs/current/concepts/tokens-cycles#cycles).

### Payment Mechanism

The actual mechanism used to facilitate the payment of an asset from a consumer to a dependency for a specific platform. Examples are [ACH](https://www.fiscal.treasury.gov/ach), [wire transfer](https://en.wikipedia.org/wiki/Wire_transfer), [ERC20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/), [SPL](https://spl.solana.com/token), or [cycles ledger](https://internetcomputer.org/docs/current/developer-docs/defi/cycles/cycles-ledger).

### Payment

The transfer of an asset from the consumer to one of its dependencies.

### Batch

A group of executed payments.

### Period

The interval of time between batches.

### Sharing Heuristic

A practical approach for determining and executing payments. For example, Burned Weighted Halving is the heuristic influencing this specification. It is explained below. Implementations may choose other heuristics which are not specified here.

### Burned Weighted Halving

This heuristic is designed for the [ICP](https://internetcomputer.org/) platform and the [cycles](https://internetcomputer.org/docs/current/concepts/tokens-cycles#cycles) asset. It greatly influences this specification, and its basic concepts are expected to influence the default heuristic for most platforms.

Burned Weighted Halving is a resource usage-based sharing heuristic. It attempts to measure resource usage at a high level rather than at the level of an individual dependency. That measurement combined with dependency weights and depths in the dependency tree are used to attempt to fairly compensate dependencies.

The total payment amount to be divided between dependencies during a batch is determined by the amount of cycles burned between periods multiplied by the shared percentage. This amount is cut in half and assigned to the first level of the dependency tree. All dependencies at the first level share this half of the total payment amount. This amount is divided between dependencies at the first level based on their weight. The weight defaults to 1 but can be overriden individually by the consumer.

The next level in the dependency tree shares half of the remaining half not assigned to the first level. This repeats until the final level. The final two levels receive the same amount to be shared between their dependencies.

### Shared Percentage

If utilizing a resource usage-based sharing heuristic, this is the percentage of resource usage between periods used to determine the total payment amount to be divided between dependencies in a batch. For example in Burned Weighted Halving, this percentage is applied to the total number of cycles burned between periods.

### Dependency Tree

The relationships between dependencies and their consumer can be modeled as a tree. The consumer is at the base of the tree. It will depend on a number of dependencies directly, which we will collectively call level or depth 0. The dependencies of those dependencies (transitive dependencies) create another level in the tree which we will call level 1. The dependencies of level 1 would create another level which we will call level 2. And the pattern repeats until there are no more dependencies.

### Depth

The level at which a dependency is located in the dependency tree.

### Weight

A number representing the proportion of the total payment assigned to a level for a batch that a dependency should receive relative to its siblings at the same level.

## Implementation

### Dependency Configuration

Dependencies must provide all of the necessary information to their consumer for proper payments to be made. Dependencies should define one or more of the following collections of properties in a configuration format and location that the implementation can automatically access:

```
type custom_value = variant {
    number(u32),
    text(string)
}

type custom_item = record {
    key: string,
    value: custom_value
}

type dependency = record {
    platform: string,
    asset: string,
    payment_mechanism: string,
    custom: list<custom_item>
}
```

For example, dependencies may elect to store the information specified above in the JSON format in a file called `.openvaluesharing.json`. This file could be in the root directory of the project, and if downloaded by the consumer (as is the case with `npm` packages), it would be automatically available for processing.

### Consumer Configuration

Consumers must provide all of the necessary information to the implementation for proper execution of the sharing heuristic. Implementations should utilize sensible defaults and not require consumers to define anything by default for the sharing heuristic to work. To do otherwise would violate ideal 1.

If necessary, a consumer configuration format and location should be specified to allow the consumer to override the following properties:

```
type weight = record {
    name: string,
    weight: u32
}

type consumer = record {
    kill_switch: bool,
    platforms: list<string>,
    assets: list<string>,
    shared_percentage: u32,
    period: u32,
    sharing_heuristic: string,
    weights: list<weight>
}
```

The `kill_switch` is required to comply with ideal 3. `platforms` and `assets` instruct the implementation to only honor those `platforms` and `assets` for the consumer. The `shared_percentage`, `period`, `sharing_heuristic`, and `weights` should have sensible defaults but can be overriden by the consumer.

### Batch Payment Processing

The implementation should use the consumer configuration and dependency configurations to implement batch payment processing on a period. The implementation should traverse the dependency tree and collect all dependency configurations. If the depth is important to the sharing heuristic then it should be obtained. Weights should have a sensible default and must be able to be overriden by the consumer. Every period payments should be made following the sharing heuristic. Implementations must honor the consumer and dependency configurations. Payments should be peer-to-peer and logging may be implemented.

### Warning

Ideal 1 is believed to be vital to the success of OVS. The consumer must not be made to do anything by default. The consumer should have power to override but should not be required to override. Anything you ask the payer to do you should consider an extreme risk of causing the consumer to turn off OVS. This is not so important for the dependency, as the funds received are believed to be motivation enough to go through some effort.

### Concerns

Various concerns have been brought up over time.

-   Consumers will fork libraries so that they don't have to pay
-   Consumers will turn OVS off

## License

MIT License

Copyright (c) 2024 Demergent Labs LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

TODO should we explain logging etc? Should we explain what the implementation should do about logging, about providing stats etc?

TODO do we need to explain supply/demand, prices, common concerns?

TODO do we need to explain how implementations vs spec works?

TODO should we have a prior art section?

TODO should we link to the Azle implementation?

TODO should we explain that we're using WIT?

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

Dependencies must define their `name`, `platforms`, `assets`, and `payment mechanisms`. Dependencies may define more than one `platform`, `asset`, or `payment mechanism`. Implementations can allow the dependency to define this information in a variety of ways, including JSON, YAML, etc file.

```wit
TODO put the dependency config info here
```

Generally an implementation should expect dependencies to define their dependency configuration in a manner that is easily accessible during a build process. For example, the dependency config could be included inside of the dependency assets required to be on the consumer's machine. During build the implementation would find all dependencies, assigning depths, weights, and extracting the `platforms`, `assets`, and `payment mechanisms` accepted by each dependency.

The implementation should use the consumer configuration to override any weights for individual dependencies. The consumer should also be able to turn OVS on or off, and should be able to specify which platforms and assets it is willing to use. It should also be able to set the period, shared percentage, and sharing heuristic.

The implementation should use the dependency and consumer configurations to initiate a periodic batch payment process. On each period payments should be made to each dependency using the platform, asset, payment mechanism, and custom information.

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

# Notes (everything below is just scratch space)

## Specification

This specification attempts to define the terms, data types, and processes common across all OVS implementations. Implementations will mostly be involved in choosing file formats and locations, languages, APIs, etc.

This specification uses [WIT](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md) to define types.

OVS mainly deals with the relationship between a `consumer` and its `dependencies`.

### Consumer

```wit
type consumer = {
    killSwitch: bool,
    platforms: list<string>,
    assets: list<string>,
    shared_percentage: u32,
    period: u32,
    sharing_heuristic: string,
    dependencies: list<dependency>
}
```

### Dependency

```wit
type custom_value = variant {
    number(u32),
    text(string),
    record(list<custom_item>)
}

type custom_item = {
    key: string,
    value: custom_value
}

type dependency = {
    name: string,
    depth: u32,
    weight: u32,
    platform: string,
    asset: string,
    payment_mechanism: string,
    custom: list<custom_entry>
}
```

Consumers and dependencies should be allowed full control over specifying as many properties as possible. Implementations should allow consumers to override dependency weights. Depth should be calculated by the consumer's implementation, and should not be specified by the dependency. The dependency should specify: name, accepted platforms, accepted asset, accepted payment mechanism, and any custom fields.

The consumer should decide its own default weight for dependencies, should allow dependency weights to be overriden, and should calculate depths. Consumers should never use a platform, asset, or payment mechanism not specified by the dependency.

Consumers should be able to provide configurations to their implementation. Dependencies as well.

A consumer config might be located in a JSON or other configuration file, same as the dependency config. Implementations are free to choose their own means of specifying the consumer and dependency configurations.

A consumer config might look like this:

```
type consumer_config = {

}
```

A dependency config might look like this:

```
type dependency_config = {

}
```

TODO should we explain logging etc? Should we explain what the implementation should do about logging, about providing stats etc?

TODO do we need to explain supply/demand, prices, common concerns?

TODO do we need to explain how implementations vs spec works?

TODO should we have a prior art section?

TODO should we link to the Azle implementation?

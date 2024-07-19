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

The implementation is executed by the consumer. Dependencies must define a configuration available to the implementation. The implementation processes the dependency tree, gathering all dependency configuration information. This information is combined with sensible defaults or a consumer configuration to produce a final list of dependencies with dependency tree depth and weight information. This information is used to initiate a periodic payment process. The consumer must not be required to do anything for a default implementation to work. Dependencies must define configurations.

This specification uses [WIT](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md) for type definitions.

### Dependency Configuration

Dependencies must provide all of the necessary information to their consumer's implementation for proper payments to be made. Dependencies should define one or more of the following collections of properties in a configuration format and location that the consumer's implementation can automatically access. For example, dependencies may elect to store the information specified above in the JSON format in a file called `.openvaluesharing.json`. This file could be in the root directory of the project, and if downloaded by the consumer (as is the case with `npm` packages), it would be automatically available for processing.

Dependency configuration data structures:

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

### Consumer Configuration

Implementations must utilize sensible defaults and not require consumers to do anything for a default implementation to work. To do otherwise would violate ideal 1. Consumers must be able to override the sensible defaults.

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
    shared_percentage: f32,
    period: u32,
    sharing_heuristic: string,
    weights: list<weight>
}
```

The `kill_switch` turns OVS on or off and is required to comply with ideal 3. Most of the remaining properties should be self-explanatory. `weights` allows the consumer to control payment amounts for individual dependencies, including turning payments off with a weight of 0.

### Periodic Payment Processing

The implementation must use the consumer configuration and dependency configurations to implement periodic payment processing. Every period payments should be made following the sharing heuristic. Payments should be peer-to-peer and logging/statistics may be implemented by consumers and dependencies.

## ICP Implementation

The first MVP implementation of OVS is live on [ICP](https://internetcomputer.org/) in the [Azle project](https://github.com/demergent-labs/azle). Demergent Labs, the author of the initial draft of this specification, also authored the Azle project and its OVS implementation.

-   [Azle's implementation of consumer and dependency config processing](https://github.com/demergent-labs/azle/blob/main/src/compiler/get_consumer_config.ts)
-   [Azle's implementation of periodic payment processing](https://github.com/demergent-labs/azle/tree/main/src/compiler/rust/open_value_sharing)

## Economic Concerns

Some have brought up the concern that consumers will decide to turn off OVS. Others fear that consumers will fork dependencies with OVS enabled.

To the first concern, ideal 3 does allow for consumers to choose to turn off OVS. The rebuttal is that there is a cost to turning off OVS, and it may be higher than the cost of leaving it on. Leaving it on costs money. Turning it off costs mental effort, team time, and the conscious decision to not give back to open source. There could also be public pressures that arise incentivizing companies to continue to give back, especially if it was the default setting and a conscious decision to not give back.

To the second concern, the rebuttal is that the consumer configuration allows consumers to turn off payments entirely or for individual dependencies. It seems illogical then for a consumer to go through the effort to fork projects to get around OVS when it is much simpler and easier to just turn it off.

## Prior Art

-   [npm Supporting Open Source Maintainers](https://blog.npmjs.org/post/187382017885/supporting-open-source-maintainers.html)
-   [Brave Rewards](https://brave.com/brave-rewards/)
-   [Sustainus](https://github.com/lastmjs/sustainus)
-   [Podcrypt](https://medium.com/hackernoon/podcrypt-automatic-fair-peer-to-peer-podcast-donations-with-ether-f0a638111410)

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

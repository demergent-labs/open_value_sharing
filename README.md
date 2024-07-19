# Open Value Sharing (OVS)

Version: 0.0.0

Open Value Sharing (OVS) is an open protocol for automatically sharing monetary value with software dependencies.

OVS enables consumers of software dependencies to automatically pay for their use while minimizing or eliminating the friction involved. Though not restricted to open source software dependencies, one main goal of OVS is to enable a new open source native business model, facilitating automatic revenue streams to open source projects without requiring changes to their licenses.

OVS embraces a number of core ideals:

1. Minimize or eliminate consumer (payer) friction
2. No changes required to existing dependency licenses
3. Voluntary participation from consumers and dependencies
4. Flexibility across platforms, payment mechanisms, sharing heuristics, and other parameters

## Definitions

### Consumer

Generally a software project that has software dependencies. The consumer is the entity that will be paying dependencies.

### Dependency

A software project that is used by a consumer. Dependencies can be installed and incorporated with a variety of mechanisms i.e. npm, PiPy, crates.io, etc.

### Platform

A broad ecosystem or project that hosts the consumer and dependencies. A platform would be something like ICP, Bitcoin, Ethereum, Solana, AWS, Azure, etc.

### Asset

A monetary instrument such as USD, Euro, Mexican Pesos, USDC, USDT, BTC, ETH, cycles, etc

### Shared Percentage

The percentage of the total resources consumed over a period of time used as the amount to be distributed to all dependencies during a batch.

### Period

The interval of time on which batches of payments will be made.

### Sharing Heuristic

The heuristic used to determine the amounts for each payment in a batch. For example, Burned Weighted Halving.

### Depth

The depth in the dependency tree that a dependency is at.

### Weight

The weight given to a dependency, generally to be used to determine the payment amount at the same level in the tree. Under the most common heuristic.

### Payment Mechanism

Once a platform and asset is selected, the payment mechanism is the rails to use. For example, ACH, wire transfer, ERC20, deposit_cycles, cycles ledger, credit card, etc.

### Payment

An actual monetary payout to a dependency, usually occuring within a batch.

### Batch

A group of payments performed on the period.

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

### Explain each concept

-   platforms
-   assets
-   payment mechanism
-   sharedPercentage
-   period
-   sharingHeuristic
-   depth
-   weight
-   periodic batch
-   payment

TODO should we explain logging etc? Should we explain what the implementation should do about logging, about providing stats etc?

### Payment Heuristic

At the end of each period, a Periodic Payout Amount is calculated as follows:

Determine the total Shared Value Source. Multiply that by the Shared Value Source Percentage. This is the PPA.

Take the PPA and multiply it be .5. This is the PPA for the first level. Divide the PPA by the number of dependencies in that level and pay out that amount to each dependency.

The new PPA is the amount leftover after multiplying the original PPA by .5.

#### Shared Value Source

The amount of cycles burned within a period.

#### Shared Value Source Percentage

The percentage of the Shared Value Source that will be paid out at the end of a period.

#### Period

The length of time between payouts.

#### Periodic Payout Amount (PPA)

The amount to be paid out at the end of a period.

TODO do we need to explain supply/demand, prices, common concerns?

TODO do we need to explain how implementations vs spec works?

TODO should we have a prior art section?

TODO should we link to the Azle implementation?

# Open Value Sharing (OVS)

Version: 0.0.0

Open Value Sharing (OVS) is an open protocol for automatically sharing monetary value with software dependencies.

OVS enables consumers of software dependencies to automatically pay for their use while minimizing or eliminating the friction involved. Though not restricted to open source software dependencies, one main goal of OVS is to enable a new open source native business model, facilitating automatic revenue streams to open source projects without requiring changes to their licenses.

OVS embraces a number of core ideals:

1. Minimize or eliminate consumer (payer) friction
2. No changes required to existing dependency licenses
3. Voluntary participation from consumers and dependencies
4. Flexibility across platforms, payment mechanisms, sharing heuristics, and other parameters

## Specification

### Consumer

Consumer config, consumer data structure

### Dependency

Dependency config, dependency data structure

### Dependency Information Format

Dependency information should be provided to the payment mechanism in a JSON file with the following structure:

```typescript
type DependencyInfo = DependencyLevel[];

type DependencyLevel = Dependency[];

type Dependency = {
    name: string;
    platform: string;
    asset: string;
    payment_mechanism: string;
    custom: Record<string, any>;
};
```

As an example:

```json
[
    [
        {
            "name": "azle",
            "weight": 1,
            "platform": "icp",
            "asset": "cycles",
            "payment_mechanism": "wallet",
            "custom": {
                "principal": "qntqt-jvgzo-jb6ii-d4ziy-sfch5-xezjz-p43eq-ojrhx-zch3w-xwist-aqe"
            }
        }
    ]
]
```

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

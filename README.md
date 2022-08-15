# Trade Aggregation
A high performance, modular and flexible trade aggregation crate, producing Candle data, 
suitable for low-latency applications and incremental updates.
It allows the user to choose the rule dictating how a new candle is created 
through the [AggregationRule](src/aggregation_rules/aggregation_rule_trait.rs) trait, 
e.g: Time, Volume based or some other information driven rule.
It also allows the user to choose which type of candle will be created from the aggregation process
through the [ModularCandle](src/modular_candle_trait.rs) trait. Combined with the [Candle](trade_aggregation_derive/src/lib.rs) macro, 
it enables the user to flexibly create any type of Candle as long as each component implements 
the [CandleComponent](src/candle_components/candle_component_trait.rs) trait.

See [MathisWellmann/go_trade_aggregation](https://github.com/MathisWellmann/go_trade_aggregation) for a go implementation with less features and performance.

## Features:
### AggregationRule:
The pre-existing rules in this crate include:
'AggregationRule' | Description
------------------|-------------
TimeRule          | Create candles every n seconds
VolumeRule      | Create candles every n units trades

If these don't satisfy your desires, just create your own by implementing the [AggregationRule](src/aggregation_rules/aggregation_rule_trait.rs) trait,
and you can plug and play it into the [GenericAggregator](src/aggregator.rs).

### CandleComponent:
These pre-existing 'CandleComponents' exist out of the box:
'CandleComponent' | Description
------------------| ---
Open              | Price at the beginning of a candle
High              | Maximum price during the candle
Low               | Minimum price during the candle
Close             | Price at the end of a candle
Volume            | The cumulative trading volume
NumTrades         | The number of trades during the candle
AveragePrice      | The equally weighted average price
WeightedPrice     | The volume weighted price
StdDevPrices      | Keeps track of the standard deviation of prices
StdDevSizes       | Keeps track of the standard deviaton of sizes
TimeVelocity      | Essentially how fast the candle was created time wise

And again, if these don't satisfy your needs, just bring your own by implementing the 
[CandleComponent](src/candle_components/candle_component_trait.rs) trait and you can plug them into your own candle struct.

## How to use:
To use this crate in your project, add the following to your Cargo.toml:

```toml
[dependencies]
trade_aggregation = "^3"
```

Lets aggregate all trades into time based 1 minute candles, consisting of open, high, low and close information.
Notice how easy it is to specify the 'AggregationRule' and 'ModularCandle' being used in the process.
One can easily plug in their own implementation of those trait for full customization.

```rust
use trade_aggregation::{
    candle_components::{Close, High, Low, Open},
    *,
};

#[derive(Debug, Default, Clone, Candle)]
struct MyCandle {
    open: Open,
    high: High,
    low: Low,
    close: Close,
}

fn main() {
    let trades = load_trades_from_csv("data/Bitmex_XBTUSD_1M.csv")
        .expect("Could not load trades from file!");

    // specify the aggregation rule to be time based
    let time_rule = TimeRule::new(M1);
    let mut aggregator = GenericAggregator::<MyCandle, TimeRule>::new(time_rule);

    let candles = aggregate_all_trades(&trades, &mut aggregator);
    println!("got {} candles", candles.len());
}
```

Alternatively, use an online updating approach to update with each tick:

```rust
use trade_aggregation::{
    candle_components::{Close, High, Low, Open},
    *,
};

#[derive(Debug, Default, Clone, Candle)]
struct MyCandle {
    open: Open,
    high: High,
    low: Low,
    close: Close,
}

fn main() {
    let trades = load_trades_from_csv("data/Bitmex_XBTUSD_1M.csv")
        .expect("Could not load trades from file!");

    // specify the aggregation rule to be time based
    let time_rule = TimeRule::new(M1);
    let mut aggregator = GenericAggregator::<MyCandle, TimeRule>::new(time_rule);

    for t in &trades {
        if let Some(candle) = aggregator.update(t) {
            println!(
                "candle created with open: {}, high: {}, low: {}, close: {}",
                candle.open(),
                candle.high(),
                candle.low(),
                candle.close()
            );
        }
    }
}
```

Notice how the code is calling the 'open()', 'high()', 'low()' and 'close()' 
methods on the 'MyCandle' struct. These are automatically generated getters that have the same name as the field.

See examples folder for more.
Run examples using
```
cargo run --release --example aggregate_all_ohlc
cargo run --release --example streaming_aggregate_ohlc
```

## Performance:
To run the benchmarks, written using criterion, run:

```shell
cargo bench
```

Here are some results running on a 12th gen Intel Core i7-12800H, aggregating 1 million trades into 1 minute candles:

Candle | Time
-------|-----------
Open   |  1.8 ms
OHLC   |  7   ms
All    | 16   ms

The more 'CandleComponent's you use, the longer it takes obviously.

### Donations :moneybag: :money_with_wings:
I you would like to support the development of this crate, feel free to send over a donation:

Monero (XMR) address:
```plain
47xMvxNKsCKMt2owkDuN1Bci2KMiqGrAFCQFSLijWLs49ua67222Wu3LZryyopDVPYgYmAnYkSZSz9ZW2buaDwdyKTWGwwb
```

![monero](img/monero_donations_qrcode.png)


### License
Copyright (C) 2020  <Mathis Wellmann wellmannmathis@gmail.com>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.

![GNU AGPLv3](img/agplv3.png)

### Commercial License
If you'd like to use this crate legally without the restrictions of the GNU AGPLv3 license, 
please contact me so we can quickly arrange a custom license.

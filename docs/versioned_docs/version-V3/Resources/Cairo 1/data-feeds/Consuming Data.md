---
id: consuming-data
title: Consuming Data
sidebar_position: 1
---

---

You can find the list of supported assets [here](./Supported%20Assets.md).

The current Pragma addresses are:

| Network               | Address                                                            | Explorer                                                                                                                                                                                                                                                |
| --------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| StarkNet Mainnet      | 0x2a85bd616f912537c50a49a4076db02c00b29b2cdc8a197ce92ed1837fa875b  | [Starkscan](https://starkscan.co/contract/0x2a85bd616f912537c50a49a4076db02c00b29b2cdc8a197ce92ed1837fa875b) [Voyager](https://voyager.online/contract/0x2a85bd616f912537c50a49a4076db02c00b29b2cdc8a197ce92ed1837fa875b)                               |
| StarkNet Alpha-Goerli | 0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167 | [Starkscan](https://testnet.starkscan.co/contract/0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167) [Voyager](https://goerli.voyager.online/contract/0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167#transactions) |
| StarkNet Sepolia | 0x5acaa0ef3a6837c98ddeae516fa3f6022018cf646914d921842e46fc00c35d3 | [Starkscan](https://sepolia.starkscan.co/contract/0x5acaa0ef3a6837c98ddeae516fa3f6022018cf646914d921842e46fc00c35d3) [Voyager](https://sepolia.voyager.online/contract/0x5acaa0ef3a6837c98ddeae516fa3f6022018cf646914d921842e46fc00c35d3#transactions) |

## Sample Code

If you are just trying to get started with our price feeds, see the self-contained code snippets below. If you'd like to use more advanced oracle functions please see the further information below. You can find a full sample data feed consumer contract [here](https://github.com/Astraly-Labs/pragma-hack/tree/master) and the full Oracle interface specification is available [here](https://github.com/Astraly-Labs/pragma-oracle/blob/main/src/oracle/oracle.cairo).

## (Optional) Add Pragma as a dependency to your scarb/snforge project

```shell
scarb add pragma_lib --git https://github.com/astraly-labs/pragma-lib
```

### BTC/USD Spot Median Price

```rust

use pragma_lib::abi::{IOracleABIDispatcher, IOracleABIDispatcherTrait};
use pragma_lib::types::{AggregationMode, DataType, PragmaPricesResponse};
use starknet::ContractAddress;
use starknet::contract_address::contract_address_const;


const KEY :felt252 = 18669995996566340; // felt252 conversion of "BTC/USD", can also write const KEY : felt252 = 'BTC/USD';

fn get_asset_price_median(oracle_address: ContractAddress, asset : DataType) -> u128  { 
    let oracle_dispatcher = IOracleABIDispatcher{contract_address : oracle_address};
    let output : PragmaPricesResponse= oracle_dispatcher.get_data(asset, AggregationMode::Median(()));
    return output.price;
}

//USAGE

let oracle_address : ContractAddress = contract_address_const::<0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167>();
let price = get_asset_price_median(oracle_address, DataType::SpotEntry(KEY));
```

#### SOL/USD Spot Average Price, filtered by sources

```rust

use pragma_lib::abi::{IOracleABIDispatcher, IOracleABIDispatcherTrait};
use pragma_lib::types::{AggregationMode, DataType, PragmaPricesResponse};
use starknet::ContractAddress;
use starknet::contract_address::contract_address_const;
use array::ArrayTrait;

const KEY: felt252 = 23449611697214276; // felt252 conversion of "SOL/USD", can also write const KEY : felt252 = 'SOL/USD'
const OKX: felt252 = 'OKX'; // felt252 conversion of "OKX"
const BINANCE: felt252 = 'BINANCE'; // felt252 conversion of "BINANCE"

fn get_asset_price_average(oracle_address: ContractAddress, asset : DataType, sources : Span<felt252>) -> u128  { 
    let oracle_dispatcher = IOracleABIDispatcher{contract_address : oracle_address};
    let output : PragmaPricesResponse= oracle_dispatcher.get_data_for_sources(asset, AggregationMode::Mean(()), sources);

    return output.price;
}

//USAGE

let oracle_address : ContractAddress = contract_address_const::<0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167>();

let mut sources = ArrayTrait::<felt252>::new();
sources.append(OKX);
sources.append(BINANCE);

let price = get_asset_price_average(oracle_address, DataType::SpotEntry(KEY), sources.span());
```


#### BTC/USD Future Price

```rust
use pragma_lib::abi::{IOracleABIDispatcher, IOracleABIDispatcherTrait};
use pragma_lib::types::{AggregationMode, DataType, PragmaPricesResponse};
use starknet::ContractAddress;
use starknet::contract_address::contract_address_const;

const KEY :felt252 = 18669995996566340; // felt252 conversion of "BTC/USD", can write const KEY : felt252 = 'BTC/USD'

fn get_asset_price_median(oracle_address: ContractAddress, asset : DataType) -> u128  { 
    let oracle_dispatcher = IOracleABIDispatcher{contract_address : oracle_address};
    let output : PragmaPricesResponse= oracle_dispatcher.get_data(asset, AggregationMode::Median(()));

    return output.price;
}

//USAGE
let oracle_address : ContractAddress = contract_address_const::<0x06df335982dddce41008e4c03f2546fa27276567b5274c7d0c1262f3c2b5d167>();
let expiration_timestamp = 1691395615; //in seconds
let price = get_asset_price_median(oracle_address, DataType::FutureEntry((KEY, expiration_timestamp)));
```

## Technical Specification

### Function: `get_data_median`

This is the the simplest function that will aggregate all data into a median for a given data type.

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).

#### Returns

This function returns a struct, PragmaPricesResponse, which contains the following fields:
- `price`: aggregation result of all entries for the given key based on the robust median algorithm. Multiplied by `10**decimals`
- `decimals`: number of places that value has been shifted to allow for greater accuracy
- `last_updated_timestamp`: timestamp of the most recent entry aggregated
- `num_sources_aggregated`: number of sources aggregated in the final answer. Use this to check if one of the sources you requested was not available, or if there are enough data reports for you to rely on the answer
- `expiration_timestamp` : timestamp of when the data will expire. Works only for futures. 


### Function : `get_data_median_for_sources`

This is the simplest function that will aggregate all data for given sources into a median for a given data type. 

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `sources`: array of sources to aggregate. Requires a Span of felt252.

#### Returns 

This function returns a struct, PragmaPricesResponse, which contains the following fields:
- `price`: aggregation result of all entries for the given key based on the robust median algorithm. Multiplied by `10**decimals`
- `decimals`: number of places that value has been shifted to allow for greater accuracy
- `last_updated_timestamp`: timestamp of the most recent entry aggregated
- `num_sources_aggregated`: number of sources aggregated in the final answer. Use this to check if one of the sources you requested was not available, or if there are enough data reports for you to rely on the answer
- `expiration_timestamp` : timestamp of when the data will expire. Works only for futures. 



#### Function: `get_data`

Similar to get_data_median except it allows for an additional parameter to specify a custom aggregated logic (median, mean and more).

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use the structure AggregationMode defined in Pragma. Option must currently be set to `MEDIAN` or `MEAN`, . Additional options `VWAP`, `EXPONENTIAL_DECAY` are coming soon.

#### Returns

- `price`: aggregation result of all entries for the given key based on the robust median algorithm. Multiplied by `10**decimals`
- `decimals`: number of places that value has been shifted to allow for greater accuracy (fixed point)
- `last_updated_timestamp`: timestamp of the most recent entry aggregated
- `num_sources_aggregated`: number of sources aggregated in the final answer. Use this to check if one of the sources you requested was not available, or if there are enough data reports for you to rely on the answer
- `expiration_timestamp` : timestamp of when the data will expire. Works only for futures. 


### Function: `get_data_with_USD_hop`

This function enables you to rebase the price, i.e. use a different base currency. For instance, if you want the price of BTC/ETH, you can combine the BTC/USD and ETH/USD price data to derive that.

#### Inputs

- `base_currency_id`: felt252 for the base currency (e.g. BTC)
- `quote_currency_id`: felt252 for the base currency (e.g. ETH)
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use constants defined in Pragma. Option must currently be set to `MEDIAN`, `MEAN`. Additional options `TWAP`, `EXPONENTIAL_DECAY` are coming soon.
- `typeof` : SimpleDataType, represents an enum of the data type you are requesting. No pair_id /expiration timestamp required on the enum. 
- `expiration_timestamp`: expiration timestamp of the data you are requesting. Only required for futures.

#### Returns

- `price`: aggregation result of all entries for the given key (using the algorithm specified by the `aggregation_mode` parameter). Multiplied by `10**decimals`
- `decimals`: number of places that value has been shifted to allow for greater accuracy (fixed point)
- `last_updated_timestamp`: timestamp of the most recent entry aggregated
- `num_sources_aggregated`: number of sources aggregated in the final answer. Use this to check if one of the sources you requested was not available, or if there are enough data reports for you to rely on the answer
- `expiration_timestamp` : timestamp of when the data will expire. Works only for futures. 


### Function: `get_data_for_sources`

This function enables you to get the price of one currency in terms of another, by specifying the path from one the quote to the base currency. For instance, if you want the price of ETH/EUR, you can combine the ETH/USD, BTC/USD and BTC/EUR price data to derive that.

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use constants defined in Pragma. Option must currently be set to `MEDIAN`. Additional options `TWAP`, `EXPONENTIAL_DECAY` and `MEAN` are coming soon.
- `sources`: array of sources to aggregate. Requires a Span of felt252.

#### Returns

- `price`: aggregation result of all entries for the given key (using the algorithm specified by the `aggregation_mode` parameter). Multiplied by `10\*\*decimals`
- `decimals`: number of places that value has been shifted to allow for greater accuracy (fixed point)
- `last_updated_timestamp`: timestamp of the most recent entry aggregated
- `num_sources_aggregated`: number of sources aggregated in the final answer. Use this to check if one of the sources you requested was not available, or if there are enough data reports for you to rely on the answer
- `expiration_timestamp` : timestamp of when the data will expire. Works only for futures.


### Function: `get_data_entry`

This function enables you to get the most recent raw data point for a specific spot asset, source and publisher.

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `source`: uppercased utf8-encoded data source, e.g. `str_to_felt("GEMINI")=78362974965321`
- `publisher`: the publisher to be considered , e.g. `str_to_felt("PRAGMA")=88314212732225`

#### Returns

- `possible_entry`: enum of PossibleEntry ( among SpotEntry, FutureEntry, and later OptionEntry), i.e. a `struct` with members `base_entry`, which in turn has a `timestamp`, `source` and `publisher`, and additional members `data_type`, `price` and `volume`.

### Function: `get_data_entries`

This function enables you to get the multiple raw data points for a specific spot asset and all sources.

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).

#### Returns

- an Array of `possible_entry`: enum of PossibleEntry ( among SpotEntry, FutureEntry, and later OptionEntry), i.e. a `struct` with members `base_entry`, which in turn has a `timestamp`, `source` and `publisher`, and additional members `data_type`, `price` and `volume`.

### Function: `get_data_entries_for_sources`

This function enables you to get the most recent raw data point for a specific spot asset and a list of sources.

#### Inputs

- `data_type`: enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `sources`: array of sources to aggregate. Requires a Span of felt252.

#### Returns

- an Array of `possible_entry`: enum of PossibleEntry ( among SpotEntry, FutureEntry, and later OptionEntry), i.e. a `struct` with members `base_entry`, which in turn has a `timestamp`, `source` and `publisher`, and additional members `data_type`, `price` and `volume`.
- `timestamp` : timestamp of the most recent entry aggregated.

### Function `get_last_checkpoint_before`

This function returns the last checkpoint, i.e. the last snapshot of the oracle price saved onchain, before a given timestamp.

#### Inputs

- `data_type` : enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `timestamp` : The timestamp for which we take the checkpoint prior to it.
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use the structure AggregationMode defined in Pragma. Option must currently be set to `MEDIAN` or `MEAN`, . Additional options `VWAP`, `EXPONENTIAL_DECAY` are coming soon.

#### Returns

- `checkpoint` : a structure Checkpoint i.e. a `struct` with members `timestamp`, the timestamp of the checkpoint, `value`, the aggregated price at that checkpoint (according to the `aggregation_mode`), the `aggregation_mode`, the mode to use for combining the data sources available, `num_sources_aggregated`, the number of sources aggregated for that checkpoint. 
- `idx` : u64, the index of the checkpoint returned. 

### Function `get_latest_checkpoint_index`

This function returns the index of the latest checkpoint, i.e. the last snapshot of the oracle price saved onchain.

#### Inputs

- `data_type` : enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use the structure AggregationMode defined in Pragma. Option must currently be set to `MEDIAN` or `MEAN`, . Additional options `VWAP`, `EXPONENTIAL_DECAY` are coming soon.

#### Returns

- `idx` : u64, the index of the latest checkpoint.
- `is_valid` : bool, whether the latest checkpoint is valid or not (in case the idx returns 0, in order to make a distinction between the case where there is no checkpoint and the case where the latest checkpoint is 0 - cf one checkpoint).

### Function `get_latest_checkpoint`

This function returns the latest checkpoint, i.e. the last snapshot of the oracle price saved onchain.

#### Inputs

- `data_type` : enum of the data type you are requesting (See DataType structure). By providing the enum data type, you also provide the pair id (for spot entries), or the pair id and the expiration timestamp (for futures).
- `aggregation_mode`: aggregation mode to use for combining the many data sources available in Pragma. Use the structure AggregationMode defined in Pragma. Option must currently be set to `MEDIAN` or `MEAN`, . Additional options `VWAP`, `EXPONENTIAL_DECAY` are coming soon.

#### Returns

- `checkpoint` : a structure Checkpoint i.e. a `struct` with members `timestamp`, the timestamp of the checkpoint, `value`, the aggregated price at that checkpoint (according to the `aggregation_mode`), the `aggregation_mode`, the mode to use for combining the data sources available, `num_sources_aggregated`, the number of sources aggregated for that checkpoint. 
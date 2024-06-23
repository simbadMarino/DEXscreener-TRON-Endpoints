
<img src="https://cryptologos.cc/logos/tron-trx-logo.png" width="100" height="100" />


# TRON <> DEX Screener Adapter

v0.1 / June 2024

The DEX Screener Adapter is a set of HTTP endpoints that allows DEX Screener to track historical and real-time data for any Partner Decentralized Exchange. Adapters are responsible for supplying accurate and up-to-date data, whereas DEX Screener handles all data ingestion, processing and serving.

## Overview

- The DEX Screener Indexer queries Adapter endpoints to continuously index events as they become available, in chunks of one or many blocks at a time. Each block is only queried once, so caution must be taken to ensure that all returned data is accurate at the time of indexing.
- Adapters are to be deployed, served and maintained by the Partner
- If adapter endpoints become unreachable indexing will halt and automatically resume once they become available again
- The Indexer allows for customizable rate limits and block chunk size to ensure Adapter endpoints are not overloaded

## Endpoints

- An in-depth explanation of the schemas expected for each endpoint is described on the `Schemas` section below
- Numbers for amounts and price can be both `number` and `string`. Strings are more suitable when dealing with extremely small or extremely large numbers that can't be accurately serialized into JSON numbers.

- Indexing will halt if schemas are invalid or contain unexpected values (i.e.: `swapEvent.priceNative=0` or `pair.name=""`)

### Latest Block

**Request:** GET `/v1/blocks/latest/events`

**Response Schema:**

```tsx

interface LatestBlockResponse {
  data: Array<{
    block_number: number
    block_timestamp: number
    caller_contract_address: string
    contract_address: string
    event_index: number
    event_name: string
    result: {
      "0": string
      "1": string
      "2": string
      from: string
      to: string
      value: string
    }
    result_type: {
      from: string
      to: string
      value: string
    }
    event: string
    transaction_id: string
  }>
}
```

**Example Response:**

```json
{
  "data": [
    {
      "block_number": 45183011,
      "block_timestamp": 1719104409000,
      "caller_contract_address": "TBVLsD6kzqeLbnPTfby5NJhWTUDw5bosek",
      "contract_address": "TBVLsD6kzqeLbnPTfby5NJhWTUDw5bosek",
      "event_index": 0,
      "event_name": "Transfer",
      "result": {
        "0": "0x20366a2b5e52cd7980936953f42035796a8e93ad",
        "1": "0xcbb1388f7031f85fabe05f07bcf6a116f146fb89",
        "2": "10624972000",
        "from": "0x20366a2b5e52cd7980936953f42035796a8e93ad",
        "to": "0xcbb1388f7031f85fabe05f07bcf6a116f146fb89",
        "value": "10624972000"
      },
      "result_type": {
        "from": "address",
        "to": "address",
        "value": "uint256"
      },
      "event": "Transfer(address indexed from, address indexed to, uint256 value)",
      "transaction_id": "117bf5228aa911c654d3dd353483a3efdb03ba40c70cbefc92c455266bc35e29"
    }
  ],
  "success": true,
  "meta": {
    "at": 1719105850589,
    "page_size": 1
  }
}
```

**Notes:**

- The `/latest-block` endpoint should be in sync with the `/events` endpoint, meaning it should only return the latest block where data from `/events` will be available. This doesn't mean it should return the latest block with an event, but it should not return a block for which `/events` has no data available yet.
    - If `/events` fetches data on-demand this isn't an issue, but if it relies on data indexed and persisted in the backend then `/latest-block` should be aware of the latest persisted block
    - During live indexing, the Indexer will continuously poll `/latest-block` and use its data to then query `/events`

### Asset

**Request:** GET `/asset?id=:string`

**Response Schema:**

```tsx
interface AssetResponse {
  asset: Asset;
}
```

**Example Response:**

```json
{
  "asset": {
    "id": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
    "name": "Wrapped Ether",
    "symbol": "WETH",
    "totalSupply": 10000000,
    "circulatingSupply": 900000,
    "coinGeckoId": "ether",
    "coinMarketCapId": "ether"
  }
}
```

### Pair

**Request:** GET `/pair?id=:string`

**Response Schema:**

```tsx
interface PairResponse {
  pair: Pair;
}
```

**Example response:**

```json
{
  "pair": {
    "id": "0x11b815efB8f581194ae79006d24E0d814B7697F6",
		"dexKey": "uniswap",
    "asset0Id": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
    "asset1Id": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
		"createdAtBlockNumber": 100,
		"createdAtBlockTimestamp": 1698126147,
		"createdAtTxnId": "0xe9e91f1ee4b56c0df2e9f06c2b8c27c6076195a88a7b8537ba8313d80e6f124e",
    "feeBps": 100
  }
}
```

### Events

**Request:** GET `/events?fromBlock=:number&toBlock=:number`

**Response Schema:**

```tsx
interface EventsResponse {
  events: Array<{ block: Block } & (SwapEvent | JoinExitEvent)>;
}
```

**Example Response:**

```json
{
  "events": [
    {
      "block": {
        "blockNumber": 10,
        "blockTimestamp": 1673319600
      },
      "eventType": "swap",
			"txnId": "0xe9e91f1ee4b56c0df2e9f06c2b8c27c6076195a88a7b8537ba8313d80e6f124e",
			"txnIndex": 4,
			"eventIndex": 3,
			"maker": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "pairId": "0x123",
      "asset0In": 10000,
      "asset1Out": 20000,
      "priceNative": 2,
      "reserves": {
        "asset0": 500,
        "asset1": 1000
      }
    },
    {
      "block": {
        "blockNumber": 10,
        "blockTimestamp": 1673319600
      },
      "eventType": "join",
			"txnId": "0xea1093d492a1dcb1bef708f771a99a96ff05dcab81ca76c31940300177fcf49f",
			"txnIndex": 0,
			"eventIndex": 0,
			"maker": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "pairId": "0x456",
      "amount0": 10,
      "amount1": 5,
      "reserves": {
        "asset0": 100,
        "asset1": 50
      }
    },
    {
      "block": {
        "blockNumber": 11,
        "blockTimestamp": 1673406000
      },
      "eventType": "swap",
			"txnId": "0xea1093d492a1dcb1bef708f771a99a96ff05dcab81ca76c31940300177fcf49f",
			"txnIndex": 1,
			"eventIndex": 20,
			"maker": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "pairId": "0x456",
      "asset0In": 0.0123456789,
      "asset1Out": 0.000123456789,
      "priceNative": 0.00000012345,
      "reserves": {
        "asset0": 0.0001,
        "asset1": 0.000000000000001
      }
    }
  ]
}
```

**Notes:**

- `fromBlock` and `toBlock` are both inclusive: a request to `/events?fromBlock=10&toBlock=15` should include all available events from block 10, 11, 12, 13, 14 and 15

## Schemas

### Block

```tsx
interface Block {
  blockNumber: number;
  blockTimestamp: number;
  metadata?: Record<string, string>;
}
```

- `blockTimestamp` should be a UNIX timestamp, **not** including milliseconds
- `metadata` includes any optional auxiliary info not covered in the default schema and not required in most cases

### Asset

```tsx
interface Asset {
  id: string;
  name: string;
  symbol: string;
  totalSupply?: string | number;
  circulatingSupply?: string | number;
  coinGeckoId?: string;
  coinMarketCapId?: string;
  metadata?: Record<string, string>;
}
```

- In most cases, asset ids will correspond to contract addresses. Ids are case-sensitive, and for EVM-compatible blockchains using checksummed addresses is highly encouraged
- All `Asset` props aside from `id` may be mutable. The Indexer will periodically query assets for their most up-to-date info
- `totalSupply` is optional but DEX Screener cannot calculate FDV/Market Cap if not available
- `circulatingSupply` is optional but DEX Screener may not be able to show accurate market cap if not available
- `coinGeckoId` and `coinMarketCapId` are optional but may be used for displaying additional token information such as image, description and self-reported/off-chain circulating supply
- `metadata` includes any optional auxiliary info not covered in the default schema and not required in most cases

### Pair

```tsx
interface Pair {
  id: string;
	dexKey: string;
  asset0Id: string;
  asset1Id: string;
	createdAtBlockNumber?: number;
	createdAtBlockTimestamp?: number;
	createdAtTxnId?: string;
  feeBps?: number;
  pool?: {
    id: string;
    name: string;
    assetIds: string[];
    pairIds: string[];
    metadata?: Record<string, string>;
  };
  metadata?: Record<string, string>;
}
```

- All `Pair` props are immutable - Indexer will not query a given pair more than once
- In most cases, pair ids will correspond to contract addresses. Ids are case-sensitive, and for EVM-compatible blockchains using checksummed addresses is highly encouraged.
- `dexKey` is an identifier for the DEX that hosts this pair. For most adapters this will be a static value such as `uniswap`, but if multiple DEXes are tracked an id such as a factory address may be used.
- `asset0` and `asset1` order should **never** change. `amount0/reserve0` will always refer to the same `asset0`, and `amount1/reserve1` will always refer to the same `asset1`. If asset order mutates then all data after the change will be invalid and a re-index will be required.
    - A simple strategy to keep this in check is to simply order assets alphabetically. For example, in a pair containing assets `0xAAA` and `0xZZZ`, `asset0=0xAAA` and `asset1=0xZZZ`
    - DEX Screener UI will automatically invert pairs as needed and default to their most logical order (i.e.: `BTC/USD` as opposed to `USD/BTC`)
- `createdAtBlockNumber`, `createdAtBlockTimestamp` and `createdAtTxnId` are optional but encouraged. If unavailable DEX Screener can bet set to assume pair creation date is the same date as its first ever event.
- `feeBps` corresponds to swap fees in bps. For instance, a fee of 1% maps to `feeBps=100`
- `pool` is only recommended for DEXes that support multi-asset pools and allows the DEX Screener UI to correlate multiple pairs in the same multi-asset pool
- `metadata` includes any optional auxiliary info not covered in the default schema and not required in most cases

### Swap Event

```tsx
interface SwapEvent {
  eventType: "swap";
	txnId: string;
	txnIndex: number;
	eventIndex: number;
	maker: string;
  pairId: string;
  asset0In?: number | string;
  asset1In?: number | string;
  asset0Out?: number | string;
  asset1Out?: number | string;
  priceNative: number | string;
  reserves?: {
    asset0: number | string;
    asset1: number | string;
  };
  metadata?: Record<string, string>;
}
```

- `txnId` is a transaction identifier such as a transaction hash
- `txnIndex` refers to the order of a transaction within a block, the higher the index the later in the block the transaction was processed
- `eventIndex` refer to the order of the event within a transaction, the higher the index the later in the transaction the event was processed
- The combination of `txnIndex` + `eventIndex` must be unique to any given event within a block, including for different event types
- `maker` is an identifier for the account responsible for submitting the transaction. In most cases the transaction is submitted by the same account that sent/received tokens, but in cases where this doesn't apply (i.e.: transaction submitted by an aggregator) an effort should be made to identify the underlying account
- All amounts (`assetIn`/`assetOut`/`reserve`) should be decimalized (`amount / (10 ** assetDecimals)`)
- Reserves refer to the pooled amount of each asset *after* a swap event has occured. If there are multiple swap events on the same block and reserves cannot be determined after each individual event then it's acceptable for only the last event to contain the `reserves` prop
    - While `reserves` are technically optional, the Indexer relies on it for accurately calculating derived USD pricing (i.e.: USD price for `FOO/BAR` derived from `BAR/USDC`) from the most suitable reference pair. Pairs with no known reserves, from both `SwapEvent` or `JoinExitEvent`, will **not** be used for derived pricing and this may result in pairs with no known USD prices
    - `reserves` are also required to calculate total liquidity for each pair: if that's not available then DEX Screener will show liquidity as `N/A` and a warning for “Unknown liquidity”
- A combination of either `asset0In + asset1Out` or `asset1In + asset0Out` is expected. If there are multiple assets in or multiple assets out then the swap event is considered invalid and indexing will halt
- `priceNative` refers to the price of `asset0` quoted in `asset1` in that event
    - For example, in a theoretical `BTC/USD` pair, `priceNative` would be `30000` (1 BTC = 30000 USD)
    - Similarly, in a theoretical `USD/BTC` pair, `priceNative` would be `0,00003333333333` (1 USD = 0,00003333333333 USD)
- `metadata` includes any optional auxiliary info not covered in the default schema and not required in most cases
- The Indexer will use up to 50 decimal places for amounts/prices/reserves and all subsequent decimal places will be ignored. Using all 50 decimal places is highly encouraged to ensure accurate prices
- The Indexer automatically handles calculations for USD pricing (`priceUsd` as opposed to `priceNative`)

### Join/Exit Event

```tsx
interface JoinExitEvent {
  eventType: "join" | "exit";
	txnId: string;
	txnIndex: number;
	eventIndex: number;
	maker: string;
  pairId: string;
  amount0: number | string;
  amount1: number | string;
  reserves?: {
    asset0: number | string;
    asset1: number | string;
  };
  metadata?: Record<string, string>;
}
```

- `txnId` is a transaction identifier such as a transaction hash
- `txnIndex` refers to the order of a transaction within a block, the higher the index the later in the block the transaction was processed
- `eventIndex` refer to the order of the event within a transaction, the higher the index the later in the transaction the event was processed
- The combination of `txnIndex` + `eventIndex` must be unique to any given event within a block, including for different event types
- `maker` is an identifier for the account responsible for submitting the transaction. In most cases the transaction is submitted by the same account that sent/received tokens, but in cases where this doesn't apply (i.e.: transaction submitted by an aggregator) an effort should be made to identify the underlying account.
- All amounts (`assetIn`/`assetOut`/`reserve`) should be decimalized (`amount / (10 ** assetDecimals)`)
- Reserves refer to the pooled amount of each asset *after* a join/exit event has occured. If there are multiple join/exit events on the same block and reserves cannot be determined after each individual event then it's acceptable for only the last event to contain the `reserves` prop.
- `metadata` includes any optional auxiliary info not covered in the default schema and not required in most cases

# Liquidity Mining Estimates API

# Retrieving Pool Rewards Data
These endpoints can be used to pull the rewards associated with either a particular Pool, or a list of Pools

## Single Pool

This endpoint returns the Rewards data for a single Pool

### HTTP Request

```jsx
GET https://api.balancer.finance/liquidity-mining/v1/pool/:address
```

### URL Parameters

|Param | Description | Required|
|---|---|---|
|address | The address of the pool | TRUE|

### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/pool/ASDF123
```

### Example Response

```jsx
{
    "success": true,
    "current_timestamp": "2020-12-28T23:14:56.645Z",
    "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
    "address": "ASDF123",
    "reward_estimate": "5024.909403748167893273",
    "velocity": "0.001617089323107406",
    "week": "1",
}
```

## List of Pools

This endpoint returns data for a list of pools

### HTTP Request

```jsx
POST https://api.balancer.finance/liquidity-mining/v1/pools
```

### Body Parameters

|Param | Description | Required|
|---|---|---|
|addresses | Comma-separated list of addresses of the pools | TRUE|

### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/pools
```

### Example Body

```json
{
	"addresses": ["liquidity_mining_estimates","ASDF123"]
}
```

### Example Response

```jsx
{
    "success": true,
    "current_timestamp": "2020-12-28T23:16:52.893Z",
    "pools": [
        {
            "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
            "address": "abcd12345",
            "reward_estimate": "123.012609369747328786",
            "velocity": "0.000000000123123123",
            "week": "1",
        },
        {
            "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
            "address": "ASDF123",
            "reward_estimate": "5025.096986109648352369",
            "velocity": "0.001617089323107406",
            "week": "1",
        }
    ]
}
```

# Retrieving Liquidity Provider Data

## Single Liquidity Provider

This endpoint returns the Rewards data for a single liquidity provider

### HTTP Request

```jsx
GET https://api.balancer.finance/liquidity-mining/v1/liquidity-provider/:address
```

### URL Parameters

|Param | Description | Required|
|---|---|---|
|address | The address of the liquidity provider | TRUE|

### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/liquidity-provider/ASDF123
```

### Example Response

```jsx
{
    "success": true,
    "current_timestamp": "2020-12-28T23:14:56.645Z",
    "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
    "address": "ASDF123",
    "reward_estimate": "5024.909403748167893273",
    "velocity": "0.001617089323107406",
    "week": "1",
}
```

## List of Liquidity Providers

This endpoint returns data for a list of liquidity providers

### HTTP Request

```jsx
POST https://api.balancer.finance/liquidity-mining/v1/liquidity-providers
```

### Body Parameters

|Param | Description | Required|
|---|---|---|
|addresses | Comma-separated list of addresses of the liquidity providers | TRUE|

### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/liquidity-providers
```

### Example Body

```json
{
	"addresses": ["ASDF123","ABCD12345"]
}
```

### Example Response

```jsx
{
    "success": true,
    "current_timestamp": "2020-12-28T23:16:52.893Z",
    "liquidity-providers": [
        {
            "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
            "address": "abcd12345",
            "reward_estimate": "123.012609369747328786",
            "velocity": "0.000000000123123123",
            "week": "1",
        },
        {
            "snapshot_timestamp": "2020-11-30T00:05:18.000Z",
            "address": "ASDF123",
            "reward_estimate": "5025.096986109648352369",
            "velocity": "0.001617089323107406",
            "week": "1",
        }
    ]
}
```

# Response Definitions
| Parameter | Definition |
|---|---|
|success|True/False dependant on if an error occurred|
|current_timestamp|The timestamp of when the request was received by the server - this is the timestamp used to calculate any estimates|
|snapshot_timestamp|The last time a snapshot was run, and the rewards were calculated/velocity determined|
|address|The address of the Pool/Liquidity Provider|
|velocity|The rate at which the rewards were increasing when the last snapshot took place - this is in bal/second|
|reward_estimate|An estimated current reward amount|
|week| |

# Liquidity Mining Estimates API
Estimated BAL earned at each pool and by each liquidity provider can be retrieved via an API. Cumulative estimates for the week in progress are computed every hour, and the velocity with each BAL is accruing is computed from the difference between two consecutive estimates. Clients can then use the velocity to update the estimates retrieved from the API.
# Retrieving Pool Data
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
| success | True/False dependant on if an error occurred |
| current_timestamp | The timestamp of when the request was received by the server |
| snapshot_timestamp | The last time the mining estimator script was executed and velocity was determined |
| address | The address of the Pool or the Liquidity Provider, depending on the endpoint used |
| velocity | The estimated rate, in `BAL/second`, at which BAL was being mined by the address |
| reward_estimate | The estimated total BAL mined in the `week` up to time `current_timestamp` |
| week | The number of week that the estimates refer to, `1` being the first week of the liquidity mining program |

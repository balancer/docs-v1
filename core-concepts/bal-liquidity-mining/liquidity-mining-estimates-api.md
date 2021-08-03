# Liquidity Mining Estimates API

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/bal-liquidity-mining/liquidity-mining-estimates-api)

## Liquidity Mining Estimates API

### Liquidity Mining Estimates API

This API provides an estimate amount of tokens earned by a liquidity provider at the current week \(and previous week if tokens weren't sent out yet\). Cumulative estimates for the week in progress are computed every hour, and the velocity with each tokens are being accrued is computed from the difference between two consecutive estimates. Clients can then use the velocity to update in real time the estimates retrieved from the API.

### Retrieving Liquidity Provider Data

#### HTTP Request

```jsx
GET https://api.balancer.finance/liquidity-mining/v1/liquidity-provider-multitoken/:address
```

#### URL Parameters

| Param | Description | Required |
| :--- | :--- | :--- |
| address | The address of the liquidity provider | TRUE |

#### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/liquidity-provider-multitoken/0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7
```

#### Example Response

```jsx
{
  "success": true,
  "result": {
    "current_timestamp": "2021-08-02T22:11:58.714Z",
    "liquidity-providers": [
      {
        "snapshot_timestamp": "2021-08-02T12:58:05.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x0d500b1d8e8ef31e21c99d1db9a6444d3adf1270",
        "chain_id": 137,
        "current_estimate": "52.852122329992465537",
        "velocity": "0.000661201814487363",
        "week": 62
      },
      {
        "snapshot_timestamp": "2021-08-02T12:58:05.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x9a71012b13ca4d3d0cdc72a177df3ef03b0e76a3",
        "chain_id": 137,
        "current_estimate": "3.523474821999491233",
        "velocity": "0.000044080120965824",
        "week": 62
      },
      {
        "snapshot_timestamp": "2021-08-01T23:58:06.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x0d500b1d8e8ef31e21c99d1db9a6444d3adf1270",
        "chain_id": 137,
        "current_estimate": "327.477708233340251809",
        "velocity": "0",
        "week": 61
      },
      {
        "snapshot_timestamp": "2021-08-01T23:58:06.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x9a71012b13ca4d3d0cdc72a177df3ef03b0e76a3",
        "chain_id": 137,
        "current_estimate": "21.831847215556017261",
        "velocity": "0",
        "week": 61
      },
      {
        "snapshot_timestamp": "2021-08-01T23:58:06.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x580a84c73811e1839f75d86d75d88cca0c241ff4",
        "chain_id": 137,
        "current_estimate": "0",
        "velocity": "0",
        "week": 61
      },
      {
        "snapshot_timestamp": "2021-08-01T23:58:06.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0xF501dd45a1198C2E1b5aEF5314A68B9006D842E0",
        "chain_id": 137,
        "current_estimate": "0",
        "velocity": "0",
        "week": 61
      },
      {
        "snapshot_timestamp": "2021-08-02T12:58:05.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0x580a84c73811e1839f75d86d75d88cca0c241ff4",
        "chain_id": 137,
        "current_estimate": "0",
        "velocity": "0.000000000000000000",
        "week": 62
      },
      {
        "snapshot_timestamp": "2021-08-02T12:58:05.000Z",
        "address": "0xa9F8E7337eBb7982f9f5497BC5Ae98e69e1a39A7",
        "token_address": "0xF501dd45a1198C2E1b5aEF5314A68B9006D842E0",
        "chain_id": 137,
        "current_estimate": "0",
        "velocity": "0.000000000000000000",
        "week": 62
      }
    ]
  }
}
```

#### Response Definitions

| Parameter | Definition |
| :--- | :--- |
| success | `false` if an error occurred, `true` otherwise |
| current\_timestamp | The timestamp of when the request was received by the server |
| snapshot\_timestamp | The last time the mining estimator script was executed and velocity was determined |
| address | The address of the liquidity provider |
| token\_address | The address of the token received as incentive for liquidity provided |
| chain\_id | The ID of the chain in which liquidity was provided |
| velocity | The estimated rate, in`tokens/second`, at which tokens were being mined by the address at the last time the estimator script was run |
| current\_estimate | The estimated total amount of `token` mined by `address` in the `week` up to time `current_timestamp` |
| week | The number of the week that the estimates refer to, `1` being the week between `Jun-01-2020 00:00:00` and `Jun-07-2020 23:59:59` |

### Client side updates

Clients can update the real time estimate by increasing the `current_estimate` retrieved from the API by `velocity` every second. The underlying data served by the API is updated every other hour, but it takes about 10 minutes to run the script, so clients should not expect any changes before at least 70 minutes have passed since `snapshot_timestamp`.

### Notes

* Estimates are approximate, as they do not account for liquidity added/removed since the last time the script was run.
* The API only provides estimates for weeks that have not yet been finalized; as soon as the estimator script detects that the previous weeks claims have been [made available](https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-claim/snapshot), the estimates for that week are removed from the underlying dataset.


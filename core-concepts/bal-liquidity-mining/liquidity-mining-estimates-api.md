# Liquidity Mining Estimates API

## Liquidity Mining Estimates API

Estimated BAL earned at each pool and by each liquidity provider can be retrieved via this API. Cumulative estimates for the week in progress are computed every hour, and the velocity with each BAL is accruing is computed from the difference between two consecutive estimates. Clients can then use the velocity to update the estimates retrieved from the API.

## Retrieving Pool Data

These endpoints can be used to fetch the aggregate amount of BAL mined by all the liquidity providers of a pool or a set of pools.

### Single Pool

#### HTTP Request

```jsx
GET https://api.balancer.finance/liquidity-mining/v1/pool/:address
```

#### URL Parameters

| Param | Description | Required |
| :--- | :--- | :--- |
| address | The address of the pool | TRUE |

#### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/pool/0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5
```

#### Example Response

```jsx
{
  "success": true,
  "result": {
    "current_timestamp": "2021-03-01T21:05:49.590Z",
    "pools": [
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "24891.553499185869441135",
        "velocity": "0",
        "week": 39
      },
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "2844.376110032700538769",
        "velocity": "0.036697707708281742",
        "week": 40
      }
    ]
  }
}
```

### List of Pools

#### HTTP Request

```jsx
POST https://api.balancer.finance/liquidity-mining/v1/pools
```

#### Body Parameters

| Param | Description | Required |
| :--- | :--- | :--- |
| addresses | List of pool addresses | TRUE |

#### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/pools
```

#### Example Body

```javascript
{
  "addresses": [
    "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
    "0x49ff149d649769033d43783e7456f626862cd160"
  ]
}
```

#### Example Response

```jsx
{
  "success": true,
  "result": {
    "current_timestamp": "2021-03-01T21:05:51.354Z",
    "pools": [
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0x49ff149d649769033d43783e7456f626862cd160",
        "current_estimate": "940.916868590979561304",
        "velocity": "0",
        "week": 39
      },
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "24891.553499185869441135",
        "velocity": "0",
        "week": 39
      },
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0x49ff149d649769033d43783e7456f626862cd160",
        "current_estimate": "117.508925428870877674",
        "velocity": "0.001506078239721282",
        "week": 40
      },
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "2844.449505448117102253",
        "velocity": "0.036697707708281742",
        "week": 40
      }
    ]
  }
}
```

## Retrieving Liquidity Provider Data

### Single Liquidity Provider

#### HTTP Request

```jsx
GET https://api.balancer.finance/liquidity-mining/v1/liquidity-provider/:address
```

#### URL Parameters

| Param | Description | Required |
| :--- | :--- | :--- |
| address | The address of the liquidity provider | TRUE |

#### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/liquidity-provider/0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6
```

#### Example Response

```jsx
{
  "success": true,
  "result": {
    "current_timestamp": "2021-03-01T21:05:52.906Z",
    "liquidity-providers": [
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "412.683708232827449422",
        "velocity": "0.005255637365620232",
        "week": 40
      },
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "2873.562341070192815096",
        "velocity": "0",
        "week": 39
      }
    ]
  }
}
```

### List of Liquidity Providers

#### HTTP Request

```jsx
POST https://api.balancer.finance/liquidity-mining/v1/liquidity-providers/
```

#### Body Parameters

| Param | Description | Required |
| :--- | :--- | :--- |
| addresses | List of liquidity providers addresses | TRUE |

#### Example Request

```jsx
https://api.balancer.finance/liquidity-mining/v1/liquidity-providers/
```

#### Example Body

```javascript
{
  "addresses": [
    "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
    "0x49a2DcC237a65Cc1F412ed47E0594602f6141936"
  ]
}
```

#### Example Response

```jsx
{
  "success": true,
  "result": {
    "current_timestamp": "2021-03-01T21:05:54.532Z",
    "liquidity-providers": [
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "412.694219507558689886",
        "velocity": "0.005255637365620232",
        "week": 40
      },
      {
        "snapshot_timestamp": "2021-03-01T19:50:35.000Z",
        "address": "0x49a2DcC237a65Cc1F412ed47E0594602f6141936",
        "current_estimate": "764.231138137508674059",
        "velocity": "0.010903977227043695",
        "week": 40
      },
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "2873.562341070192815096",
        "velocity": "0",
        "week": 39
      },
      {
        "snapshot_timestamp": "2021-02-28T23:51:04.000Z",
        "address": "0x49a2DcC237a65Cc1F412ed47E0594602f6141936",
        "current_estimate": "6078.915337562162676477",
        "velocity": "0",
        "week": 39
      }
    ]
  }
}
```

## Response Definitions

| Parameter | Definition |
| :--- | :--- |
| success | `false` if an error occurred, `true` otherwise |
| current\_timestamp | The timestamp of when the request was received by the server |
| snapshot\_timestamp | The last time the mining estimator script was executed and velocity was determined |
| address | The address of the pool or the liquidity provider, depending on the endpoint used |
| velocity | The estimated rate, in `BAL/second`, at which BAL was being mined by the address last time the estimator script was run |
| current\_estimate | The estimated total BAL mined in the `week` up to time `current_timestamp` |
| week | The number of the week that the estimates refer to, `1` being the week between `Jun-01-2020 00:00:00` and `Jun-07-2020 23:59:59` |

## Client side updates

Clients can update the real time estimate by increasing the `current_estimate` retrieved from the API by `velocity` every second. The underlying data served by the API is updated every other hour, but it takes about 40 minutes to run the script, so clients should not expect any changes before at least 160 minutes have passed since `snapshot_timestamp`.

## Notes

* Estimates are approximate, as the actual mining script relies on a set of snapshot blocks that can only be determined after the end of the week.
* Liquidity providers estimates take into account the govFactor; pools estimates do not.
* The API only provides estimates for weeks that have not yet been finalized; as soon as the estimator script detects that the previous weeks claims have been [made available](https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-claim/snapshot), the estimates for that week are removed from the underlying dataset.


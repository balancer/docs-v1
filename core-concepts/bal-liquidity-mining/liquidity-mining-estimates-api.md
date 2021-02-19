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
    "current_timestamp": "2021-02-18T20:14:58.945Z",
    "pools": [
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "15641.269416764125622345",
        "velocity": "0.047098212932782470",
        "week": 38
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
    "current_timestamp": "2021-02-18T20:16:19.616Z",
    "pools": [
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0x49ff149d649769033d43783e7456f626862cd160",
        "current_estimate": "491.552587731498458112",
        "velocity": "0.001479777794362991",
        "week": 38
      },
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0x1eff8af5d577060ba4ac8a29a13525bb0ee2a3d5",
        "current_estimate": "15645.084372011681002415",
        "velocity": "0.047098212932782470",
        "week": 38
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
    "current_timestamp": "2021-02-18T20:16:21.439Z",
    "liquidity-providers": [
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "4648.386462398353610112",
        "velocity": "0.013993535037820807",
        "week": 38
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
    "current_timestamp": "2021-02-18T20:16:22.899Z",
    "liquidity-providers": [
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0x49a2DcC237a65Cc1F412ed47E0594602f6141936",
        "current_estimate": "3472.144685499055635212",
        "velocity": "0.010452505653507421",
        "week": 38
      },
      {
        "snapshot_timestamp": "2021-02-18T18:52:20.000Z",
        "address": "0xAfC2F2D803479A2AF3A72022D54cc0901a0ec0d6",
        "current_estimate": "4648.414449468429251726",
        "velocity": "0.013993535037820807",
        "week": 38
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

Clients can update the real time estimate by increasing the `current_estimate` retrieved from the API by `velocity` every second. The underlying data served by the API is updated hourly, but it takes about 40 minutes to run the script, so clients should not expect any changes before at least 100 minutes have passed since `snapshot_timestamp`.

## Notes

* Estimates are approximate, as the actual mining script relies on a set of snapshot blocks that can only be determined after the end of the week.
* Liquidity providers estimates take into account the govFactor; pools estimates do not.
* The API only provides estimates for weeks that have not yet been finalized; as soon as the estimator script detects that the previous weeks claims have been [made available](https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-claim/snapshot), the estimates for that week are removed from the underlying dataset.


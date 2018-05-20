# 0x Standard Relayer API

Crypdex supports the 0x's [Standard Relayer API v0](https://github.com/0xProject/standard-relayer-api/blob/master/http/v0.md). Developers may find more detailed information and interactions in our standard REST API.

## General Information

### Pagination

* Requests that return multiple items respond to the ?page and ?per_page query parameters.
* Page numbering is 1-indexed, not 0-indexed. Maximum page size is be 100.
* These requests include the `token_pairs`, `orders`, and `orderbook` endpoints.

### Misc

* All requests and responses are of application/json content type
* All token amounts are sent in amounts of the smallest level of precision (base units). (e.g if a token has 18 decimal places, selling 1 token would show up as selling '1000000000000000000' units by this API).
* All addresses are sent as lower-case (non-checksummed) Ethereum addresses with the 0x prefix.

## Token pairs

```shell
curl "http://api.crypdex.io/0x/v0/token_pairs"
```

> Example Response

```json
[
  {
    "tokenA":{
      "address":"0x323b5d4c32345ced77393b3530b1eed0f346429d",
      "minAmount":"0",
      "maxAmount":"10000000000000000000",
      "precision":5
    },
    "tokenB":{
      "address":"0xef7fff64389b814a946f3e92105513705ca6b990",
      "minAmount":"0",
      "maxAmount":"50000000000000000000",
      "precision":5
    }
  },
  ...
]
```

Retrieves a list of available token pairs and the information required to trade them. This endpoint is be paginated.

### Request

`GET http://api.crypdex.io/0x/v0/token_pairs`

### Query Parameters

These are token addresses. Returns token pairs that contain tokenA and tokenB (in any order). Setting only tokenA or tokenB returns pairs filtered by that token only (optional)

| Parameter | Description     |
| --------- | --------------- |
| tokenA    | Token A address |
| tokenB    | Token B address |

### Response

| Field     | Description                                                                        |
| --------- | ---------------------------------------------------------------------------------- |
| address   | address of the token                                                               |
| minAmount | the minimum trade amount the relayer will accept                                   |
| maxAmount | the maximum trade amount the relayer will accept                                   |
| precision | the desired price precision a relayer would like to support within their orderbook |

<!-- ------ -->

<!-- ORDERS -->

<!-- ------ -->

## Orders

> [See response schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/signed_orders_schema.ts#L1)

```json
[
  {
    "exchangeContractAddress":"0x12459c951127e0c374ff9105dda097662a027093",
    "maker":"0x9e56625509c2f60af937f23b7b532600390e8c8b",
    "taker":"0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
    "makerTokenAddress":"0x323b5d4c32345ced77393b3530b1eed0f346429d",
    "takerTokenAddress":"0xef7fff64389b814a946f3e92105513705ca6b990",
    "feeRecipient":"0xb046140686d052fff581f63f8136cce132e857da",
    "makerTokenAmount":"10000000000000000",
    "takerTokenAmount":"20000000000000000",
    "makerFee":"100000000000000",
    "takerFee":"200000000000000",
    "expirationUnixTimestampSec":"42",
    "salt":"67006738228878699843088602623665307406148487219438534730168799356281242528500",
    "ecSignature":{
      "v":27,
      "r":"0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
      "s":"0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
    }
  },
  ...
]
```

Retrieves a list of orders given query parameters. This endpoint should be paginated. For querying an entire orderbook snapshot, the orderbook endpoint is recommended.

### Request

`GET http://api.crypdex.io/0x/v0/orders`

### Query Parameters

All parameters are optional.

| Parameter                   | Description                                                                  |
| --------------------------- | ---------------------------------------------------------------------------- |
| **exchangeContractAddress** | returns orders created for this exchange address                             |
| **tokenAddress**            | returns orders where makerTokenAddress or takerTokenAddress is token address |
| **makerTokenAddress**       | returns orders with specified makerTokenAddress                              |
| **takerTokenAddress**       | returns orders with specified makerTokenAddress                              |
| **maker**                   | returns orders where maker is maker address                                  |
| **taker**                   | returns orders where taker is taker address                                  |
| **trader**                  | returns orders where maker or taker is trader address                        |
| **feeRecipient**            | returns orders where feeRecipient is feeRecipient address                    |

If both makerTokenAddress and takerTokenAddress are specified, returned orders will be sorted by price determined by (takerTokenAmount/makerTokenAmount) in ascending order. By default, orders returned by this endpoint are unsorted.

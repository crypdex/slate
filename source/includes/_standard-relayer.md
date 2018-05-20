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

## List Token Pairs

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

## List Orders

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

Retrieves a specific order by orderHash.

## Show Order

> [See response schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/order_schemas.ts#L24)

```json
{
  "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
  "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
  "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
  "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
  "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
  "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
  "makerTokenAmount": "10000000000000000",
  "takerTokenAmount": "20000000000000000",
  "makerFee": "100000000000000",
  "takerFee": "200000000000000",
  "expirationUnixTimestampSec": "42",
  "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500",
  "ecSignature": {
    "v": 27,
    "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
    "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
  }
}
```

### Request

`GET http://api.crypdex.io/0x/v0/order/:orderHash`

Returns HTTP 404 if no order with specified orderHash was found.

## Show Orderbook

> Example Response

```json
{
  "bids": [
    {
      "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
      "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
      "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
      "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
      "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
      "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
      "makerTokenAmount": "10000000000000000",
      "takerTokenAmount": "20000000000000000",
      "makerFee": "100000000000000",
      "takerFee": "200000000000000",
      "expirationUnixTimestampSec": "42",
      "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500",
      "ecSignature": {
        "v": 27,
        "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
        "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
      }
    },
    ...
  ],
  "asks": [
    {
      "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
      "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
      "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
      "makerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
      "takerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
      "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
      "makerTokenAmount": "22000000000000000",
      "takerTokenAmount": "10000000000000000",
      "makerFee": "100000000000000",
      "takerFee": "200000000000000",
      "expirationUnixTimestampSec": "632",
      "salt": "54515451557974875123697849345751275676157243756715784155226239582178",
      "ecSignature": {
        "v": 27,
        "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
        "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
      }
    },
    ...
  ]
}
```

Retrieves the orderbook for a given token pair.

### Request

`GET http://api.crypdex.io/0x/v0/orderbook`

### Query Parameters

| Parameters                     | Description                                                                                        |
| ------------------------------ | -------------------------------------------------------------------------------------------------- |
| **baseTokenAddress** [string]  | address of token designated as the baseToken in the currency pair calculation of price (required)  |
| **quoteTokenAddress** [string] | address of token designated as the quoteToken in the currency pair calculation of price (required) |

### Response

[See response schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/relayer_api_orderbook_response_schema.ts#L1)

| Key      | Description                                                                 |
| -------- | --------------------------------------------------------------------------- |
| **bids** | array of signed orders where takerTokenAddress is equal to baseTokenAddress |
| **asks** | array of signed orders where makerTokenAddress is equal to baseTokenAddress |

Bids will be sorted in descending order by price, and asks will be sorted in ascending order by price. Within the price sorted orders, the orders are further sorted first by total fees, then by expiration in ascending order.

## Calculate Fees

> Example Request

```json
{
  "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
  "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
  "taker": "0x0000000000000000000000000000000000000000",
  "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
  "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
  "makerTokenAmount": "10000000000000000",
  "takerTokenAmount": "20000000000000000",
  "expirationUnixTimestampSec": "42",
  "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500"
}
```

> Example Response

```json
{
  "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
  "makerFee": "100000000000000",
  "takerFee": "200000000000000"
}
```

Given an unsigned order without the fee-related properties, returns the required feeRecipient, makerFee, and takerFee of that order.

### Request

`POST http://api.crypdex.io/0x/v0/fees`

[See request schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/relayer_api_fees_payload_schema.ts#L1)

The request payload is essentially an unsigned order without fee-related properties.

### Response

[See response schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/relayer_api_fees_response_schema.ts#L1)

## Create Order

> Example Request

```json
{
  "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
  "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
  "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
  "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
  "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
  "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
  "makerTokenAmount": "10000000000000000",
  "takerTokenAmount": "20000000000000000",
  "makerFee": "100000000000000",
  "takerFee": "200000000000000",
  "expirationUnixTimestampSec": "42",
  "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500",
  "ecSignature": {
    "v": 27,
    "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
    "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
  }
}
```

> Example Error Response

```json
{
  "code": 101,
  "reason": "Validation failed",
  "validationErrors": [
    {
      "field": "maker",
      "code": 1002,
      "reason": "Invalid address"
    }
  ]
}
```

Submit a signed order to the relayer.

### Request

`POST http://api.crypdex.io/0x/v0/order`

[See request schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/order_schemas.ts#L24)

### Response

#### Success Response

Returns HTTP 201 upon success.

#### Error Response

Error response will be sent with a non-2xx HTTP status code

[See error response schema](https://github.com/0xProject/0x.js/blob/development/packages/json-schemas/schemas/relayer_api_error_response_schema.ts#L1)

### Error Codes

#### General Error Codes

| Code | Reason                    |
| ---- | ------------------------- |
| 100  | Validation Failed         |
| 101  | Malformed JSON            |
| 102  | Order submission disabled |
| 103  | Throttled                 |

#### Validation Error Codes

| Code | Reason                |
| ---- | --------------------- |
| 1000 | Required field        |
| 1001 | Incorrect format      |
| 1002 | Invalid address       |
| 1003 | Address not supported |
| 1004 | Value out of range    |
| 1005 | Invalid ECDSA or Hash |
| 1006 | Unsupported option    |

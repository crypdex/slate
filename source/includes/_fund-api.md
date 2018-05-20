# Fund API

Crypdex

## General Information

### Pagination

* Requests that return multiple items respond to the ?page and ?per_page query parameters.
* Page numbering is 1-indexed, not 0-indexed. Maximum page size is be 100.
* These requests include the `token_pairs`, `orders`, and `orderbook` endpoints.

### Misc

* All requests and responses are of application/json content type
* All token amounts are sent in amounts of the smallest level of precision (base units). (e.g if a token has 18 decimal places, selling 1 token would show up as selling '1000000000000000000' units by this API).
* All addresses are sent as lower-case (non-checksummed) Ethereum addresses with the 0x prefix.

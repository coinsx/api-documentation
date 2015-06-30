---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the BTC.sx API.

This documentation is in pre-alpha state.

# Authentication

> Example of authentication

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

> Make sure to replace `meowmeowmeow` with your API key.

We use long-lived application keys to authenticate requests. Please contact support (mailto://support@btc.sx).

Every request should be signed with the following format... TODO

<aside class="notice">
Notice...
</aside>

# Trading

## Place trade

```shell
curl "https://btc.sx/v1/api/trades"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1
  },
  {
    "id": 2
  }
]
```

Returns 200 OK when the trade is correctly placed. Otherwise, please see error list.

### HTTP Request

`POST https://btc.sx/v1/api/trade`

### Query Parameters

Parameter | Required | Default | Description
--------- | ------- | ----------- | ----------------
exchange | yes | none | Exchange to be used for the trade
positionSize | yes | none | Size of the position to market.
leverage | no | 5 | Leverage of the position (valid values are 2, 5 or 10).

<aside class="success">
Has to be authenticated
</aside>

## Get trade

```shell
curl "https://btc.sx/v1/api/trade"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2
}
```

This endpoint retrieves a specific trade.

### HTTP Request

`GET https://btc.sx/v1/api/trade<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the trade to retrieve


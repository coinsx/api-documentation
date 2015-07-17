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

> The code below is an example of this process in Javascript.

```javascript
        // transform all request to include basic auth.
        var headers = headersGetter();
        var tonce = new Date().getTime() * 1000; // any tonce
        var shaObj = new jsSHA(tonce + publicKey + data, "TEXT");
        var hmacSignature = shaObj.getHMAC(privateKey, "TEXT", "SHA-512", "B64");

        headers.Authorization = "Basic " + Base64.encode(publicKey + ":" + hmacSignature);
        headers.Tonce = tonce;

```

> Example of authentication

```shell
# With shell, you can just pass the correct header with each request
curl "apiEndpoing"
  -H "Authorization: Basic theHash"
  -H "Tonce: theTonce"
```

> Make sure to replace "theHash" with the result of the HMAC signature, and "theTonce" with the tonce used in the request.

We use pairs of long-lived application keys to authenticate requests. Please contact <support@btc.sx> to request an API key .

Every request should include the following header values

Header | Required | Description
--------- | ------- | -----------
Tonce | yes | Continuously increasing numeric value.
Authorization | yes | The string "Basic ", appended with the signature resulting of calculating the HMAC SHA-512 value of the tonce, the public key that you were assigned, and the body of the request.



<aside class="notice">
Notice...
</aside>

# Trading

## Place trade

```shell
curl "https://btc.sx/v1/api/trades"
  -H "Authorization: Basic theHash"
  -H "Tonce: theTonce"
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
  -H "Authorization: Basic theHash"
  -H "Tonce: theTonce"
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


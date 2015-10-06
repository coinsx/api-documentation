---
title: API Reference

language_tabs:
  - javascript
  - shell
  - php

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the magnr.com Trading API.

This documentation is in pre-alpha state.

# Authentication

> The code below is an example of this process in Javascript.

```javascript
        // transform all request to include basic auth.
        var tonce = new Date().getTime() * 1000; // any tonce
        var shaObj = new jsSHA(tonce + publicKey + data, "TEXT");
        var hmacSignature = shaObj.getHMAC(privateKey, "TEXT", "SHA-512", "B64");

        headers.Authorization = "Basic " + Base64.encode(publicKey + ":" + hmacSignature);
        headers.Tonce = tonce;

```

> Example of authentication

```shell
# With shell, you can just pass the correct header with each request
curl "/api/v1/<some end point>/"
  -H "Authorization: Basic <base64(pubkey:HMAC)>"
  -H "Tonce: <a tonce>"
```

> Make sure to replace "<base64(pubkey:HMAC)>" with the result of the HMAC signature, and "<a tonce>" with the tonce used in the request.

We use pairs of long-lived application keys to authenticate requests. Please contact <support@magnr.com> to request an API key .

Every request should include the following header values

Header | Required | Description
--------- | ------- | -----------
Tonce | yes | It should be a non-repeating number, and be around a 5 minutes window around the current timestamp.
Authorization | yes | The string "Basic ", appended with the signature resulting of calculating the HMAC SHA-512 value of the tonce, the public key that you were assigned, and the body of the request.



<aside class="notice">
Notice...
</aside>

# Trading

## Place trade

```shell
curl "https://btc.sx/v1/api/trades"
  -X POST
  -H "Authorization: Basic <AUTH>"
  -H "Tonce: <tonce>"
  -d '{"leverage":10, "side":"BUY", "exchange":"Bitfinex", "pair":"BTCUSD", "margin":5, "size":0.2, "take":10}'
```

Returns 201 CREATED when the trade is correctly placed. Otherwise, please see error list.

> Response

```json
{
  "id":"<some identifier>"
}
```

### HTTP Request

`POST https://magnr.com/api/v1/trades/`

### BODY Parameters

Parameter | Type | Required | Default | Description
--------- | ---- | ------- | ----------- | ----------------
leverage | number | YES | none | 10,5,2 (Leverage to apply on this trade)
side | string | YES | none | buy or sell (case sensitive)
exchange | string | YES | none | Bitstamp,ItBit,Bitfinex (Exchange to be used for the trade)
pair | string | YES | none | BTCUSD, XBTUSD (exchange dependent)
margin | float | YES | none | An amount in $USD the price can move against you before being automatically closed out. 
size | float | YES | none | Size of the position to market.
type | string | NO | market | market is the only option at present
take |float | NO | none | The amount in $USD you wish the price to move in your favour before closing out.

<aside class="success">
Has to be authenticated
</aside>

### RESPONSE

Parameter | Type | Description
--------- | ---- | ----------------
id | string | identifier to look up trade thereafter

### ERRORS

Response | Description
-------- | -----------
400 Bad Request | The request parameters were missing or incorrect, check the response object
409 Conflict | The request could be completed due to a state conflict with the account, check the response object
500 Internal Server Error | The system encountered an error, or the exchange failed to accept the trade, check the response object 

## Get trade

```shell
curl "https://magnr.com/api/v1/trades/"
  -H "Authorization: Basic <AUTH>"
  -H "Tonce: <tonce>"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2
}
```

This endpoint retrieves a specific trade.

### HTTP Request

`GET https://magnr.com/api/v1/trades/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the trade to retrieve


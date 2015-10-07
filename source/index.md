---
title: API Reference

language_tabs:
  - shell
  - javascript
  - php

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

Welcome to the magnr.com Trading API.

This documentation is in pre-alpha state.

## Environments

Magnr provides two API environments. 

1. A production environent at https://magnr.com/api/v1/
2. A sandbox environment at https://sandbox.magnr.com/api/v1/

Tokens between the 2 environments differ so you will have to request one token from each environment separately.

<aside class="notice">
In the SANDBOX environment it is advisable to utilise the "LOCAL" exchange and "BTCUSD" currency pair instead of the known production exchanges/pairs.
</aside>

## Error Handling

> Example JSON Error

```json
{
  "errors": ["some error string", "another error string"]
}
```

All errors follow general REST principles. Included in the body of any error response (e.g. non-2xx status code) will be an error object of the form:

Parameter | Value
---------- | -------
errors | List of error strings

# Authentication


> Example of authentication

```javascript
// transform all request to include basic auth.
var tonce = new Date().getTime() * 1000; // any tonce
var shaObj = new jsSHA(tonce + publicKey + data, "TEXT");
var hmacSignature = shaObj.getHMAC(privateKey, "TEXT", "SHA-512", "B64");

headers.Authorization = "Basic " + Base64.encode(publicKey + ":" + hmacSignature);
headers.Tonce = tonce;
```

```php
<?php
$privkey = "your private key";
$pubkey = "your public key";

// @note SET THIS DATA CORRECTLY ON EACH REQUEST AND BEFORE SIG AND AUTH GENERATION
$data = "empty string or json string";
$tonce = intval(microtime(true)*1000);

$sig = base64_encode(hash_hmac("sha512", $tonce.$pubkey.$data, $privkey, true));
$auth = base64_encode($pubkey.":".$sig);
```

```shell
# With shell, you can pass a signed header with each request
TONCE=$(($(date +%s)*1000))
PRIVKEY=`echo -e "your private key here"`
PUBKEY=`echo -e "your public key here"`
DATA="some json or empty string"
SIG=`echo -en "$TONCE$PUBKEY$DATA" | openssl dgst -sha512 -hmac "$PRIVKEY" -binary | base64`
AUTH=`echo -en "$PUBKEY:$SIG" | base64`
curl "https://magnr.com/api/v1/<some end point>/"
  -H "Content-Type: application/json"
  -H "Authorization: Basic $AUTH"
  -H "Tonce: $TONCE"
  -d $DATA
  
Note: The rest of the shell examples assume the shell variables TONCE and AUTH are regenerated per request.
```

We use pairs of long-lived application keys to authenticate requests. Please contact <support@magnr.com> to request an API key .

Every request should include the following header values

Header | Required | Description
--------- | ------- | -----------
Tonce | yes | It should be a non-repeating number, and be around a 5 minutes window around the current timestamp. THIS TONCE NEEDS TO BE MILLISECONDS
Authorization | yes | The string "Basic ", appended with the signature resulting of calculating the HMAC SHA-512 value of the tonce, the public key that you were assigned, and the body of the request.

<aside class="notice">
TONCE NEEDS TO BE IN MILLISECONDS
</aside>

# Trading

## Create Trade

```shell
DATA="{\"leverage\":10, \"side\":\"BUY\", \"exchange\":\"Bitfinex\", \"pair\":\"BTCUSD\", \"margin\":5, \"size\":0.2, \"take\":10}"
curl "https://magnr.com/api/v1/trades/" \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Basic $AUTH" \
  -H "Tonce: $TONCE" \
  -d $DATA
```

Returns 201 CREATED when the trade is correctly placed. See error list for other response codes.

> Response - 201 CREATED

```php
<?php
$data = json_encode(["leverage"=>10, "side"=>"buy", "exchange"=>"Bitfinex", "pair"=>"BTCUSD", "margin"=>5, "size"=>0.2, "take"=>10]);

/**
 * DO AUTH STUFF HERE TO GENERATE $auth AND $tonce
 */

$curl = curl_init("https://magnr.com/api/v1/trades/$id/");
// Set to a POST request
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $data);

// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: BASIC $auth", "Tonce: $tonce", "Content-Type: application/json", "Accept: application/json"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

$json = json_decode($body,true); // the json as an array
echo "STATUS: $status\nBODY:\n$body";
```


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

### RESPONSE

Parameter | Type | Description
--------- | ---- | ----------------
id | string | Trade identifier

### ERRORS

Response | Description
-------- | -----------
400 Bad Request | The request parameters were missing or incorrect
409 Conflict | The request could be completed due to a state conflict with the account
500 Internal Server Error | The system encountered an error, or the exchange failed to accept the trade
 
## List Trades

```shell
curl "https://magnr.com/api/v1/trades/"
  -H "Content-Type: application/json"
  -H "Accept: application/json"
  -H "Authorization: Basic $AUTH"
  -H "Tonce: <tonce>"
```

```php
<?php

$data = "";

/**
 * DO AUTH STUFF HERE TO SET $auth AND $tonce
 */

$curl = curl_init("https://magnr.com/api/v1/trades/");
// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: BASIC $auth", "Tonce: $tonce"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

echo "STATUS: $status\nBODY:\n$body";
```

> Response - 200 OK

```json
{
  "trades": [
    {"id":1},
    {"id":2},
    {"id":3}
  ]
}
```

### HTTP Request

`GET https://magnr.com/api/v1/trades/`

### URL Parameters

`NONE`



## Get Trade

```shell
ID=1234
// regenerate TONCE and AUTH
curl "https://magnr.com/api/v1/trades/$ID/"
  -H "Content-Type: application/json"
  -H "Accept: application/json"
  -H "Authorization: Basic $AUTH"
  -H "Tonce: $TONCE"
```

```php
<?php

$id = 1234;
$data = "";

/**
 * DO AUTH STUFF HERE TO SET $auth AND $tonce
 */

$curl = curl_init("https://magnr.com/api/v1/trades/$id/");
// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: BASIC $auth", "Tonce: $tonce"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

echo "STATUS: $status\nBODY:\n$body";
```

> Example response

```json
{
  "trade": {
    "id": 2,
    "more fields here":""
  }
}
```

This endpoint retrieves a specific trade.

### HTTP REQUEST

`GET /api/v1/trades/:id/`

### URL PARAMETERS

Parameter | Type | Required | Description
--------- | -----------
id | string | YES | The ID of the trade to retrieve

### RESPONSE

Returns a Trade object 

### ERRORS

Response | Description
-------- | -----------
400 Bad Request | The request parameters were missing or incorrect
404 Not Found | The trade you requested does not exist
500 Internal Server Error | The system encountered an error


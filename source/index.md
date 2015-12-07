---
title: API Reference

language_tabs:
  - shell: cURL
  - ruby
  - php

toc_footers:
  - <a href='https://support.magnr.com/hc/en-gb'>Support Desk</a>

search: true
---

# Introduction

Welcome to the Magnr Trading API. Technical details on how to implement and connect to our API is provided below.

## Environments

Magnr provides 2 separate environments for development and production. 
API pub/priv keys differ between the 2 environments so you will have to request one set from each environment separately.


All responses are of content type `application/json`

### PRODUCTION ENVIRONMENT

The Magnr production environment API is live and used by our own website [https://magnr.com]() .

- Production Site: [https://magnr.com/]()
- Production API : [https://magnr.com/api/v1/]()

### TEST ENVIRONMENT

All the examples of code in this documentation use the sandbox API in URLs. It is recommended that attempts to use the API are performed against sandbox before live.

- Sandbox Site: [https://sandbox.magnr.com/]()
- Sandbox API : [https://sandbox.magnr.com/api/v1/]()

The sandbox environment is connected to bitcoins testnet, consequently you can fund your account from one of many testnet faucets

- <a href="http://faucet.haskoin.com/" target="_blank">http://faucet.haskoin.com/</a>
- <a href="http://tpfaucet.appspot.com/" target="_blank">http://tpfaucet.appspot.com/</a>
- <a href="https://testnet.coinfaucet.eu/en/" target="_blank">https://testnet.coinfaucet.eu/en/</a>
- <a href="https://accounts.blockcypher.com/testnet-faucet" target="_blank">https://accounts.blockcypher.com/testnet-faucet</a>


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

curl "https://sandbox.magnr.com/api/v1/<some end point>/" \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $AUTH" \
  -H "Tonce: $TONCE" \
  -d $DATA
  
Note: The rest of the shell examples assume the shell variables TONCE and AUTH are regenerated per request.
```

```ruby
require 'uri'
require 'net/http'
require 'time'
require 'securerandom'
require 'base64'

uri = URI.parse("https://sandbox.magnr.com")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

pubkey = "YOUR PUBLIC KEY"
privkey = "YOUR PRIVATE KEY"

# milliseconds time
tonce = Time.now.to_i * 1000

data = ""
signature = Base64.strict_encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha512'), privkey, tonce.to_s+pubkey+data))

request = Net::HTTP::Get.new("/api/v1/<some end point>/")
request.add_field('Content-Type', 'application/json')
request.add_field('Tonce', tonce)
request.basic_auth(pubkey,signature)

response = http.request(request)
print response.body
```

We use pairs of long-lived application keys to authenticate requests. To request an API key log in to Magnr, click on Account Settings (In the Account menu drop-down) and select the API tab.

Every request should include the following header values

Header | Required | Description
--------- | ------- | -----------
Tonce | yes | It should be a non-repeating number, and be around a 5 minutes window around the current timestamp. THIS TONCE NEEDS TO BE IN MILLISECONDS
Authorization | yes | The string "Basic ", appended with the signature resulting of calculating the HMAC SHA-512 value of the tonce, the public key that you were assigned, and the body of the request.

<aside class="notice">
TONCE NEEDS TO BE IN MILLISECONDS
</aside>

# Trading

## Create Trade

```shell
DATA="{\"leverage\":10, \"side\":\"BUY\", \"exchange\":\"Bitfinex\", \"pair\":\"BTCUSD\", \"stop_margin\":5, \"size\":0.2, \"take\":10}"
curl "https://sandbox.magnr.com/api/v1/trades/" \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Basic $AUTH" \
  -H "Tonce: $TONCE" \
  -d $DATA
```

Returns 201 CREATED when the trade is correctly placed. See error list for other response codes.

```php
<?php
$data = json_encode(["leverage"=>10, "side"=>"buy", "exchange"=>"Bitfinex", "pair"=>"BTCUSD", "stop_margin"=>5, "size"=>0.2, "take"=>10]);

/**
 * DO AUTH STUFF HERE TO GENERATE $auth AND $tonce
 */

$curl = curl_init("https://sandbox.magnr.com/api/v1/trades/");
// Set to a POST request
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $data);

// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: Basic $auth", "Tonce: $tonce", "Content-Type: application/json", "Accept: application/json"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

$json = json_decode($body,true); // the json as an array
echo "STATUS: $status\nBODY:\n$body";
```

```ruby
require 'uri'
require 'net/http'
require 'time'
require 'securerandom'
require 'base64'
require 'json'

uri = URI.parse("https://sandbox.magnr.com")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

#AUTHENTICATION HERE ... (see authentication section)

pubkey = "YOUR PRIVATE KEY"
privkey = "YOUR PUBLIC KEY"

# milliseconds time
tonce = Time.now.to_i * 1000

# PLACE AN ORDER!
data = {:leverage =>10, :side =>"buy", :exchange =>"LOCAL", :pair =>"BTCUSD", :stop_margin =>5, :size =>0.3, :take =>10}.to_json
signature = Base64.strict_encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha512'), privkey, tonce.to_s+pubkey+data))

##### END AUTH

request = Net::HTTP::Post.new("/api/v1/trades/")
request.add_field('Content-Type', 'application/json')
request.add_field('Tonce', tonce)
request.basic_auth(pubkey,signature)
request.body = data

response = http.request(request)
json = JSON.parse(response.body)
print json
```

### HTTP Request

`POST /api/v1/trades/`

### BODY Parameters

Parameter | Type | Required | Default | Description
--------- | ---- | ------- | ----------- | ----------------
side | string | YES | none | buy or sell (case sensitive)
size | float | YES | none | Size of the position to market.
exchange | string | YES | none | Bitstamp,ItBit,Bitfinex (Exchange to be used for the trade)
pair | string | YES | none | BTCUSD, XBTUSD (exchange dependent)
leverage | number | NO | 10 | 10,5,2 (Leverage to apply on this trade)
stop_margin | float | NO | max stop for deposit | An amount in $USD the price can move against you before being automatically closed out. 
type | string | NO | market | market is the only option at present
take |float | NO | none | The amount in $USD you wish the price to move in your favour before closing out.

> Example response - 201 CREATED

```json
{
  "id":1234
}
```

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
curl "https://sandbox.magnr.com/api/v1/trades/"
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

$curl = curl_init("https://sandbox.magnr.com/api/v1/trades/");
// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: Basic $auth", "Tonce: $tonce"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

echo "STATUS: $status\nBODY:\n$body";
```

```ruby
require 'uri'
require 'net/http'
require 'time'
require 'securerandom'
require 'base64'
require 'json'

uri = URI.parse("https://sandbox.magnr.com")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

#AUTHENTICATION HERE ... (see authentication section)

pubkey = "YOUR PRIVATE KEY"
privkey = "YOUR PUBLIC KEY"

# milliseconds time
tonce = Time.now.to_i * 1000

data = ""
signature = Base64.strict_encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha512'), privkey, tonce.to_s+pubkey+data))

##### END AUTH

request = Net::HTTP::Get.new("/api/v1/trades/")
request.add_field('Content-Type', 'application/json')
request.add_field('Tonce', tonce)
request.basic_auth(pubkey,signature)

response = http.request(request)
json = JSON.parse(response.body)
print json
```

> Response - 200 OK

```json
{
  "trades": [
    {
        "id"  : 1234,
        "side": "buy",
        "leverage": 10,
        "size": 0.2,
        "deposit": 0.002,
        "exchange":"LOCAL",
        "created_at":1444251389,
        "closed_at": null,
        "open_price": null,
        "close_price": null,
        "closed_reason":null,
        "trade_rate": 0.48,
        "daily_rate": 0.2,
        "trades_fees": 0.004,
        "daily_fees": 0.00,
        "status": "pending",
        "profit": null
    },
    {
        "id"  : 1233,
        "side": "sell",
        "leverage": 5,
        "size": 0.3,
        "deposit": 0.008,
        "exchange":"LOCAL",
        "created_at":1444230624,
        "closed_at":1444612316,
        "open_price": 234.45,
        "close_price": 235.56,
        "closed_reason":"stop",
        "trade_rate": 0.48,
        "daily_rate": 0.2,
        "trades_fees": 0.002,
        "daily_fees": 0.0009,
        "status": "settled",
        "profit": -0.00000667
      }
  ]
}
```

### HTTP Request

`GET /api/v1/trades/`

### URL Parameters

`NONE`

### RESPONSE

Parameter | Type | Description
--------- | ---- | -----------
trades    | array| An array of trade objects (see [Get Trade](#get-trade) call)

## Get Trade

```shell
ID=1234
// regenerate TONCE and AUTH
curl "https://sandbox.magnr.com/api/v1/trades/$ID/"
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

$curl = curl_init("https://sandbox.magnr.com/api/v1/trades/$id/");
// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: Basic $auth", "Tonce: $tonce"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // curl_exec returns the result
curl_setopt($curl, CURLOPT_HEADER, 0); // don't include headers in the response from curl_exec

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

echo "STATUS: $status\nBODY:\n$body";
```

```ruby
require 'uri'
require 'net/http'
require 'time'
require 'securerandom'
require 'base64'
require 'json'

uri = URI.parse("https://sandbox.magnr.com")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

#AUTHENTICATION HERE ... (see authentication section)

pubkey = "YOUR PRIVATE KEY"
privkey = "YOUR PUBLIC KEY"

# milliseconds time
tonce = Time.now.to_i * 1000

data = ""
signature = Base64.strict_encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha512'), privkey, tonce.to_s+pubkey+data))

##### END AUTH

request = Net::HTTP::Get.new("/api/v1/trades/#{id}")
request.add_field('Content-Type', 'application/json')
request.add_field('Tonce', tonce)
request.basic_auth(pubkey,signature)

response = http.request(request)
json = JSON.parse(response.body)
print json
```

> Example response

```json
{
  "trade": {
    "id"  : 1234,
    "side": "buy",
    "leverage": 10,
    "size": 0.2,
    "deposit": 0.002,
    "exchange":"LOCAL",
    "created_at":1444230624,
    "closed_at":1444612316,
    "open_price": 234.45,
    "close_price": 235.56,
    "closed_reason":"user",
    "trade_rate": 0.48,
    "daily_rate": 0.2,
    "trades_fees": 0.004,
    "daily_fees": 0.00,
    "status": "settled",
    "profit": 0.00019382
  }
}
```

This endpoint retrieves a specific trade.

### HTTP REQUEST

`GET /api/v1/trades/:id/`

### URL PARAMETERS

Parameter | Type | Required | Description
--------- | ----------- |----- | ------
id | string | YES | The ID of the trade to retrieve

### RESPONSE

Returns a Trade object 

Field   | Type   | Description
------- | ------ | -----------
id      | string | id of the trade
side    | string | "buy" or "sell"
leverage| int    | the leverage applied on the trade
size    | float  | size of the trade at market
deposit | float  | size of the deposit taken for the trade
exchange      | string | the name of the exchange that the trade was opened on
created_at    | int    | timestamp of trade creation (in seconds since epoch)
closed_at     | int    | timestamp of the trade close (in seconds since epoch)
open_price    | float  | price the trade was opened at
close_price   | float  | price the trade closed at
closed_reason | string | the reason the trade closed (user,stop,limit,insufficient funds)
trade_rate    | float  | trade charge PERCENTAGE (e.g. 0.48%)
daily_rate    | float  | daily charge PERCENTAGE (e.g. 0.2%)
trade_fees    | float  | bitcoin trade fees (absolute) applied to the trade (note: this includes BOTH open AND close fees)
daily_fees    | float  | bitcoin daily fees (absolute) applied to the trade since it was opened
status  | string | the state of the trade (**pending** -> **open** -> **closed** -> **settled**), see status info.
profit  | float  | the profit on the trade if it has settled, otherwise NULL
stop_margin  | float  | size in currency (e.g. USD) of the stop margin
trail_margin | float  | size in currency (e.g. USD) of the trailing stop, otherwise NULL if not set
sell_limit   | float  | size in currency (e.g. USD) of the sell limit, otherwise NULL if not set

### STATUS
Field | Description
----- | -----------
pending | The trade has been placed at the exchange but is awaiting fill (i.e. open price is NULL).
open    | The trade has been filled and now has an open price
closed  | The trade has been closed either by user or system but it awaiting a closing price (i.e. close price is NULL).
settled | The trade has been filled at a close price and now has a profit, all deposits should have been refunded.

### CLOSED REASON
Field | Description
----- | -----------
user  | You (the user) have requested the trade be closed.
stop  | The trade has reached the stop margin specified, and it has been automatically closed.
trail | The trade has reached the trail margin specified, and it has been automatically closed.
limit | The trade has reached the target sell limit, and has been automatically closed.
insufficient funds | A healthly balance was NOT maintained and a daily charge would not be applied, as a result the trade has been automatically closed.

### ERRORS

Response | Description
-------- | -----------
400 Bad Request | The request parameters were missing or incorrect
404 Not Found | The trade you requested does not exist
500 Internal Server Error | The system encountered an error


## Liquidate (Close) Trade

```shell
ID=1234
// regenerate TONCE and AUTH
curl "https://sandbox.magnr.com/api/v1/trades/$ID/"
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

$curl = curl_init("https://sandbox.magnr.com/api/v1/trades/$id/");

// DELETE REQUEST TYPE
curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "DELETE");

// apply headers
curl_setopt($curl, CURLOPT_HTTPHEADER, ["Authorization: Basic $auth", "Tonce: $tonce"]);

// define how we want our response.
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($curl, CURLOPT_HEADER, 0);

// gimme gimme!
$body = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

echo "STATUS: $status\nBODY:\n$body";
```

> Example Response - 200 OK

```json
{
  "id":1234
}
```

### HTTP REQUEST

`DELETE /api/v1/trades/:id/`

### URL PARAMETERS

Parameter | Type | Required | Description
--------- | ----------- |----- | ------
id        | string | yes | The ID of the trade to liquidate/close

### RESPONSE

Returns a identity object indicating the ID of the liquidated position.

### ERRORS

Response | Description
-------- | -----------
404 Not Found | The trade you requested to delete does not exist.
409 Conflict  | The trade is in an invalid state (possibly still pending)
500 Internal Server Error | The system encountered an error
504 Gateway Timeout | The exchange encountered an issue and could not complete the request.

# Price Data

## Exchange Pricing

For the lowest latency, we recommend using price data direct from the exchanges we connect with. This will allow for better pricing data to be fed directly into your trading scripts. Magnr's position execution policy is at-best market price. Frequent price updates will result in execution prices closer to market prices.

### Bitfinex

- Bitfinex: [https://api.bitfinex.com/v1/pubticker/BTCUSD](https://api.bitfinex.com/v1/pubticker/BTCUSD)

<a href='http://docs.bitfinex.com/#rest'>Bitfinex Documentation</a>

### Bitstamp

- Bitstamp: [https://www.bitstamp.net/api/ticker/](https://www.bitstamp.net/api/ticker/)

<a href='https://www.bitstamp.net/api/'>Bitstamp Documentation</a>

### ItBit

- ItBit: [https://api.itbit.com/v1/markets/XBTUSD/ticker](https://api.itbit.com/v1/markets/XBTUSD/ticker)

<a href='https://api.itbit.com/docs'>ItBit Documentation</a>
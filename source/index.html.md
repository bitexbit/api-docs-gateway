---
title: API Docs Gateway | bitextbit

language_tabs:
  - python: Python
  - javascript: Node.js

toc_footers:
  - <a href='https://www.bitexbit.com/auth/login'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

# includes:
#   - errors

code_clipboard: true

search: true
---


# Introduction


Welcome to the bitexbit API docs!

Service [bitexbit.com](https://www.bitexbit.com) provides open API for trading operations and broadcasting of all trading events. 



# HTTP API Gateway


HTTP API endpoint available on [https://www.bitexbit.com/api/gateway](https://www.bitexbit.com/api/gateway/).

Some methods requires authorization [read below](#authorization).

<aside class="success">
We have limit in 2 calls per second from single account to authorization required methods and 100 calls per secong from single IP address to public methods.
</aside>

All HTTP methods accept `JSON` formats of requests and responses if it not specified by headers.  

## Authorization

> To generate auth headers, use this code:

```python
import hmac
from time import time
from urllib.parse import urlencode

def get_auth_headers(self, data = {}):
        msg = 'your key' + urlencode(sorted(data.items(), key=lambda val: val[0]))

        sign = hmac.new('your keys secret'.encode(), msg.encode(), digestmod='sha256').hexdigest()

        return {
            'X-API-Key': 'your key',
            'X-Signature': sign,
            'X-Nonce': str(int(time() * 1000)),
        }
```

```javascript
const hmacSha256 = require('crypto-js/hmac-sha256'); // sha256 hash. or use you favorite u like
const request = require('request'); // for http requests. or use you favorite u like

const BASE_URL = 'https://www.bitexbit.com/api/v1';
const API_KEY = '0000-0000...';
const SECRET = 'Z%2........';

//Serialize for singing only. Can be used in request body if u like urlencoded form format instead of json
function serializePayload(payload) {
  return Object
    .keys(payload) // get keys of payload object
    .sort() // sort keys
    .map((key) => key + "=" + encodeURIComponent(payload[key])) // each value should be url encoded. the most sensitive part for sign checking
    .join('&'); // to sting, separate with ampersand
}

// Generates auth headers
function getAuthHeaders(payload = {}) {
  // get SHA256 of <API_KEY><sorted urlencoded payload string><SECRET>
  const sign = hmacSha256(API_KEY + serializePayload(payload), SECRET).toString();

  return {
    'X-API-Key': API_KEY,
    'X-Signature': sign,
    'X-Nonce': Date.now()
  };
}
```

In order for access to private API methods, generate authorization keys [in profile settings](https://www.bitexbit.com/profile/api).

- Make `nonce` - a number that must be unique and must increase with each call to the API
- Make `payload` - string, concatenated with `path`, `nonce`, and `body`. Where
  `path` - relative path without query params (example: `/api/v2/private/balance`) and
  `body` - for `POST` request - json string from request data, for `GET` request empty string.
- Make `signature` from `payload` and `secret`, using HMAC Authentication with SHA-256.

### All request to these methods must contain the following headers:

* `X-API-Key` - your key.
* `X-Signature` - query's POST data, sorted by keys and signed by your key's "secret" according to the HMAC-SHA256 method.
* `X-Nonce` - integer value, must be greater then nonce in previous api call.

# HTTP API (v1)

Read more about API v1 [here](https://bitexbit.github.io/api-docs-v1)

# HTTP API (v2)

Read more about API v2 [here](https://bitexbit.github.io/api-docs-v2)

# Errors

We are using standard HTTP statuses in responses.

| Error Code | Meaning                                                                                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400        | If response has http status 400 that means that client should change some input parameters to make it success. What exactly wrong returned in response body.         |
| 401        | In case when HTTP endpoint require authentication but provided authentication headers not valid - response with status 401 will be returned and the details in body. |
| 403        | You do not have access to do the operation. For example when you try to cancel orders that was created by other user.                                                |
| 429        | We have different rate limits to call API methods. If you reached this - response with status 429 will be returned.                                                  |
| 500        | Nobody is safe to make mistakes. This is the case when error happened on serer side. No details but we are see it.                                                   |
| 502        | Looks like we under high load. Try your request later.                                                                                                               |

# Payment Methods

## Methods

```python
import requests

response = requests.get(
                        'https://www.bitexbit.com/api/gateway​/payment-methods',
                        headers = get_auth_headers({})
                        )
```

```javascript
const request = require('request');

request.get({
              url: 'https://www.bitexbit.com/​api​/gateway​/payment-methods',
              headers: getAuthHeaders()
            }, 
            function (error, response, body) {
            // process response    
});
```

> Sample output

```json
[
  {
    "ticker": "BTC",
    "network": "bitcoin",
    "min_deposit": "0.00100000",
    "min_withdraw": "0.00060000",
    "max_withdraw": "100.00000000",
    "withdraw_fee_amount": "0.00040000",
    "withdraw_fee_rate": "0.00000000",
    "withdraw_fee_currency": "BTC",
    "is_deposit_enabled": true,
    "is_withdraw_enabled": true
  },
  {
    "ticker": "USDT",
    "network": "tron",
    "min_deposit": "0.50000000",
    "min_withdraw": "10.00000000",
    "max_withdraw": "50000.00000000",
    "withdraw_fee_amount": "2.00000000",
    "withdraw_fee_rate": "0.00000000",
    "withdraw_fee_currency": "USDT",
    "is_deposit_enabled": true,
    "is_withdraw_enabled": true
  },
]
```

<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`GET https://www.bitexbit.com/api/gateway​/payment-methods`

Returns information about all available payment methods.

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
ordering  | default | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting


# Wallets

## Wallet Create

```python
import requests

body =  {
  "ticker": "ETH",
  "network": "ethereum",
  "convert_ticker": "USDT",
  "convert_limit": 10000,
  "convert_percent": 2
}

response = requests.post(
                        'https://www.bitexbit.com/​api​/gateway​/wallet​/create',
                        data = body, 
                        headers = get_auth_headers(body)
                        )
```

```javascript
const request = require('request');

const body =  {
  "ticker": "ETH",
  "network": "ethereum",
  "convert_ticker": "USDT",
  "convert_limit": 10000,
  "convert_percent": 2
}

request.post({
              url:'https://www.bitexbit.com/​api​/gateway​/wallet​/create',
              form:body,
              headers:getAuthHeaders(body)
              }, 
              function (error, response, body) {
    // process response    
});
```

> Sample output

```json
{
  "address": "0x35f03647b25bd3e3221608dc05f079c0ce3ba3b1",
  "slug": "659b46e76ae05ce03b19ec39f64dbea6f29e",
  "ticker": "ETH",
  "network": "ethereum",
  "callback_url": null,
  "convert_ticker": "USDT",
  "convert_limit": "10000.00000000",
  "convert_percent": "2.00000000",
  "created_at": "2021-05-21T12:10:54.552073Z"
}
```


<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`POST https://www.bitexbit.com/api/gateway​/wallet/create`

Returns information about created wallets.


### POST Params

Parameter       | Default | Required |Description
--------------- | ------- | ---------|-----------
ticker          |         | `true`   | Ticker.
network         |         | `true`   | Ticker network .
callback_url    |         | `false`  | Callback url.
convert_ticker  |         | `false`  | Will create order convert_ticker/ticker OR ticker/convert_ticker. If pair doesn't exist will return error.
convert_limit   |         | `false`  | Order max amount. If deposit 100 and convert_limit = 20. Order amount set to 20.
convert_percent | 5       | `false`  | Order price relative to market. If market price=100 and convert_percent=10, order price for sell=90, for buy=110. Can be negative


## Wallets List

```python
import requests

response = requests.get('https://www.bitexbit.com/​api​/gateway​/wallets',headers = get_auth_headers({}))
```

```javascript
const request = require('request');

request.get({
              url: 'https://www.bitexbit.com/​api​/gateway​/wallets',
              headers: getAuthHeaders()
            }, 
            function (error, response, body) {
            // process response    
});
```

> Sample output

```json
{
  "count": 2,
  "next":" https://www.bitexbit.com/api/gateway/wallets?page=2",
  "previous": null,
  "results": [
    {
      "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
      "slug": "529e67dd83f210e727702d15ffbde9548917",
      "ticker": "USDT",
      "network": "tron",
      "callback_url": null,
      "convert_ticker": "ETH",
      "convert_limit": "30.00000000",
      "convert_percent": "0.10000000",
      "created_at": "2021-05-20T14:42:15.210193Z"
    },
    {
      "address": "TKpR2mrEj2Stnm4QLNMf97zWvgmSGvuLMU",
      "slug": "fd00bbe3f2b24da40b62cf6675f03f656e3a",
      "ticker": "USDT",
      "network": "tron",
      "callback_url": null,
      "convert_ticker": null,
      "convert_limit": null,
      "convert_percent": null,
      "created_at": "2021-05-20T14:06:35.498294Z"
    }
  ]
}
```

<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`GET https://www.bitexbit.com/api/gateway​/wallets`

Returns information about all created wallets.


### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page      | 1       | A page number within the paginated result set.
limit     | all     | Limiting results.
ordering  |default  | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting



## Wallet History

```python
import requests

response = requests.get('https://www.bitexbit.com/​api​/gateway​/wallet​/history', headers = get_auth_headers(body))
```

```javascript
const request = require('request');

request.get({
              url:'https://www.bitexbit.com/​api​/gateway​/wallet​/history',
              headers:getAuthHeaders(body)
              }, 
              function (error, response, body) {
    // process response    
});
```

> Sample output

```json
 {
  "count": 123,
  "next":" https://www.bitexbit.com/api/gateway/wallet/history?page=2",
  "previous": null,
  "results": [
    {
      "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
      "wallet_slug": "529e67dd83f210e727702d15ffbde9548917",
      "date": "2021-05-20T14:56:10.196967Z",
      "amount": "5.00000000",
      "fee": "0.00000000",
      "ticker": "USDT",
      "network": "tron",
      "status": "5",
      "txid": "b104deee6cdbd07e4d97f16b88ec21014059d5837e2428a7f0d48e58e80d5d81",
      "actual_confirmations": null,
      "required_confirmations": 40,
      "aml_success": true,
      "aml_risk": 0,
      "aml_url": null,
      "created_at": "2021-05-20T14:56:10.196967Z"
    }
  ]
}
```

<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`GET https://www.bitexbit.com/api/gateway​/wallet/history`

Returns information about all wallets deposits.


### Query Parameters

Parameter       | Description
--------------- | -----------
ticker          | Ticker.
network         | Ticker network .
page            | A page number within the paginated result set.
status          | 
txid            | 
address         | 
payment_id      |
memo            |
destination_tag |
ordering        | Which field to use when ordering the results.
limit           | Limiting results.


## Deposit Statuses

Code   |   Status   |Description
-------|------------|----------
1      |  PENDING   | when waiting for confirmation
5      |  CONFIRMED | when on balance
6      |  REJECTED  |when impossible to approve



# Withdraw

## Withdraw Create

```python
import requests

body =  {
  "ticker": "USDT",
  "network": "tron",
  "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
  "payment_id": "",
  "memo": "",
  "comment": "Call after success",
  "destination_tag": "",
  "amount": 22,
  "client_order_id": "order_123",
  "fee_payer": 1,
  "auto_buy_ticker": "ETH",
  "auto_buy_percent": 10
}

response = requests.post(
                        'https://www.bitexbit.com/​api​/gateway​/withdraw/create',
                        data = body, 
                        headers = get_auth_headers(body)
                        )
```

```javascript
const request = require('request');

const body =  {
  "ticker": "USDT",
  "network": "tron",
  "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
  "payment_id": "",
  "memo": "",
  "comment": "Call after success",
  "destination_tag": "",
  "amount": 22,
  "client_order_id": "order_123",
  "fee_payer": 1,
  "auto_buy_ticker": "ETH",
  "auto_buy_percent": 10
}

request.post({
              url:'https://www.bitexbit.com/​api​/gateway​/withdraw/create',
              form:body,
              headers:getAuthHeaders(body)
              }, 
              function (error, response, body) {
    // process response    
});
```

> Sample output

```json
{
  "id": 0,
  "ticker": "USDT",
  "network": "tron",
  "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
  "payment_id": null,
  "memo": null,
  "comment": "Call after success",
  "destination_tag":null,
  "amount": 22,
  "callback_url": null,
  "client_order_id": "order_123",
  "fee_payer": 1,
  "status": 0,
  "auto_buy": {
    "symbol": "ETH",
    "amount": 22,
    "price": 3300,
    "side": 1,
    "state": "string"
  },
  "txid": "string",
  "aml_success": "string",
  "aml_risk": "string",
  "aml_url": "string",
  "created_at": "2021-05-21T13:28:32.095Z"
}
```


<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`POST https://www.bitexbit.com/api/gateway​/withdraw/create`

Returns information about created withdraw.


### POST Params

Parameter       | Default | Required |Description
--------------- | ------- | ---------|-----------
ticker          |         | `true`   | Ticker.
network         |         | `true`   | Ticker network .
address         |         | `true`   | Address to withdraw.
amount          |         | `true`   | Amount to withdraw.
callback_url    |         | `false`  | Callback url.
comment         |         | `false`  | Comment.
client_order_id |         | `false`  | Id from your exchange
fee_payer       |  1      | `false`  | `1` charge fee from account balance. `2` charge fee from withdraw amount. If fee currency different than the withdraw `ticker`, then only `1` option is possible.
auto_buy_ticker |         | `false`  | Will create order auto_buy_ticker/ticker OR ticker/auto_buy_ticker. If pair doesn't exist will return error.
auto_buy_percent | 5      | `false`  | Order price relative to market. If market price=100 and convert_percent=10, order price for sell=90, for buy=110. Can be negative


## Withdraw History


```python
import requests

response = requests.post(
                        'https://www.bitexbit.com/​api​/gateway​/withdraw/history',
                        headers = get_auth_headers(body)
                        )
```

```javascript
const request = require('request');

request.get({
              url:'https://www.bitexbit.com/​api​/gateway​/withdraw/create',
              headers:getAuthHeaders(body)
              }, 
              function (error, response, body) {
    // process response    
});
```

> Sample output

```json
 {
  "count": 123,
  "next":" https://www.bitexbit.com/api/gateway/withdraw/history?page=2",
  "previous": null,
  "results": [
    {
      "id": 0,
      "ticker": "USDT",
      "network": "tron",
      "address": "TR4d6N2HoC3Annqt2JK6HhQWuNARMNx5pW",
      "payment_id": null,
      "memo": null,
      "comment": "Call after success",
      "destination_tag":null,
      "amount": 22,
      "callback_url": null,
      "client_order_id": "order_123",
      "fee_payer": 1,
      "status": 0,
      "auto_buy": {
        "symbol": "ETH",
        "amount": 22,
        "price": 3300,
        "side": 1,
        "state": "string"
      },
      "txid": "string",
      "aml_success": "string",
      "aml_risk": "string",
      "aml_url": "string",
      "created_at": "2021-05-21T13:28:32.095Z"
    }
  ]
```

<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`GET https://www.bitexbit.com/api/gateway​/withdraw/history`

Returns information about all created withdraw.


### Query Parameters

Parameter       | Description
--------------- | -----------
ticker          | Ticker.
network         | Ticker network .
page            | A page number within the paginated result set.
status          | 
txid            | 
address         | 
ordering        | Which field to use when ordering the results.
limit           | Limiting results.


## Withdraw Statuses

Code   |  Status    |Description
-------|------------|----------
10     |  NEW       | Created new, need to be confirmed
20     |  CONFIRMED | Confirmed by user, need to be verified
23     |  VERIFIED  | Verified by automatic system
25     | MODERATION | Moderation by operator
26     |  APPROVED  | Approved for send
27     |  SUSPENDED | Suspended by operator
30     |  DONE      | Done
40     |  REFUSED   | Refused by operator
50     |  CANCELLED | Cancelled by user

# AML

## Check AML

`Comming soon`

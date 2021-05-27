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

All methods requires authorization [read below](#authorization).

All HTTP methods accept `JSON` formats of requests and responses if it not specified by headers.  

## Authorization

> To generate auth headers, use this code:

```python
import hmac
from time import time
import json
import requests

BASE_URL = "https://www.bitexbit.com"
API_KEY = "0000-0000..."
SECRET = "Z%2........"


def get_signature(key: str, path: str, timeout: int, body: str = ''):
    """
    Returns signature of the request that signed by secret key
    :param key: public api key
    :param path: ape request path
    :param timeout: unique nonce
    :param body: serialized body of the future request
    :return: sha56 hash as hexadecimal string
    """

    # concatenate path with timeout and body of future request
    payload = f'{key}{path}{timeout}{body}'
    # return hex string of sha256 of payload, signed by secret key. secret key already is bytes array
    return hmac.new(SECRET.encode(), payload.encode(), digestmod='sha256').hexdigest()


def get(path: str, **params):
    timeout = int(time() * 1000) + 500  # 500 is timeout for execute
    # get signature hash
    signature = get_signature(API_KEY, path, timeout, '')

    # create headers dict with standard and custom API headers
    headers = {
        'Content-Type': 'application/json; charset=utf-8',
        'Accept': 'application/json',
        'X-Key': API_KEY,
        'X-Timeout': str(timeout),
        'X-Sign': signature,
    }

    response = requests.get(
        url=f'{BASE_URL}{path}',
        headers=headers,
        params=params,
    )

    return response.json()


def post(path: str, payload):
    timeout = int(time() * 1000) + 500  # 500 is timeout for execute

    # dump payload object into JSON string
    body = json.dumps(payload)

    # get signature hash
    signature = get_signature(API_KEY, path, timeout, body)

    # create headers dict with standard and custom API headers
    headers = {
        'Content-Type': 'application/json; charset=utf-8',
        'Accept': 'application/json',
        'X-Key': API_KEY,
        'X-Timeout': str(timeout),
        'X-Sign': signature,
    }

    # execute POST request on server using prepared data
    response = requests.post(
        url=f'{BASE_URL}{path}',
        headers=headers,
        data=body.encode(),
    )

    return response.json()

```

```javascript
const sha256 = require("js-sha256"); // sha256 hash. or use you favorite u like
const axios = require("axios"); // for http requests. or use you favorite u like

const BASE_URL = "https://www.bitexbit.com";
const API_KEY = "0000-0000...";
const SECRET = "Z%2........";

const client = axios.create({
  baseURL: BASE_URL,
  timeout: 10000,
  headers: {
    "Content-Type": "application/json; charset=utf-8",
    Accept: "application/json",
  },
  responseType: "json",
});

const getSignature = (path, timeout, body = "") => {
  const payload = API_KEY + path + timeout + body;
  const hash = sha256.hmac.create(SECRET);
  hash.update(payload);
  return hash.hex();
};

const get = (path, params) => {
  const timeout = Date.now() + 500; // 500 is timeout for execute

  const signature = getSignature(path, timeout, "");

  const config = {
    params,
    headers: {
      "X-Key": API_KEY,
      "X-Timeout": timeout,
      "X-Sign": signature,
    },
  };

  return client.get(path, config);
};

const post = (path, body = {}) => {
  const timeout = Date.now() + 500; // 500 is timeout for execute

  const signature = getSignature(path, timeout, JSON.stringify(body));

  const config = {
    headers: {
      "X-Key": API_KEY,
      "X-Timeout": timeout,
      "X-Sign": signature,
    },
  };

  return client.post(path, body, config);
};
```

In order for access to private API methods, generate authorization keys [in profile settings](https://www.bitexbit.com/profile/api).

- Make `timeout` - int, timeout to execute call
- Make `payload` - string, concatenated with `api key`,`path`, `timeout`, and `body`. Where `path` - relative path without query params (example: `/api/gateway/payment-methods`) and
  `body` - for `POST` request - json string from request data, for `GET` request empty string.
- Make `signature` from `payload` and `secret`, using HMAC Authentication with SHA-256.

### All request to these methods must contain the following headers:

* `X-Key` - your key.
* `X-Sign` - query's POST data, signed by your `secret` according to the HMAC-SHA256 method.
* `X-Timeout` - integer value, timeout to execute call.

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

get("/api/gateway​/payment-methods")

```

```javascript

get("/api/gateway​/payment-methods")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
  });

```

> Sample output

```json
[
  {
    "ticker": "BTC",
    "network": "bitcoin",
    "min_deposit": "0.00100000",
    "max_deposit": "10000000000.00000000",
    "deposit_fee_amount": "0.00000000",
    "deposit_fee_rate": "0.00000000",
    "min_withdraw": "0.00060000",
    "max_withdraw": "100.00000000",
    "withdraw_fee_amount": "0.00040000",
    "withdraw_fee_rate": "0.00000000",
    "withdraw_fee_currency": "BTC",
    "is_fiat": false,
    "is_deposit_enabled": true,
    "is_withdraw_enabled": true
  },
  {
    "ticker": "UAH",
    "network": "wiretransfer",
    "min_deposit": "25.20000000",
    "max_deposit": "100000.00000000",
    "deposit_fee_amount": "5.00000000",
    "deposit_fee_rate": "0.01000000",
    "min_withdraw": "100.00000000",
    "max_withdraw": "10000.00000000",
    "withdraw_fee_amount": "5.00000000",
    "withdraw_fee_rate": "0.01500000",
    "withdraw_fee_currency": "UAH",
    "is_fiat": true,
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


# Crypto Wallets

## Wallet Create

```python

body =  {
  "ticker": "ETH",
  "network": "ethereum",
  "convert_ticker": "USDT",
  "convert_limit": 10000,
  "convert_percent": 2
}

post("/​api​/gateway​/wallet​/create", body)

```

```javascript

const body =  {
  "ticker": "ETH",
  "network": "ethereum",
  "convert_ticker": "USDT",
  "convert_limit": 10000,
  "convert_percent": 2
}

post("/​api​/gateway​/wallet​/create", body)
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
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
callback_url    |         | `false`  | Callback url. `POST` method. Must return `200` status.
convert_ticker  |         | `false`  | Will create order convert_ticker/ticker OR ticker/convert_ticker. If pair doesn't exist will return error.
convert_limit   |         | `false`  | Order max amount. If deposit 100 and convert_limit = 20. Order amount set to 20.
convert_percent | 5       | `false`  | Order price relative to market. If market price=100 and convert_percent=10, order price for sell=90, for buy=110. Can be negative


## Wallets List

```python

get("/​api​/gateway​/wallets")

```

```javascript

get("/​api​/gateway​/wallets")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
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

get("https://www.bitexbit.com/​api​/gateway​/wallet​/history")

```

```javascript

get("/​api​/gateway​/wallet​/history")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
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



# Fiat Invoice

## Create invoice

```python

body =  {
  "ticker": "USDT",
  "network": "payeer",
  "amount" : 120,
  "payment_url": "https://exchange.com/success",
  "cancel_url":  "https://exchange.com/cancel",
  "callback_url" : "https://exchange.com/api/callback",
  "convert_ticker": "ETH",
  "convert_limit": 10000,
  "convert_percent": 2,
  "client_ip": "192.168.0.1"
}

post("/​api​/gateway​/invoice/create", body)

```

```javascript

const body =  {
  "ticker": "USDT",
  "network": "payeer",
  "amount" : 120,
  "payment_url": "https://exchange.com/success",
  "cancel_url":  "https://exchange.com/cancel",
  "callback_url" : "https://exchange.com/api/callback",
  "convert_ticker": "ETH",
  "convert_limit": 10000,
  "convert_percent": 2,
  "client_ip": "192.168.0.1"
}

post("/​api​/gateway​/invoice/create", body)
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
  });

```

> Sample output

```json
{
  "slug": "659b46e33ae05ce03b19ec39f64dbea6f29e",
  "ticker": "USDT",
  "network": "payeer",
  "amount": 120,
  "proceed_url": "payment-url",
  "payment_url": "https://exchange.com/success",
  "convert_ticker": "ETH",
  "convert_limit": "10000.00000000",
  "convert_percent": "2.00000000",
  "created_at": "2021-05-21T12:10:54.552073Z",

}
```


<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`POST https://www.bitexbit.com/api/gateway​/invoice/create`

Returns information about invoice and payment url.


### POST Params

Parameter       | Default | Required |Description
--------------- | ------- | ---------|-----------
ticker          |         | `true`   | Ticker.
network         |         | `true`   | Ticker network.
amount          |         | `true`   | Amount to deposit.
callback_url    |         | `false`  | Callback url. `POST` method. Must return `200` status.
payment_url     |         | `false`  | Redirect url on success.
cancel_url      |         | `false`  | Redirect url on cancel.
convert_ticker  |         | `false`  | Will create order convert_ticker/ticker OR ticker/convert_ticker. If pair doesn't exist will return error.
client_ip       |         | `true`   | User ip. Only from this ip deposit is possible. 
convert_limit   |         | `false`  | Order max amount. If deposit 100 and convert_limit = 20. Order amount set to 20.
convert_percent | 5       | `false`  | Order price relative to market. If market price=100 and convert_percent=10, order price for sell=90, for buy=110. Can be negative


## Invoices List

```python

get("/​api​/gateway​/invoices")

```

```javascript

get("/​api​/gateway​/invoices")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
  });


```

> Sample output

```json
{
  "count": 2,
  "next":" https://www.bitexbit.com/api/gateway/invoices?page=2",
  "previous": null,
  "results": [
    {
      "slug": "659b46e33ae05ce03b19ec39f64dbea6f29e",
      "ticker": "USDT",
      "network": "payeer",
      "amount": 120,
      "callback_url" : "https://exchange.com/api/callback",
      "convert_ticker": "ETH",
      "convert_limit": "10000.00000000",
      "convert_percent": "2.00000000",
      "created_at": "2021-05-21T12:10:54.552073Z"
    }
  ]
}
```

<aside class="warning">
This method requires authorization.
</aside>

### HTTP Request

`GET https://www.bitexbit.com/api/gateway​/invoices`

Returns information about all created invoices.


### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page      | 1       | A page number within the paginated result set.
limit     | all     | Limiting results.
ordering  |default  | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting



## Invoices History

```python

get("/​api​/gateway​/invoice/history")

```

```javascript

get("/​api​/gateway​/invoice/history")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
  });

```

> Sample output

```json
 {
  "count": 123,
  "next":" https://www.bitexbit.com/api/gateway/invoice/history?page=2",
  "previous": null,
  "results": [
      {
        "date": "2021-05-25T14:54:32.826135Z",
        "amount": "292.00000000",
        "fee": "8.00000000",
        "ticker": "UAH",
        "network": "wiretransfer",
        "status": "5",
        "txid": "361865033",
        "convert": {
          "symbol": "USDT_UAH",
          "amount": "9.04537866",
          "price": "28.52850000",
          "side": "BUY",
          "state": "partial-filled"
        },
        "created_at": "2021-05-25T14:54:32.826135Z"
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





# Withdraw

## Withdraw Create

```python

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

post("/​api​/gateway​/withdraw/create", body)


```

```javascript

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


post("/​api​/gateway​/withdraw/create", body)
.then(resp => {
  //process response
})
.catch(error => {
  // handle error
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
  "amount": "22.0000000",
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
address         |         |          | Address to withdraw. Required if Crypto.
card            |         |          | Card to withdraw. Required if Fiat.
amount          |         | `true`   | Amount to withdraw.
callback_url    |         | `false`  | Callback url. `POST` method. Must return `200` status.
comment         |         | `false`  | Comment.
client_order_id |         | `false`  | Id from your exchange
fee_payer       |  1      | `false`  | `1` charge fee from account balance. `2` charge fee from withdraw amount. If fee currency different than the withdraw `ticker`, then only `1` option is possible.
auto_buy_ticker |         | `false`  | Will create order auto_buy_ticker/ticker OR ticker/auto_buy_ticker. If pair doesn't exist will return error.
auto_buy_percent | 5      | `false`  | Order price relative to market. If market price=100 and convert_percent=10, order price for sell=90, for buy=110. Can be negative


## Withdraw History


```python

get("/​api​/gateway​/withdraw/history")

```

```javascript

get("/​api​/gateway​/withdraw/history")
  .then(resp => {
    //process response
  })
  .catch(error => {
    // handle error
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





# Statuses

## Deposit Statuses

Code   |   Status   |Description
-------|------------|----------
1      |  PENDING   | When waiting for confirmation
5      |  CONFIRMED | When on balance
6      |  REJECTED  | When impossible to approve

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

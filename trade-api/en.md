# Payeer Trade API (en)

API URL: https://payeer.com/api/trade/

## Public API 

* [Connection test](#connection-test-time) [/time]
* [Pairs and limits](#pairs-and-limits-info) [/info]
    * [All pairs](#all-pairs-get)
    * [Selection](#selection-post)
* [Price statistics](#price-statistics-ticker) [/ticker]
    * [All pairs](#all-pairs-get-1)
    * [Selection](#selection-post-1)
* [Orders](#orders-orders) [/orders]
* [Trades](#trades-trades) [/trades]

## Private API 

* [Balance](#balance-account) [/account]
* [New order](#new-order-order_create) [/order_create]
    * [Limit](#limit-post)
    * [Market](#market-post)
    * [Stop-limit](#stop-limit-post)
* [Order status](#order-status-order_status) [/order_status]
* [Cancel order](#cancel-order-order_cancel) [/order_cancel]
* [Cancel orders](#cancel-orders-orders_cancel) [/orders_cancel]
    * [Cancellation with pairs and directions](#cancellation-with-pairs-and-directions-post)
    * [Cancellation with pairs](#cancellation-with-pairs-post)
    * [Cancel all orders](#cancel-all-orders-post)
* [My orders](#my-orders-my_orders) [/my_orders]
    * [Without filtering](#without-filtering-post)
    * [With filtering](#with-filtering-post)
* [My history](#my-history-my_history) [/my_history] 
    * [Without filtering](#without-filtering-post-1)
    * [Paginated navigation](#paginated-navigation-post)
    * [With filtering](#with-filtering-post-1)
* [My trades](#my-trades-my_trades) [/my_trades]
    * [Without filtering](#without-filtering-post-2)
    * [Paginated navigation](#paginated-navigation-post-1)
    * [With filtering](#with-filtering-post-2)

<br>
API Settings: https://payeer.com/en/account/api/?tab=trade
<br><br>

To access the Private API, you should pass the following keys in the headers:
* API-ID - ID of API-user
* API-SIGN - signature of request

Also in the request body, the required parameter is <code>ts</code>, the value of which is equal to the current timestamp <b>in milliseconds</b>.<br>
<br>
The request will be processed if it reaches our server within 60 seconds to avoid significant network delays.<br>
<br>
The maximum difference between the <code>ts</code> parameter and the real time should not exceed 1 second, do not forget to set up time synchronization on your server.<br>
<br>
The request body is passed as a json string.<br>
<br><br>
<b>Signature generation:</b><br>
<br>
<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$method = 'account';

$req = json_encode(array(
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);
```

<code>apiSecret</code> - the secret key of the API user.<br>
<br>
The signature is created using the HMAC-SHA256 method.

<br>
<b>Python</b><br>

```python
import hashlib
import hmac
import time
import json
from urllib2 import Request, urlopen

ts = int(round(time.time()*1000))

method = 'account'
req = json.dumps({
    'ts': ts,
})
print req

H = hmac.new('api_secret_key', digestmod=hashlib.sha256)
H.update(method + req)
sign = H.hexdigest()
print sign

headers = {
    'Content-Type': 'application/json',
    'API-ID': 'bd443f00-092c-4436-92a4-a704ef679e24',
    'API-SIGN': sign
}

request = Request('https://payeer.com/api/trade/' + method, data=req, headers=headers)

response_body = urlopen(request).read()
print response_body
```

<br>
<b>Node.js</b><br>

```node.js
var crypto = require('crypto')
	request = require('request');

apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
apiSecret = 'api_secret_key';

method = 'account';

ts = Math.floor(new Date().getTime());

req = JSON.stringify({
    'ts': ts,
});

sign = crypto.createHmac('sha256', apiSecret).update(method+req).digest('hex');

request({
  method: 'POST',
  url: 'https://payeer.com/api/trade/' + method,
  headers: {
    'Content-Type': 'application/json',
    'API-ID': apiId,
    'API-SIGN': sign
  },
  body: req
}, function (error, response, body) {
	console.log(body);
});
```

## Errors

List of possible errors.<br>

<table>
    <tr>
        <th>Error</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>INVALID_SIGNATURE</b></td>
        <td>
            Reasons:<br>
            - invalid signature API-SIGN<br>
            - invalid API-ID<br>
            - API user is blocked (can be unblocked in settings)<br>
        </td>
    </tr>
    <tr>
        <td><b>INVALID_IP_ADDRESS</b></td>
        <td>The IP address <code>ip</code> of the request source does not match the one specified in the API settings</td>
    </tr>
    <tr>
        <td><b>LIMIT_EXCEEDED</b></td>
        <td>Exceeding the limits (number of requests/weights/orders), more details are specified in the <code>desc</code> parameter</td>
    </tr>
    <tr>
        <td><b>INVALID_TIMESTAMP</b></td>
        <td>
            the <code>ts</code> parameter is invalid:<br>
            - the request went to the API server for more than 60 seconds<br>
            - wrong time on your server, set up synchronization
        </td>
    </tr>
    <tr>
        <td><b>ACCESS_DENIED</b></td>
        <td>Access to the API/order is prohibited. Check the <code>order_id</code></td>
    </tr>
    <tr>
        <td><b>INVALID_PARAMETER</b></td>
        <td>parameter <code>parameter</code> is invalid</td>
    </tr>
    <tr>
        <td><b>PARAMETER_EMPTY</b></td>
        <td>parameter <code>parameter</code> must be filled</td>
    </tr>
    <tr>
        <td><b>INVALID_STATUS_FOR_REFUND</b></td>
        <td>the status <code>status</code> of the order does not allow to make a refund (the order has already been returned or canceled)</td>
    </tr>
    <tr>
        <td><b>REFUND_LIMIT</b></td>
        <td>the order can be canceled at least 1 minute after creation</td>
    </tr>
    <tr>
        <td><b>UNKNOWN_ERROR</b></td>
        <td>Unknown error on the exchange. Trading suspended for review. Try again in 15 minutes.</td>
    </tr>
    <tr>
        <td><b>INVALID_DATE_RANGE</b></td>
        <td>Invalid date range for filtering (period must not exceed 32 days)</td>
    </tr>
    <tr>
        <td><b>INSUFFICIENT_FUNDS</b></td>
        <td>insufficient funds to create an order (<code>max_amount</code>, <code>max_value</code>)</td>
    </tr>
    <tr>
        <td><b>INSUFFICIENT_VOLUME</b></td>
        <td>not enough volume to create an order (<code>max_amount</code>, <code>max_value</code>)</td>
    </tr>
    <tr>
        <td><b>INCORRECT_PRICE</b></td>
        <td>the price is out of range (<code>min_price</code>, <code>max_price</code>)</td>
    </tr>
    <tr>
        <td><b>MIN_AMOUNT</b></td>
        <td>quantity less than the minimum <code>min_amount</code> for the pair</td>
    </tr>
    <tr>
        <td><b>MIN_VALUE</b></td>
        <td>value is less than the minimum <code>min_value</code> for the pair</td>
    </tr>
</table>         



## Connection test [/time] 

Checking the connection, getting the server time.<br>
<br>
<b>Request Weight:</b> 1<br>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>time</b></td>
        <td>server timestamp</td>
        <td>1644322909335</td>
    </tr>
</table>

### Connection test [GET]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/time");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Response 200 (application/json)

```json
{
  "success": true,
  "time": 1644322909335
}
```


## Pairs and limits [/info]

Getting limits, available pairs and their parameters.<br>
<br>
<b>Request Weight:</b> 1<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>limits</b></td>
        <td>
            list of limits:<br>
            &nbsp;&nbsp;&nbsp;<code>requests</code><br>
            &nbsp;&nbsp;&nbsp;<code>weights</code><br>
            &nbsp;&nbsp;&nbsp;<code>orders</code>
        </td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>interval</b></td>
        <td>interval</td>
        <td>min, hour, day</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>interval_num</b></td>
        <td>
            the value of the interval:<br>
            &nbsp;&nbsp;&nbsp;min, 1: in one minute<br>
            &nbsp;&nbsp;&nbsp;min, 5: in 5 minutes<br>
            &nbsp;&nbsp;&nbsp;hour, 1: in 1 hour
        </td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>limit</b></td>
        <td>
            limit:<br>
            &nbsp;&nbsp;&nbsp;<code>requests</code>, min 1, 600: 600 requests per minute<br>
            &nbsp;&nbsp;&nbsp;<code>weights</code>, min 1, 600: 600 weights per minute<br>
            &nbsp;&nbsp;&nbsp;<code>orders</code>, min 1, 120: 120 orders per minute
        </td>
        <td>600</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>list of pairs</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price_prec</b></td>
        <td>precision <code>price</code> parameter</td>
        <td>2</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_price</b></td>
        <td>minimum price for creating an order</td>
        <td>4375.74</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>max_price</b></td>
        <td>maximum price for creating an order</td>
        <td>83139.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_amount</b></td>
        <td>minimum amount to create an order</td>
        <td>0.0001</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_value</b></td>
        <td>minimum value to create an order</td>
        <td>0.5</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>fee_maker_percent</b></td>
        <td>maker's commission</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>fee_taker_percent</b></td>
        <td>taker's commission</td>
        <td>0.095</td>
    </tr>
</table>
        
### All pairs [GET]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/info");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "limits": {
    "requests": [
      {
        "interval": "min",
        "interval_num": 1,
        "limit": 600
      }
    ]
  },
  "pairs": {
    "BTC_USD": {
      "price_prec": 2,
      "min_price": "4375.74",
      "max_price": "83139.00",
      "min_amount": 0.0001,
      "min_value": 0.5,
      "fee_maker_percent": 0.01,
      "fee_taker_percent": 0.095
    },
    "BTC_RUB": {
      "price_prec": 2,
      "min_price": "326269.32",
      "max_price": "6199117.08",
      "min_amount": 0.0001,
      "min_value": 20,
      "fee_maker_percent": 0.01,
      "fee_taker_percent": 0.095
    },
    "BTC_EUR": {
      "price_prec": 2,
      "min_price": "3798.60",
      "max_price": "72173.39",
      "min_amount": 0.0001,
      "min_value": 0.5,
      "fee_maker_percent": 0.01,
      "fee_taker_percent": 0.095
    }
  }
}
```

### Selection [POST]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/info");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
)));

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json"
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
            
    + Body

```json
{
    "pair": "BTC_USD,BTC_RUB"
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "limits": {
    "requests": [
      {
        "interval": "min",
        "interval_num": 1,
        "limit": 600
      }
    ]
  },
  "pairs": {
    "BTC_USD": {
      "price_prec": 2,
      "min_price": "4332.68",
      "max_price": "82321.00",
      "min_amount": 0.0001,
      "min_value": 0.5,
      "fee_maker_percent": 0.01,
      "fee_taker_percent": 0.095
    },
    "BTC_RUB": {
      "price_prec": 2,
      "min_price": "326269.32",
      "max_price": "6199117.08",
      "min_amount": 0.0001,
      "min_value": 20,
      "fee_maker_percent": 0.01,
      "fee_taker_percent": 0.095
    }
  }
}
```


## Price statistics [/ticker]

Getting price statistics and their changes over the last 24 hours.<br>
<br>
<b>Request Weight:</b> 1<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>list of pairs</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>ask</b></td>
        <td>ask price</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bid</b></td>
        <td>bid price</td>
        <td>43502.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>last</b></td>
        <td>last price</td>
        <td>43501.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min24</b></td>
        <td>minimum price for 24 hours</td>
        <td>43006.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>max24</b></td>
        <td>maximum price for 24 hours</td>
        <td>46000.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>delta</b></td>
        <td>price change in 24 hours as a percentage</td>
        <td>0.70</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>delta_price</b></td>
        <td>price change in 24 hours</td>
        <td>301.00</td>
    </tr>
</table>
        
### All pairs [GET]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/ticker");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "pairs": {
    "BTC_USD": {
      "ask": "43555.55",
      "bid": "43506.01",
      "last": "43555.55",
      "min24": "43006.01",
      "max24": "46000.00",
      "delta": "1.06",
      "delta_price": "455.55"
    },
    "BTC_RUB": {
      "ask": "3258999.99",
      "bid": "3230001.02",
      "last": "3230001.00",
      "min24": "3175000.00",
      "max24": "3390000.00",
      "delta": "1.41",
      "delta_price": "45000.97"
    },
    "BTC_EUR": {
      "ask": "40000.00",
      "bid": "39000.00",
      "last": "40100.00",
      "min24": "37560.00",
      "max24": "40100.00",
      "delta": "6.79",
      "delta_price": "2549.00"
    }
  }
}
```

### Selection [POST]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/ticker");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
)));

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json"
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
            
    + Body
    
```json
{
    "pair": "BTC_USD,BTC_RUB"
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "pairs": {
    "BTC_USD": {
      "ask": "43790.00",
      "bid": "43502.00",
      "last": "43501.00",
      "min24": "43006.01",
      "max24": "46000.00",
      "delta": "0.70",
      "delta_price": "301.00"
    },
    "BTC_RUB": {
      "ask": "3255999.99",
      "bid": "3238600.00",
      "last": "3230001.02",
      "min24": "3175000.00",
      "max24": "3390000.00",
      "delta": "1.57",
      "delta_price": "50001.02"
    }
  }
}
```


## Orders [/orders]

Getting available orders for the specified pairs.<br>
<br>
<b>Request Weight:</b> 1 * count of pairs<br>
<br>
<b>Request Parameters</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>list of pairs</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>ask</b></td>
        <td>ask price</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bid</b></td>
        <td>bid price</td>
        <td>43520.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>asks</b></td>
        <td>list of sell orders</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.00031422</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>13.76</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bids</b></td>
        <td>list of buy orders</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>43520.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.00034788</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>15.13</td>
    </tr>
</table>

### Orders [POST]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/orders");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
)));

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json"
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
            
    + Body

```json
{
    "pair": "BTC_USD,BTC_RUB"
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "pairs": {
    "BTC_USD": {
      "ask": "43790.00",
      "bid": "43520.00",
      "asks": [
        {
          "price": "43790.00",
          "amount": "0.00031422",
          "value": "13.76"
        },
        {
          "price": "43800.00",
          "amount": "0.00125530",
          "value": "54.99"
        }
      ],
      "bids": [
        {
          "price": "43520.00",
          "amount": "0.00034788",
          "value": "15.13"
        },
        {
          "price": "43502.00",
          "amount": "0.04065736",
          "value": "1768.67"
        }
      ]
    },
    "BTC_RUB": {
      "ask": "3255999.99",
      "bid": "3238600.00",
      "asks": [
        {
          "price": "3255999.99",
          "amount": "0.00010000",
          "value": "325.60"
        }
      ],
      "bids": [
        {
          "price": "3238600.00",
          "amount": "0.00022212",
          "value": "719.35"
        },
        {
          "price": "3230001.02",
          "amount": "0.00083607",
          "value": "2700.50"
        }
      ]
    }
  }
}
```


## Trades [/trades]

Getting the history of transactions for the specified pairs.<br>
<br>
<b>Request Weight:</b> 1 * count of pairs<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>list of pairs</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id</td>
        <td>14162760</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp</td>
        <td>1644327656</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.00031422</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>13.76</td>
    </tr>
</table>

### Trades [POST]

<b>PHP</b><br>

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/trades");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
)));

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json"
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
            
    + Body

```json
{
    "pair": "BTC_USD,BTC_RUB"
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "pairs": {
    "BTC_USD": [
      {
        "id": "14162760",
        "date": 1644327656,
        "type": "buy",
        "amount": "0.00060372",
        "price": "44299.99",
        "value": "26.75"
      },
      {
        "id": "14162734",
        "date": 1644327545,
        "type": "sell",
        "amount": "0.00000119",
        "price": "44399.98",
        "value": "0.06"
      }
    ],
    "BTC_RUB": [
      {
        "id": "14162750",
        "date": 1644327611,
        "type": "buy",
        "amount": "0.01152539",
        "price": "3230100.00",
        "value": "37228.16"
      },
      {
        "id": "14162749",
        "date": 1644327610,
        "type": "buy",
        "amount": "0.00988444",
        "price": "3230100.00",
        "value": "31927.73"
      }
    ]
  }
}
```


## Balance [/account]

Getting the user's balance.<br>
<br>
<b>Request Weight:</b> 10<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644327967240</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>balances</b></td>
        <td>list of user's currencies</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>total</b></td>
        <td>total balance</td>
        <td>1598.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>available</b></td>
        <td>available balance</td>
        <td>1548.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>hold</b></td>
        <td>frozen balance</td>
        <td>50</td>
    </tr>
</table>

### Balance [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'account';

$req = json_encode(array(
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

<br><br>
<b>Python</b><br>

```python
import hashlib
import hmac
import time
import json
from urllib2 import Request, urlopen

ts = int(round(time.time()*1000))

method = 'account'
req = json.dumps({
    'ts': ts,
})
print req

H = hmac.new('api_secret_key', digestmod=hashlib.sha256)
H.update(method + req)
sign = H.hexdigest()
print sign

headers = {
    'Content-Type': 'application/json',
    'API-ID': 'bd443f00-092c-4436-92a4-a704ef679e24',
    'API-SIGN': sign
}

request = Request('https://payeer.com/api/trade/' + method, data=req, headers=headers)

response_body = urlopen(request).read()
print response_body
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body

```json
{
    "ts": 1644327967240
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "balances": {
    "USD": {
      "total": 0.92,
      "available": 0.92,
      "hold": 0
    },
    "RUB": {
      "total": 1598.99,
      "available": 1548.99,
      "hold": 50
    },
    "EUR": {
      "total": 2.97,
      "available": 0,
      "hold": 2.97
    },
    "BTC": {
      "total": 0,
      "available": 0,
      "hold": 0
    },
    "ETH": {
      "total": 0,
      "available": 0,
      "hold": 0
    },
    "BCH": {
      "total": 0,
      "available": 0,
      "hold": 0
    },
    "LTC": {
      "total": 0,
      "available": 0,
      "hold": 0
    },
    "DASH": {
      "total": 0,
      "available": 0,
      "hold": 0
    },
    "USDT": {
      "total": 3,
      "available": 0,
      "hold": 3
    },
    "XRP": {
      "total": 0.9981,
      "available": 0.9981,
      "hold": 0
    },
    "DOGE": {
      "total": 94.55549964,
      "available": 94.55549964,
      "hold": 0
    },
    "TRX": {
      "total": 223.681806,
      "available": 208.681806,
      "hold": 15
    }
  }
}
```


## New order [/order_create]

Creating an order of supported types: limit, market, stop limit.<br>
<br>
<b>Request Weight:</b> 5 (10 for a market order)<br>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>id of the created order</td>
        <td>37057767</td>
    </tr>
    <tr>
        <td><b>params</b></td>
        <td>parameters of the created order</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>order type</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>stop price</td>
        <td>0.07800</td>
    </tr>
</table>

### Limit [POST]

<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>order type</td>
        <td>limit</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>amount</b></td>
        <td>amount</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>price</b></td>
        <td>price</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644488888211</td>
    </tr>
</table>

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'order_create';

$req = json_encode(array(
    'pair' => 'TRX_USD',
    'type' => 'limit',
    'action' => 'buy',
    'amount' => '10',
    'price' => '0.08',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "TRX_USD",
    "type": "limit",
    "action": "buy",
    "amount": "10",
    "price": "0.08",
    "ts": 1644488888211
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "order_id": 37054386,
  "params": {
    "pair": "TRX_USD",
    "type": "limit",
    "action": "buy",
    "amount": "10.000000",
    "price": "0.08000",
    "value": "0.80"
  }
}
```

### Market [POST]

<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>order type</td>
        <td>market</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>* amount</td>
        <td>amount</td>
        <td>10</td>
    </tr>
    <tr>
        <td>* value</td>
        <td>value</td>
        <td>10000</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644489441948</td>
    </tr>
</table>

\* - it is possible to specify one of two parameters for creating a market order (<code>amount</code> or <code>value</code>)

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'order_create';

$req = json_encode(array(
    'pair' => 'TRX_USD',
    'type' => 'market',
    'action' => 'buy',
    'amount' => '10',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN:f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "TRX_USD",
    "type": "market",
    "action": "buy",
    "amount": "10",
    "ts": 1644489441948
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "order_id": 37054922,
  "params": {
    "pair": "TRX_USD",
    "type": "market",
    "action": "buy",
    "amount": "10.000000",
    "price": "0.07200",
    "value": "0.72"
  }
}
```

### Stop limit [POST]

<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>order type</td>
        <td>stop_limit</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>amount</b></td>
        <td>amount</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>price</b></td>
        <td>price</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td><b>stop_price</b></td>
        <td>stop price</td>
        <td>0.078</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644492621264</td>
    </tr>
</table>

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'order_create';

$req = json_encode(array(
    'pair' => 'TRX_USD',
    'type' => 'stop_limit',
    'action' => 'buy',
    'amount' => '10',
    'price' => '0.08',
    'stop_price' => '0.078',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "TRX_USD",
    "type": "stop_limit",
    "action": "buy",
    "amount": "10",
    "price": "0.08",
    "stop_price": "0.078",
    "ts": 1644492621264
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "order_id": 37057767,
  "params": {
    "pair": "TRX_USD",
    "type": "stop_limit",
    "action": "buy",
    "amount": "10.000000",
    "price": "0.08000",
    "value": "0.80",
    "stop_price": "0.07800"
  }
}
```

        
## Order status [/order_status]

Getting detailed information about your order by id.<br>
<br>
<b>Request Weight:</b> 5<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>order id</td>
        <td>37054293</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644493185157</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>order</b></td>
        <td>order parameters</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>order id</td>
        <td>37054293</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp of order creation</td>
        <td>1644488809</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>order type</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>order status</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>stop price</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>processed amount</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>remaining amount</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>processed value</td>
        <td>0.72</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>remaining value</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>mark of order creation via API</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>avg_price</b></td>
        <td>average price</td>
        <td>0.07200</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;trades</td>
        <td>list of trades</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>trade id</td>
        <td>14190472</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp</td>
        <td>1644488809</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>status</td>
        <td>success, processing</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>0.07150</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.054165</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>is_maker</b></td>
        <td>you are the maker</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>is_taker</b></td>
        <td>you are the taker</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_transaction_id</td>
        <td>maker's transaction id</td>
        <td>1598693542</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_fee</td>
        <td>maker's commission</td>
        <td>0.03</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t_transaction_id</td>
        <td>taker's transaction id</td>
        <td>1598693543</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t_fee</td>
        <td>taker's commission</td>
        <td>0.36</td>
    </tr>
</table>

### Order status [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'order_status';

$req = json_encode(array(
    'order_id' => '37054293',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "order_id": "37054293",
    "ts": 1644493185157
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "order": {
    "id": "37054293",
    "date": 1644488809,
    "pair": "TRX_USD",
    "action": "buy",
    "type": "limit",
    "status": "success",
    "amount": "10.000000",
    "price": "0.08000",
    "value": "0.80",
    "amount_processed": "10.000000",
    "amount_remaining": "0.000000",
    "value_processed": "0.72",
    "value_remaining": "0.08",
    "avg_price": "0.07200",
    "trades": {
      "14190472": {
        "id": "14190472",
        "date": 1644488809,
        "status": "success",
        "price": "0.07150",
        "amount": "0.054165",
        "value": "0.01",
        "is_maker": false,
        "is_taker": true,
        "t_transaction_id": "1598693542"
      },
      "14190473": {
        "id": "14190473",
        "date": 1644488809,
        "status": "success",
        "price": "0.07165",
        "amount": "7.117935",
        "value": "0.51",
        "is_maker": false,
        "is_taker": true,
        "t_transaction_id": "1598693549"
      },
      "14190474": {
        "id": "14190474",
        "date": 1644488809,
        "status": "success",
        "price": "0.07200",
        "amount": "2.827900",
        "value": "0.20",
        "is_maker": false,
        "is_taker": true,
        "t_transaction_id": "1598693554"
      }
    }
  }
}
```

    
## Cancel order [/order_cancel]

Cancellation of your order by id.<br>
<br>
<b>Request Weight:</b> 10<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>order id</td>
        <td>36942337</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644330189737</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
</table>

### Cancel order [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'order_cancel';

$req = json_encode(array(
    'order_id' => '36942337',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "order_id": "36942337",
    "ts": 1644330189737
}
```

+ Response 200 (application/json)

    + Body

```json
{
   "success": true
}
```

            
## Cancel orders [/orders_cancel] 

Cancellation all/partially of orders.<br>
<br>
<b>Request Weight:</b> 300<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs to cancel orders</td>
        <td>TRX_RUB,DOGE_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644330189737</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>list of cancelled orders</td>
        <td></td>
    </tr>
</table>

### Cancellation with pairs and directions [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'orders_cancel';

$req = json_encode(array(
    'pair' => 'TRX_RUB,DOGE_RUB',
    'action' => 'buy',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "TRX_RUB,DOGE_RUB",
    "action": "buy",
    "ts": 1644389071011
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": [
    "36987301",
    "36987294"
  ]
}
```
            
### Cancellation with pairs [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'orders_cancel';

$req = json_encode(array(
    'pair' => 'TRX_RUB,DOGE_RUB',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "TRX_RUB,DOGE_RUB",
    "ts": 1644389254696
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": [
    "36987703",
    "36987698"
  ]
}
```

### Cancel all orders [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'orders_cancel';

$req = json_encode(array(
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "ts": 1644388528500
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": [
    "36987021",
    "36987015"
  ]
}
```


## My orders [/my_orders] 

Getting your open orders with the ability to filtering.<br>
<br>
<b>Request Weight:</b> 60<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs</td>
        <td>BTC_USD,TRX_USD</td>
    </tr>
    <tr>
        <td>action</td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644396742116</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>list of orders</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>order id</td>
        <td>36989287</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp of order creation</td>
        <td>1644391218</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>pair</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>order type</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>0.05000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>stop price</td>
        <td>0.04000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>0.50</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>processed amount</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>remaining amount</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>processed value</td>
        <td>0.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>remaining value</td>
        <td>0.50</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>mark of order creation via API</td>
        <td>true, false</td>
    </tr>
</table>

### Without filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_orders';

$req = json_encode(array(
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "ts": 1644398635534
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "36149941": {
      "id": "36149941",
      "date": 1643186519,
      "pair": "TRX_RUB",
      "action": "buy",
      "type": "stop_limit",
      "amount": "10.000000",
      "price": "5.00",
      "value": "50.00",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "50.00",
      "stop_price": "4.30"
    },
    "36150146": {
      "id": "36150146",
      "date": 1643186799,
      "pair": "TRX_RUB",
      "action": "sell",
      "type": "stop_limit",
      "amount": "15.000000",
      "price": "4.00",
      "value": "60.00",
      "amount_processed": "0.000000",
      "amount_remaining": "15.000000",
      "value_processed": "0.00",
      "value_remaining": "60.00",
      "stop_price": "4.00"
    },
    "36989287": {
      "id": "36989287",
      "date": 1644391218,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "limit",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50"
    },
    "36989301": {
      "id": "36989301",
      "date": 1644391233,
      "pair": "TRX_USD",
      "action": "sell",
      "type": "limit",
      "amount": "10.000000",
      "price": "0.07000",
      "value": "0.70",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.70"
    },
    "36989322": {
      "id": "36989322",
      "date": 1644391269,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "stop_limit",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50",
      "stop_price": "0.04000"
    },
    "36995144": {
      "id": "36995144",
      "date": 1644398632,
      "pair": "DASH_USD",
      "action": "buy",
      "type": "limit",
      "amount": "0.01000000",
      "price": "100.00",
      "value": "1.00",
      "amount_processed": "0.00000000",
      "amount_remaining": "0.01000000",
      "value_processed": "0.00",
      "value_remaining": "1.00"
    }
  }
}
```

### With filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_orders';

$req = json_encode(array(
    'pair' => 'BTC_USD,TRX_USD',
    'action' => 'buy',
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "BTC_USD,TRX_USD",
    "action": "buy",
    "ts": 1644396742116
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "36989287": {
      "id": "36989287",
      "date": 1644391218,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "limit",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50"
    },
    "36989322": {
      "id": "36989322",
      "date": 1644391269,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "stop_limit",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50",
      "stop_price": "0.04000"
    }
  }
}
```

    
## My history [/my_history] 

Getting the history of your orders with the possibility of filtering and page-by-page loading.<br>
<br>
<b>Request Weight:</b> 60<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>status</td>
        <td>status</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>date_from</td>
        <td>timestamp of the beginning of the filtering period</td>
        <td>1630443600</td>
    </tr>
    <tr>
        <td>date_to</td>
        <td>timestamp of the end of the filtering period<br>the filtering period should not exceed 32 days</td>
        <td>1633035599</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id of the last known order for page-by-page loading of elements</td>
        <td>36989301</td>
    </tr>
    <tr>
        <td>limit</td>
        <td>the count of returned items (by default: 50)</td>
        <td>3</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644399382490</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>list of orders</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>order id</td>
        <td>28203919</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp of order creation</td>
        <td>1631198637</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>pair</td>
        <td>BTC_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>order type</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>status</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>47880.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>stop price</td>
        <td>60000.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>4.79</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>processed amount</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>remaining amount</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>processed value</td>
        <td>0.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>remaining value</td>
        <td>4.79</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>mark of order creation via API</td>
        <td>true, false</td>
    </tr>
</table>

### Without filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_history';

$req = json_encode(array(
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "limit": 3,
    "ts": 1644399161866
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "36989301": {
      "id": "36989301",
      "date": 1644391233,
      "pair": "TRX_USD",
      "action": "sell",
      "type": "limit",
      "status": "processing",
      "amount": "10.000000",
      "price": "0.07000",
      "value": "0.70",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.70"
    },
    "36989322": {
      "id": "36989322",
      "date": 1644391269,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "stop_limit",
      "status": "processing",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50",
      "stop_price": "0.04000"
    },
    "36995144": {
      "id": "36995144",
      "date": 1644398632,
      "pair": "DASH_USD",
      "action": "buy",
      "type": "limit",
      "status": "processing",
      "amount": "0.01000000",
      "price": "100.00",
      "value": "1.00",
      "amount_processed": "0.00000000",
      "amount_remaining": "0.01000000",
      "value_processed": "0.00",
      "value_remaining": "1.00"
    }
  }
}
```

### Paginated navigation [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_history';

$req = json_encode(array(
    'append' => 36989301,
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "append": 36989301,
    "limit": 3,
    "ts": 1644399899071
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "36942226": {
      "id": "36942226",
      "date": 1644328859,
      "pair": "TRX_RUB",
      "action": "buy",
      "type": "limit",
      "status": "success",
      "amount": "10.000000",
      "price": "5.10",
      "value": "51.00",
      "amount_processed": "10.000000",
      "amount_remaining": "0.000000",
      "value_processed": "51.00",
      "value_remaining": "0.00"
    },
    "36942337": {
      "id": "36942337",
      "date": 1644328980,
      "pair": "TRX_RUB",
      "action": "buy",
      "type": "stop_limit",
      "status": "canceled",
      "amount": "10.000000",
      "price": "6.00",
      "value": "60.00",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "60.00",
      "stop_price": "5.50"
    },
    "36989287": {
      "id": "36989287",
      "date": 1644391218,
      "pair": "TRX_USD",
      "action": "buy",
      "type": "limit",
      "status": "processing",
      "amount": "10.000000",
      "price": "0.05000",
      "value": "0.50",
      "amount_processed": "0.000000",
      "amount_remaining": "10.000000",
      "value_processed": "0.00",
      "value_remaining": "0.50"
    }
  }
}
```

### With filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_history';

$req = json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
    'action' => 'buy',
    'status' => 'canceled',
    'date_from' => strtotime('01.09.2021 00:00:00'),
    'date_to' => strtotime('30.09.2021 23:59:59'),
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method); 
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "BTC_USD,BTC_RUB",
    "action": "buy",
    "status": "canceled",
    "date_from": 1630443600,
    "date_to": 1633035599,
    "limit": 3,
    "ts": 1644399382490
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "28203919": {
      "id": "28203919",
      "date": 1631198637,
      "pair": "BTC_USD",
      "action": "buy",
      "type": "stop_limit",
      "status": "canceled",
      "amount": "0.00010000",
      "price": "47880.00",
      "value": "4.79",
      "amount_processed": "0.00000000",
      "amount_remaining": "0.00010000",
      "value_processed": "0.00",
      "value_remaining": "4.79",
      "stop_price": "60000.00"
    },
    "28252727": {
      "id": "28252727",
      "date": 1631274716,
      "pair": "BTC_USD",
      "action": "buy",
      "type": "limit",
      "status": "canceled",
      "amount": "0.00010000",
      "price": "10000.00",
      "value": "1.00",
      "amount_processed": "0.00000000",
      "amount_remaining": "0.00010000",
      "value_processed": "0.00",
      "value_remaining": "1.00"
    },
    "29032159": {
      "id": "29032159",
      "date": 1632483924,
      "pair": "BTC_USD",
      "action": "buy",
      "type": "stop_limit",
      "status": "canceled",
      "amount": "0.00010000",
      "price": "44717.16",
      "value": "4.48",
      "amount_processed": "0.00000000",
      "amount_remaining": "0.00010000",
      "value_processed": "0.00",
      "value_remaining": "4.48",
      "stop_price": "10000.00"
    }
  }
}
```


## My trades [/my_trades] 

Getting your trades with the possibility of filtering and page-by-page loading.<br>
<br>
<b>Request Weight:</b> 60<br>
<br>
<b>Request Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>list of pairs</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>date_from</td>
        <td>timestamp of the beginning of the filtering period</td>
        <td>1630443600</td>
    </tr>
    <tr>
        <td>date_to</td>
        <td>timestamp of the end of the filtering period<br>the filtering period should not exceed 32 days</td>
        <td>1633035599</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id of the last known transaction for page-by-page loading of items</td>
        <td>14163029</td>
    </tr>
    <tr>
        <td>limit</td>
        <td>the count of returned items (by default: 50)</td>
        <td>3</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp in milliseconds</td>
        <td>1644399382490</td>
    </tr>
</table>
<br>
<b>Response Parameters:</b><br>
<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>request success indicator</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>list of trades</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>trade id</td>
        <td>12259786</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>trade timestamp</td>
        <td>1632754083</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>pair</td>
        <td>BTC_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>action</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>status</td>
        <td>success, processing</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>amount</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>price</td>
        <td>44144.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>value</td>
        <td>4.42</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>is_maker</b></td>
        <td>you are a maker</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>is_taker</b></td>
        <td>you are a taker</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_order_id</td>
        <td>maker's order id</td>
        <td>38209521</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_transaction_id</td>
        <td>maker's transaction id</td>
        <td>1505475255</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_fee</td>
        <td>maker's commission</td>
        <td>0.03</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_order_id</td>
        <td>taker's order id</td>
        <td>38209521</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_transaction_id</td>
        <td>taker's transaction id</td>
        <td>15054752556</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_fee</td>
        <td>taker's commission</td>
        <td>0.36</td>
    </tr>
</table>

### Without filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_trades';

$req = json_encode(array(
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "limit": 3,
    "ts": 1644400380453
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "14163029": {
      "id": "14163029",
      "date": 1644328859,
      "pair": "TRX_RUB",
      "action": "buy",
      "status": "success",
      "amount": "10.000000",
      "price": "5.10",
      "value": "51.00",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1597309838"
    },
    "14175584": {
      "id": "14175584",
      "date": 1644400287,
      "pair": "TRX_USD",
      "action": "sell",
      "status": "success",
      "amount": "10.000000",
      "price": "0.06961",
      "value": "0.70",
      "is_maker": true,
      "is_taker": true,
      "m_transaction_id": "1597903047",
      "t_transaction_id": "1597903042"
    },
    "14175600": {
      "id": "14175600",
      "date": 1644400375,
      "pair": "TRX_USD",
      "action": "buy",
      "status": "success",
      "amount": "10.000000",
      "price": "0.06963",
      "value": "0.70",
      "is_maker": true,
      "is_taker": false,
      "m_transaction_id": "1597904047"
    }
  }
}
```

### Paginated navigation [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_trades';

$req = json_encode(array(
    'append' => 14163029,
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "append": 14163029,
    "limit": 3,
    "ts": 1644400526581
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "14162999": {
      "id": "14162999",
      "date": 1644328727,
      "pair": "TRX_RUB",
      "action": "buy",
      "status": "success",
      "amount": "8.983649",
      "price": "5.10",
      "value": "45.81",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1597308117"
    },
    "14163000": {
      "id": "14163000",
      "date": 1644328727,
      "pair": "TRX_RUB",
      "action": "buy",
      "status": "success",
      "amount": "1.016351",
      "price": "5.10",
      "value": "5.18",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1597308120"
    },
    "14163006": {
      "id": "14163006",
      "date": 1644328754,
      "pair": "TRX_RUB",
      "action": "buy",
      "status": "success",
      "amount": "10.000000",
      "price": "5.10",
      "value": "51.00",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1597308453"
    }
  }
}
```

### With filtering [POST]

<b>PHP</b><br>

```php
$msec = round(microtime(true) * 1000);

$apiId = 'bd443f00-092c-4436-92a4-a704ef679e24';
$apiSecret = 'api_secret_key';

$method = 'my_trades';

$req = json_encode(array(
    'pair' => 'BTC_USD,BTC_RUB',
    'action' => 'buy',
    'date_from' => strtotime('01.09.2021 00:00:00'),
    'date_to' => strtotime('30.09.2021 23:59:59'),
    'limit' => 3,
    'ts' => $msec,
));

$sign = hash_hmac('sha256', $method.$req, $apiSecret);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://payeer.com/api/trade/".$method);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $req);

curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json",
    "API-ID: ".$apiId,
    "API-SIGN: ".$sign
));

$response = curl_exec($ch);
curl_close($ch);

echo '<pre>'; print_r(json_decode($response, true)); echo '</pre>';
```

+ Request (application/json)
    
    + Headers

            API-ID: bd443f00-092c-4436-92a4-a704ef679e24
            API-SIGN: f133d2e7a960a3db86052be6d4f7699313e9416bce557868e7ad0f026c67c9ca
            
    + Body
    
```json
{
    "pair": "BTC_USD,BTC_RUB",
    "action": "buy",
    "date_from": 1630443600,
    "date_to": 1633035599,
    "limit": 3,
    "ts": 1644400662184
}
```

+ Response 200 (application/json)

    + Body

```json
{
  "success": true,
  "items": {
    "12259786": {
      "id": "12259786",
      "date": 1632754083,
      "pair": "BTC_USD",
      "action": "buy",
      "status": "success",
      "amount": "0.00010000",
      "price": "44144.00",
      "value": "4.42",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1505475255"
    },
    "12259796": {
      "id": "12259796",
      "date": 1632754158,
      "pair": "BTC_USD",
      "action": "buy",
      "status": "success",
      "amount": "0.00010000",
      "price": "44000.00",
      "value": "4.40",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1505476013"
    },
    "12259915": {
      "id": "12259915",
      "date": 1632754502,
      "pair": "BTC_USD",
      "action": "buy",
      "status": "success",
      "amount": "0.00010000",
      "price": "44000.00",
      "value": "4.40",
      "is_maker": false,
      "is_taker": true,
      "t_transaction_id": "1505480022"
    }
  }
}
```

# Payeer Trade API (ru)

API URL: https://payeer.com/api/trade/

## Public API 

* [Проверка соединения](#проверка-соединения-time) [/time]
* [Пары и лимиты](#пары-и-лимиты-info) [/info]
    * [Все пары](#все-пары-get)
    * [Выборка](#выборка-post)
* [Статистика цен](#статистика-цен-ticker) [/ticker]
    * [Все пары](#все-пары-get-1)
    * [Выборка](#выборка-post-1)
* [Ордера](#ордера-orders) [/orders]
* [Сделки](#сделки-trades) [/trades]

## Private API 

* [Баланс пользователя](#баланс-пользователя-account) [/account]
* [Создание ордера](#создание-ордера-order_create) [/order_create]
    * [Лимит](#лимит-post)
    * [Маркет](#маркет-post)
    * [Стоп-лимит](#стоп-лимит-post)
* [Статус ордера](#статус-ордера-order_status) [/order_status]
* [Отмена ордера](#отмена-ордера-order_cancel) [/order_cancel]
* [Отмена ордеров](#отмена-ордеров-orders_cancel) [/orders_cancel]
    * [Отмена с указанием пар и направления](#отмена-с-указанием-пар-и-направления-post)
    * [Отмена с указанием пар](#отмена-с-указанием-пар-post)
    * [Отмена всех ордеров](#отмена-всех-ордеров-post)
* [Мои ордера](#мои-ордера-my_orders) [/my_orders]
    * [Без фильтрации](#без-фильтрации-post)
    * [С фильтрацией](#с-фильтрацией-post)
* [Моя история](#моя-история-my_history) [/my_history] 
    * [Без фильтрации](#без-фильтрации-post-1)
    * [Постраничная навигация](#постраничная-навигация-post)
    * [С фильтрацией](#с-фильтрацией-post-1)
* [Мои сделки](#мои-сделки-my_trades) [/my_trades]
    * [Без фильтрации](#без-фильтрации-post-2)
    * [Постраничная навигация](#постраничная-навигация-post-1)
    * [С фильтрацией](#с-фильтрацией-post-2)

<br>
Настройки API: https://payeer.com/ru/account/api/?tab=trade
<br><br>

Для доступа к Private API необходимо передать в заголовках следующие ключи:
* API-ID - ID API-пользователя
* API-SIGN - подпись запроса

Также в теле запроса обязательным параметром является <code>ts</code>, значение которого равно текущему timestamp <b>в миллисекундах</b>.<br>
<br>
Запрос будет обработан только если дойдет до нашего сервера в течение 60 секунд, чтобы исключить существенные задержки сети.<br>
<br>
Максимальная разница параметра <code>ts</code> от реального времени не должна превышать 1 секунду, не забудьте настроить синхронизацию времени на своем сервере.<br>
<br>
Тело запроса передается в виде json-строки.<br>
<br><br>
<b>Генерация подписи:</b><br>
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

где <code>apiSecret</code> - секретный ключ API-пользователя.<br>
<br>
Подпись создается с помощью метода HMAC-SHA256. 

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

## Ошибки

Список возможных ошибок.<br>

<table>
    <tr>
        <th>Ошибка</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>INVALID_SIGNATURE</b></td>
        <td>
            Возможные причины:<br>
            - неверная подпись API-SIGN<br>
            - неверно указан API-ID<br>
            - API-пользователь заблокирован (можно разблокировать в настройках)<br>
        </td>
    </tr>
    <tr>
        <td><b>INVALID_IP_ADDRESS</b></td>
        <td>IP-адрес <code>ip</code> источника запроса не совпадает с тем, который прописан в настройках API</td>
    </tr>
    <tr>
        <td><b>LIMIT_EXCEEDED</b></td>
        <td>Превышение установленных лимитов (количество запросов/весов/ордеров), подробнее указано в параметре <code>desc</code></td>
    </tr>
    <tr>
        <td><b>INVALID_TIMESTAMP</b></td>
        <td>
            параметр <code>ts</code> указан неверно:<br>
            - запрос шел до сервера API более 60 секунд<br>
            - на вашем сервере неверное время, настройте синхронизацию
        </td>
    </tr>
    <tr>
        <td><b>ACCESS_DENIED</b></td>
        <td>Доступ к API/ордеру запрещен. Проверьте <code>order_id</code></td>
    </tr>
    <tr>
        <td><b>INVALID_PARAMETER</b></td>
        <td>параметр <code>parameter</code> указан неверно</td>
    </tr>
    <tr>
        <td><b>PARAMETER_EMPTY</b></td>
        <td>параметр <code>parameter</code> обязателен (не должен быть пустым)</td>
    </tr>
    <tr>
        <td><b>INVALID_STATUS_FOR_REFUND</b></td>
        <td>статус <code>status</code> ордера не позволяет произвести возврат (ордер уже возвращен или отменен)</td>
    </tr>
    <tr>
        <td><b>REFUND_LIMIT</b></td>
        <td>ордер может быть отменен не менее через 1 минуту после создания</td>
    </tr>
    <tr>
        <td><b>UNKNOWN_ERROR</b></td>
        <td>Неизвестная ошибка на бирже. Торги приостановлены для проверки. Попробуйте через 15 минут.</td>
    </tr>
    <tr>
        <td><b>INVALID_DATE_RANGE</b></td>
        <td>Неверно указан диапазон дат для фильтрации (период не должен превышать 32 дня)</td>
    </tr>
    <tr>
        <td><b>INSUFFICIENT_FUNDS</b></td>
        <td>недостаточно средств для создания ордера (<code>max_amount</code>, <code>max_value</code>)</td>
    </tr>
    <tr>
        <td><b>INSUFFICIENT_VOLUME</b></td>
        <td>недостаточно объема для создания ордера (<code>max_amount</code>, <code>max_value</code>)</td>
    </tr>
    <tr>
        <td><b>INCORRECT_PRICE</b></td>
        <td>цена выходит из допустимого диапазона (<code>min_price</code>, <code>max_price</code>)</td>
    </tr>
    <tr>
        <td><b>MIN_AMOUNT</b></td>
        <td>количество меньше минимального <code>min_amount</code> для выбранной пары</td>
    </tr>
    <tr>
        <td><b>MIN_VALUE</b></td>
        <td>сумма ордера меньше минимальной <code>min_value</code> для выбранной пары</td>
    </tr>
</table>         

<br>


## Проверка соединения [/time] 

Проверка соединения, получение времени сервера.<br>
<br>
<b>Вес запроса:</b> 1<br>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>time</b></td>
        <td>timestamp сервера</td>
        <td>1644322909335</td>
    </tr>
</table>

### Проверка соединения [GET]

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


## Пары и лимиты [/info]

Получение лимитов, доступных пар и их параметров.<br>
<br>
<b>Вес запроса:</b> 1<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>limits</b></td>
        <td>
            список лимитов:<br>
            &nbsp;&nbsp;&nbsp;<code>requests</code> - запросов<br>
            &nbsp;&nbsp;&nbsp;<code>weights</code> - весов<br>
            &nbsp;&nbsp;&nbsp;<code>orders</code> - ордеров
        </td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>interval</b></td>
        <td>интервал</td>
        <td>min, hour, day</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>interval_num</b></td>
        <td>
            величина интервала:<br>
            &nbsp;&nbsp;&nbsp;min, 1: за одну минуту<br>
            &nbsp;&nbsp;&nbsp;min, 5: за 5 минут<br>
            &nbsp;&nbsp;&nbsp;hour, 1: за 1 час
        </td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>limit</b></td>
        <td>
            лимит:<br>
            &nbsp;&nbsp;&nbsp;<code>requests</code>, min 1, 600: 600 запросов в минуту<br>
            &nbsp;&nbsp;&nbsp;<code>weights</code>, min 1, 600: 600 весов в минуту<br>
            &nbsp;&nbsp;&nbsp;<code>orders</code>, min 1, 120: 120 ордеров в минуту
        </td>
        <td>600</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>список пар</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price_prec</b></td>
        <td>количеcтво знаков после точки для параметра <code>price</code></td>
        <td>2</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_price</b></td>
        <td>минимальная цена для создания ордера</td>
        <td>4375.74</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>max_price</b></td>
        <td>максимальная цена для создания ордера</td>
        <td>83139.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_amount</b></td>
        <td>минимальное количество для создания ордера</td>
        <td>0.0001</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min_value</b></td>
        <td>минимальная сумма для создания ордера</td>
        <td>0.5</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>fee_maker_percent</b></td>
        <td>комиссия мейкера</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>fee_taker_percent</b></td>
        <td>комиссия тейкера</td>
        <td>0.095</td>
    </tr>
</table>
        
### Все пары [GET]

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

### Выборка [POST]

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
            

## Статистика цен [/ticker]

Получение статистики цен и их колебания за последние 24 часа.<br>
<br>
<b>Вес запроса:</b> 1<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>список пар</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>ask</b></td>
        <td>цена предложения - наименьшая цена продавца, по которой он согласен продать</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bid</b></td>
        <td>цена спроса - наивысшая цена покупателя, по которой он согласен купить</td>
        <td>43502.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>last</b></td>
        <td>цена последней сделки</td>
        <td>43501.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>min24</b></td>
        <td>минимальная цена за 24 часа</td>
        <td>43006.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>max24</b></td>
        <td>максимальная цена за 24 часа</td>
        <td>46000.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>delta</b></td>
        <td>изменение цены за 24 часа в процентах</td>
        <td>0.70</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>delta_price</b></td>
        <td>изменение цены за 24 часа</td>
        <td>301.00</td>
    </tr>
</table>
        
### Все пары [GET]

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

### Выборка [POST]

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


## Ордера [/orders]

Получение доступных ордеров по указанным парам.<br>
<br>
<b>Вес запроса:</b> 1 * количество пар<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>список пар</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>ask</b></td>
        <td>цена предложения - наименьшая цена продавца, по которой он согласен продать</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bid</b></td>
        <td>цена спроса - наивысшая цена покупателя, по которой он согласен купить</td>
        <td>43520.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>asks</b></td>
        <td>список ордеров на продажу</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.00031422</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>13.76</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>bids</b></td>
        <td>список ордеров на покупку</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>43520.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.00034788</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>15.13</td>
    </tr>
</table>

### Ордера [POST]

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
        

## Сделки [/trades]

Получение истории сделок по указанным парам.<br>
<br>
<b>Вес запроса:</b> 1 * количество пар<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>pairs</b></td>
        <td>список пар</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id сделки</td>
        <td>14162760</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp сделки</td>
        <td>1644327656</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.00031422</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>43790.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>13.76</td>
    </tr>
</table>

### Сделки [POST]

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
            

## Баланс пользователя [/account]

Получение баланса пользователя.<br>
<br>
<b>Вес запроса:</b> 10<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644327967240</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>balances</b></td>
        <td>список валют пользователя</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>total</b></td>
        <td>общий баланс</td>
        <td>1598.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>available</b></td>
        <td>доступный баланс</td>
        <td>1548.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>hold</b></td>
        <td>заблокированные д/с</td>
        <td>50</td>
    </tr>
</table>

### Баланс пользователя [POST]

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
    

## Создание ордера [/order_create]

Создание ордера поддерживаемых типов: лимит, маркет, стоп-лимит.<br>
<br>
<b>Вес запроса:</b> 5 (10 для маркет-ордера)<br>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>id созданного ордера</td>
        <td>37057767</td>
    </tr>
    <tr>
        <td><b>params</b></td>
        <td>параметры созданного ордера</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>тип ордера</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>стоп-цена</td>
        <td>0.07800</td>
    </tr>
</table>

### Лимит [POST]

<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>тип ордера</td>
        <td>limit</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>amount</b></td>
        <td>количество</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>price</b></td>
        <td>цена</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
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

### Маркет [POST]

<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>тип ордера</td>
        <td>market</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>* amount</td>
        <td>количество</td>
        <td>10</td>
    </tr>
    <tr>
        <td>* value</td>
        <td>сумма</td>
        <td>10000</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644489441948</td>
    </tr>
</table>

\* - возможно указание одного из двух параметров для создания маркет-ордера (<code>amount</code> или <code>value</code>)

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

### Стоп-лимит [POST]

<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>тип ордера</td>
        <td>stop_limit</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>amount</b></td>
        <td>количество</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>price</b></td>
        <td>цена</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td><b>stop_price</b></td>
        <td>стоп-цена</td>
        <td>0.078</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
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

        
## Статус ордера [/order_status]

Получение подробной информации о своем ордере по его id.<br>
<br>
<b>Вес запроса:</b> 5<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>id ордера</td>
        <td>37054293</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644493185157</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>order</b></td>
        <td>параметры ордера</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id ордера</td>
        <td>37054293</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp создания ордера</td>
        <td>1644488809</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>тип ордера</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>статус ордера</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>стоп-цена</td>
        <td>0.08000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>обработанное количество</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>оставшееся количество</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>обработанная сумма</td>
        <td>0.72</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>оставшаяся сумма</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>признак создания ордера через API</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>avg_price</b></td>
        <td>средняя цена</td>
        <td>0.07200</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;trades</td>
        <td>список сделок</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id сделки</td>
        <td>14190472</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp сделки</td>
        <td>1644488809</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>статус</td>
        <td>success, processing</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>0.07150</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.054165</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>is_maker</b></td>
        <td>вы являетесь мейкером по сделке</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>is_taker</b></td>
        <td>вы являетесь тейкером по сделке</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_transaction_id</td>
        <td>id транзакции мейкера</td>
        <td>1598693542</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_fee</td>
        <td>комиссия мейкера</td>
        <td>0.03</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t_transaction_id</td>
        <td>id транзакции тейкера</td>
        <td>1598693543</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t_fee</td>
        <td>комиссия тейкера</td>
        <td>0.36</td>
    </tr>
</table>

### Статус ордера [POST]

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

    
## Отмена ордера [/order_cancel]

Отмена своего ордера по его id.<br>
<br>
<b>Вес запроса:</b> 10<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>order_id</b></td>
        <td>id ордера</td>
        <td>36942337</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644330189737</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
</table>

### Отмена ордера [POST]

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
            
            
## Отмена ордеров [/orders_cancel] 

Отмена всех/части своих ордеров.<br>
<br>
<b>Вес запроса:</b> 300<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар для отмены ордеров</td>
        <td>TRX_RUB,DOGE_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644330189737</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>список отмененных ордеров</td>
        <td></td>
    </tr>
</table>

### Отмена с указанием пар и направления [POST]

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
            
### Отмена с указанием пар [POST]

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
            
### Отмена всех ордеров [POST]

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


## Мои ордера [/my_orders] 

Получение своих открытых ордеров с воможностью фильтрации.<br>
<br>
<b>Вес запроса:</b> 60<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар</td>
        <td>BTC_USD,TRX_USD</td>
    </tr>
    <tr>
        <td>action</td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644396742116</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>список ордеров</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id ордера</td>
        <td>36989287</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp создания ордера</td>
        <td>1644391218</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>пара</td>
        <td>TRX_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>тип ордера</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>0.05000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>стоп-цена</td>
        <td>0.04000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>0.50</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>обработанное количество</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>оставшееся количество</td>
        <td>10.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>обработанная сумма</td>
        <td>0.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>оставшаяся сумма</td>
        <td>0.50</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>признак создания ордера через API</td>
        <td>true, false</td>
    </tr>
</table>

### Без фильтрации [POST]

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
            
### С фильтрацией [POST]

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
        
    
## Моя история [/my_history] 

Получение истории своих ордеров с возможностью фильтрации и постраничной загрузки.<br>
<br>
<b>Вес запроса:</b> 60<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>status</td>
        <td>статус</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>date_from</td>
        <td>timestamp начала периода фильтрации</td>
        <td>1630443600</td>
    </tr>
    <tr>
        <td>date_to</td>
        <td>timestamp окончания периода фильтрации<br>период фильтрации не должен превышать 32 дней</td>
        <td>1633035599</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id поседнего известного ордера для постраничной загрузки элементов</td>
        <td>36989301</td>
    </tr>
    <tr>
        <td>limit</td>
        <td>количество возвращаемых элементов (по-умолчанию: 50)</td>
        <td>3</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644399382490</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>список ордеров</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id ордера</td>
        <td>28203919</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp создания ордера</td>
        <td>1631198637</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>пара</td>
        <td>BTC_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>type</b></td>
        <td>тип ордера</td>
        <td>limit, market, stop_limit</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>статус</td>
        <td>success, processing, waiting, canceled</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>47880.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;stop_price</td>
        <td>стоп-цена</td>
        <td>60000.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>4.79</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_processed</b></td>
        <td>обработанное количество</td>
        <td>0.000000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount_remaining</b></td>
        <td>оставшееся количество</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_processed</b></td>
        <td>обработанная сумма</td>
        <td>0.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value_remaining</b></td>
        <td>оставшаяся сумма</td>
        <td>4.79</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>api</b></td>
        <td>признак создания ордера через API</td>
        <td>true, false</td>
    </tr>
</table>

### Без фильтрации [POST]

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

### Постраничная навигация [POST]

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

### С фильтрацией [POST]

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


## Мои сделки [/my_trades] 

Получение своих сделок с возможностью фильтрации и постраничной загрузки.<br>
<br>
<b>Вес запроса:</b> 60<br>
<br>
<b>Параметры запроса:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td>pair</td>
        <td>список пар</td>
        <td>BTC_USD,BTC_RUB</td>
    </tr>
    <tr>
        <td>action</td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>date_from</td>
        <td>timestamp начала периода фильтрации</td>
        <td>1630443600</td>
    </tr>
    <tr>
        <td>date_to</td>
        <td>timestamp окончания периода фильтрации<br>период фильтрации не должен превышать 32 дней</td>
        <td>1633035599</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id последней известной сделки для постраничной загрузки элементов</td>
        <td>14163029</td>
    </tr>
    <tr>
        <td>limit</td>
        <td>количество возвращаемых элементов (по-умолчанию: 50)</td>
        <td>3</td>
    </tr>
    <tr>
        <td><b>ts</b></td>
        <td>timestamp в миллисекундах</td>
        <td>1644399382490</td>
    </tr>
</table>
<br>
<b>Параметры ответа:</b><br>
<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>признак успешности запроса</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td><b>items</b></td>
        <td>список сделок</td>
        <td></td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>id</b></td>
        <td>id сделки</td>
        <td>12259786</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>date</b></td>
        <td>timestamp сделки</td>
        <td>1632754083</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>pair</b></td>
        <td>пара</td>
        <td>BTC_USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>action</b></td>
        <td>действие</td>
        <td>buy, sell</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>status</b></td>
        <td>статус</td>
        <td>success, processing</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>количество</td>
        <td>0.00010000</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>price</b></td>
        <td>цена</td>
        <td>44144.00</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>value</b></td>
        <td>сумма</td>
        <td>4.42</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>is_maker</b></td>
        <td>вы являетесь мейкером по сделке</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>is_taker</b></td>
        <td>вы являетесь тейкером по сделке</td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_order_id</td>
        <td>id ордера мейкера</td>
        <td>38209521</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_transaction_id</td>
        <td>id транзакции мейкера</td>
        <td>1505475255</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;m_fee</td>
        <td>комиссия мейкера</td>
        <td>0.03</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_order_id</td>
        <td>id ордера тейкера</td>
        <td>38209521</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_transaction_id</td>
        <td>id транзакции тейкера</td>
        <td>15054752556</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;t_fee</td>
        <td>комиссия тейкера</td>
        <td>0.36</td>
    </tr>
</table>

### Без фильтрации [POST]

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

### Постраничная навигация [POST]

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

### С фильтрацией [POST]

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

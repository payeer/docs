# Payeer API (ru)

API URL: https://payeer.com/ajax/api/api.php

* [Проверка авторизации](#проверка-авторизации)
* [Проверка баланса](#проверка-баланса)
* [Перевод средств](#перевод-средств)
* [Проверка существования аккаунта](#проверка-существования-аккаунта)
* [Курсы автоматической конвертации](#курсы-автоматической-конвертации)
* [Проверка возможности выплаты](#проверка-возможности-выплаты)
* [Выплата](#выплата)
* [Получение доступных платежных систем](#получение-доступных-платежных-систем)
* [Информация о платеже](#информация-о-платеже)
* [История операций](#история-операций)
* [Информация по выплате](#информация-по-выплате)
* [Создание счета для оплаты](#создание-счета-для-оплаты)
* [API Merchant](#api-merchant)

Payeer предоставляет возможность внешним разработчикам программными средствами взаимодействовать с сервисом. Одним из способов такого взаимодействия является использование API.
API определяет набор функций для осуществления взаимодействия по протоколу HTTPs. С помощью данных функций можно получить доступ к различным ресурсам системы, например, информацию о платежных направлениях в системе Payeer, курсах валют, информацию по конкретному платежу в системе, а также создать вывод средств. 
Взаимодействие осуществляется с помощью POST запросов в кодировке UTF-8 к URL: https://payeer.com/ajax/api/api.php
Каждый ответ от сервера Payeer обязательно содержит поле <code>auth_error</code>, который показывает правильно ли указаны параметры аутентификации.
В ответе на запрос также присутствует массив <code>errors</code>, который говорит о наличии ошибок в выполнении запроса. Пустой errors, а также равный false, null или 0 говорит об отсутствии каких либо ошибок в выполнении запроса, иначе возвращается текстовое описание возникшей ошибки. 

## Начало работы с API
Для работы с API, необходимо в личном кабинете https://payeer.com/ru/account/?tab=api нажать на кнопку "Активация" в правом столбце.

Заполните название пользователя, сгенерируйте секретный ключ для работы с API и для обеспечения дополнительной безопасности, пропишите IP адрес, с которого Вы будете отправлять запросы на сервер Payeer. 

Внимание! Никому не сообщайте секретный ключ для API.

После добавления API Вам выдается ID пользователя, который необходимо использовать, совместно с ключом и номером аккаунта, для аутентификации в запросах.  

## Часто задаваемые вопросы
* При запросе к API мне возвращается ошибка "Auth error: account or apiId or apiPass is incorrect or api-user was blocked"
<br><br><code>Проверьте правильность указания номера счета, ID апи-пользователя и секретного ключа, а также выключенную блокировку пользователя в настройках Массовых выплат.</code>

* Не удается избавиться от ошибки "IP 1.2.3.4 does not satisfy the security settings"
<br><br><code>Укажите IP, который возвращается в ошибке, в настройках массовых выплат и повторите запрос.</code>

* Через некоторое время опять появляется ошибка "Auth error: account or apiId or apiPass is incorrect or api-user was blocked", хотя никакие настройки не менялись
<br><br><code>Через некоторое количество попыток авторизации с неверным секретным ключом апи-пользователь блокируется, разблокировать его можно в настройках массовых выплат. Возможно Вы сменили секретный ключ не во всех местах на своем сайте, или кто-то пытается подобрать пароль к Вашему API: <br>1) Удалите старого апи-пользователя<br>2) Создайте нового с защитой по IP<br>3) Пропишите новые данные доступа на своем сайте</code>

* У меня взломали сайт и вывели все деньги через API
<br><br><code>Для дополнительной безопасности Вы можете использовать для выплат второй аккаунт, на котором держать сумму, необходимую для переводов в краткосрочном периоде. Пожалуйста, не пренебрегайте аудитами безопасности Вашего сайта.</code>

## Проверка авторизации
Cамый простой запрос - проверка авторизации. Выполнять его отдельно от основных действий не обязательно.

Для проверки авторизации необходимо отправить POST-запрос на URL https://payeer.com/ajax/api/api.php с тремя параметрами: 
<code>account</code>, <code>apiId</code> и <code>apiPass</code>:

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
</table>

### Проверка авторизации [POST]

В теле ответа будут возвращены два параметра: <code>auth_error</code> и <code>errors</code>:

<table>
    <tr>
        <th>auth_error</th>
        <th>errors</th>
        <th></th>
    </tr>
    <tr>
        <td><b>0</b></td>
        <td></td>
        <td><b>Авторизация успешна</b></td>
    </tr>
    <tr>
        <td><b>1</b></td>
        <td>Auth error: account or apiId or apiPass is incorrect or api-user was blocked</td>
        <td><b>Ошибка авторизации</b><br>Номер аккаунта, apiId или apiPass указан неверно или данный апи-пользователь заблокирован в настройках</td>
    </tr>
    <tr>
        <td><b>1</b></td>
        <td>IP 1.2.3.4 does not satisfy the security settings</td>
        <td><b>Ошибка авторизации</b><br>Запросы приходят с IP 1.2.3.4, хотя в настройках апи-пользователя прописан другой</td>
    </tr>
</table> 

+ Request

    + Headers
    
            Content-Type: application/x-www-form-urlencoded
            
    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty
        
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[]}

## Проверка баланса

Получение баланса кошелька.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: получение баланса</td>
        <td>getBalance</td>
    </tr>
    <tr>
        <td>type</td>
        <td>тип возвращаемого значения</td>
        <td>object (по-умолчанию), array</td>
    </tr>
</table>

### Проверка баланса [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>currency</b></td>
        <td>валюта</td>
    </tr>
    <tr>
        <td><b>total</b></td>
        <td>общий баланс</td>
    </tr>
    <tr>
        <td><b>available</b></td>
        <td>доступный баланс для расходных операций</td>
    </tr>
    <tr>
        <td><b>hold</b></td>
        <td>сумма замороженных средств, которые находятся в обработке</td>
    </tr>
</table> 

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=getBalance
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"balance":{"USD":{"currency":"USD","total":92.49,"available":92.49,"hold":0},"RUB":{"currency":"RUB","total":218.07,"available":207.77,"hold":10.3},"EUR":{"currency":"EUR","total":4.04,"available":4.04,"hold":0},"BTC":{"currency":"BTC","total":0.00639515,"available":0.00639515,"hold":0},"ETH":{"currency":"ETH","total":0.04548519,"available":0.04548519,"hold":0},"BCH":{"currency":"BCH","total":0.02113917,"available":0.02113917,"hold":0},"LTC":{"currency":"LTC","total":0.0898946,"available":0.0898946,"hold":0},"DASH":{"currency":"DASH","total":0.03482208,"available":0.03482208,"hold":0},"USDT":{"currency":"USDT","total":1.0e-8,"available":1.0e-8,"hold":0},"XRP":{"currency":"XRP","total":52.216613,"available":52.216613,"hold":0}}}
        
## Перевод средств

Перевод средств внутри платежной системы Payeer.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: перевод</td>
        <td>transfer</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>валюта, с которой будет производиться списание</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>sum</b></td>
        <td>сумма списания (сумма зачисления будет рассчитана автоматически с вычетом всех комиссий с получателя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>валюта зачисления (в случае отличия валюты списания от валюты зачисления конвертация будет произведена автоматически по курсу системы Payeer на момент перевода)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>сумма зачисления (сумма списания будет рассчитана автоматически с вычетом всех комиссий с отправителя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>номер Payeer счета или email получателя (если email не был зарегистрирован до перевода, то регистрация произойдет автоматически)</td>
        <td>P1000000, test@mail.com</td>
    </tr>
    <tr>
        <td>comment</td>
        <td>комментарий к переводу (желательно здесь указывать id операции в Вашей системе учета)</td>
        <td>Перевод #1365</td>
    </tr>
    <tr>
        <td>protect</td>
        <td>включение протекции сделки (для зачисления перевода получателю необходимо будет ввести код протекции protectCode в течение protectPeriod дней, в противном случае перевод будет отменен и деньги останутся у отправителя)</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>protectPeriod</td>
        <td>период протекции: от 1 до 30 дней</td>
        <td>от 1 до 30</td>
    </tr>
    <tr>
        <td>protectCode</td>
        <td>код протекции</td>
        <td>12345</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

<b>жирным</b> отмечены обязательные параметры

### Перевод средств [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>массив ошибок или false</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>historyId</td>
        <td>номер транзакции</td>
    </tr>
</table> 

Список возможных ошибок <code>errors</code>:

<table>
    <tr>
        <th>Ошибка</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>balanceError</b></td>
        <td>недостаточно средств для перевода</td>
    </tr>
    <tr>
        <td><b>balanceError000</b></td>
        <td>недостаточно средств для перевода с учетом ограничений по аккаунту</td>
    </tr>
    <tr>
        <td><b>transferHimselfForbidden</b></td>
        <td>переводы самому себе запрещены (при совпадении валюты списания и валюты получения)</td>
    </tr>
    <tr>
        <td><b>transferError</b></td>
        <td>ошибка перевода, необходимо повторить перевод через некоторое время</td>
    </tr>
    <tr>
        <td><b>convertForbidden</b></td>
        <td>автоматический обмен валюты curIn на curOut временно запрещен</td>
    </tr>
    <tr>
        <td><b>protectDay_1_30</b></td>
        <td>ошибка задания периода протекции перевода</td>
    </tr>
    <tr>
        <td><b>protectCodeNotEmpty</b></td>
        <td>при протекции код протекции не должен быть пустой</td>
    </tr>
    <tr>
        <td><b>sumNotNull</b></td>
        <td>сумма отправления и получения не должна быть нулевой</td>
    </tr>
    <tr>
        <td><b>sumInNotMinus</b></td>
        <td>сумма отправления не должна быть быть отрицательной</td>
    </tr>
    <tr>
        <td><b>sumOutNotMinus</b></td>
        <td>сумма получения не должна быть отрицательной</td>
    </tr>
    <tr>
        <td><b>fromError</b></td>
        <td>отправитель не найден</td>
    </tr>
    <tr>
        <td><b>toError</b></td>
        <td>получатель указан неверно</td>
    </tr>
    <tr>
        <td><b>curInError</b></td>
        <td>валюта списания не поддерживается</td>
    </tr>
    <tr>
        <td><b>curOutError</b></td>
        <td>валюта получения не поддерживается</td>
    </tr>
    <tr>
        <td><b>outputHold</b></td>
        <td>У нас возникли вопросы к Вашей деятельности. Пожалуйста, обратитесь в Службу поддержки</td>
    </tr>
    <tr>
        <td><b>transferToForbiddenCountry</b></td>
        <td>Перевод клиентам в некоторые страны запрещен</td>
    </tr>
</table> 

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=transfer&curIn=USD&sum=1&curOut=USD&to=P1000001
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":false,"historyId":100000000}
        

## Проверка существования аккаунта

Можно проверить существование номера аккаунта до перевода в системе Payeer.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: проверка существования аккаунта</td>
        <td>checkUser</td>
    </tr>
    <tr>
        <td><b>user</b></td>
        <td>номер счета пользователя в формате P1000000</td>
        <td>P1000000</td>
    </tr>
</table>

### Проверка существования аккаунта [POST]

Если массив <code>errors</code> пустой - значит пользователь существует.

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=checkUser&user=P1000441
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[]}
        

## Курсы автоматической конвертации

Если при вводе/переводе/выводе валюта списания <code>curIn</code> отличается от валюты получения <code>curOut</code> конвертация происходит по внутренним курсам Payeer, которые можно получить с помощью данного метода.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: получение курсов автоматической конвертации</td>
        <td>getExchangeRate</td>
    </tr>
    <tr>
        <td><b>output</b></td>
        <td>выбор направления курсов конвертации</td>
        <td><b>N</b> - получить курсы ввода<br><b>Y</b> - получить курсы вывода</td>
    </tr>
</table>

### Курсы автоматической конвертации [POST]

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=getExchangeRate&output=Y
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"rate":{"RUB\/USD":"0.01477541","RUB\/RUB":1,"RUB\/EUR":"0.01308525","USD\/USD":1,"USD\/RUB":"61.47230000","USD\/EUR":"0.85757203","EUR\/USD":"1.09716731","EUR\/RUB":"69.53140000"}}
        
## Проверка возможности выплаты

Метод позволяет проверить возможность выплаты без её реального создания (можно получить сумму списания/получения или проверить ошибки в параметрах).

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: проверка возможности выплаты</td>
        <td>payoutChecking</td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>id выбранной платежной системы</td>
        <td>1136053</td>
    </tr>
    <tr>
        <td><b>sumIn</b></td>
        <td>сумма списания (сумма зачисления будет рассчитана автоматически с вычетом всех комиссий с получателя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>валюта, с которой будет производиться списание</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>сумма зачисления (сумма списания будет рассчитана автоматически с вычетом всех комиссий с отправителя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>валюта зачисления (в случае отличия валюты списания от валюты зачисления конвертация будет произведена автоматически по курсу системы Payeer на момент перевода)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>param_ACCOUNT_NUMBER</b></td>
        <td>номер счета получателя в выбранной платежной системе <code>ps</code></td>
        <td>P1000441</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

### Проверка возможности выплаты [POST]
Список возможных ошибок <code>errors</code>:

<table>
    <tr>
        <th>Ошибка</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>curIn invalid</b></td>
        <td>валюта списания не поддерживается</td>
    </tr>
    <tr>
        <td><b>curOut invalid</b></td>
        <td>валюта получения не поддерживается</td>
    </tr>
    <tr>
        <td><b>This type of exchange is not possible</b></td>
        <td>автоматический обмен валюты curIn на curOut временно запрещен</td>
    </tr>
    <tr>
        <td><b>invalid format</b></td>
        <td>неверный формат параметра</td>
    </tr>
    <tr>
        <td><b>pay_sys_no_isset</b></td>
        <td>платежное направление отключено или не существует</td>
    </tr>
    <tr>
        <td><b>cur_no_pay_sys</b></td>
        <td>несуществующая валюта получения</td>
    </tr>
    <tr>
        <td><b>sum_less_min</b></td>
        <td>сумма получения меньше минимальной</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>сумма получения больше максимальной</td>
    </tr>
    <tr>
        <td><b>output_block</b></td>
        <td>У нас возникли вопросы к Вашей деятельности. Пожалуйста, обратитесь в Службу поддержки</td>
    </tr>
    <tr>
        <td><b>balans_no</b></td>
        <td>недостаточно средств для вывода</td>
    </tr>
    <tr>
        <td><b>balans_no_hold</b></td>
        <td>недостаточно средств для вывода с учетом ограничений по аккаунту</td>
    </tr>
    <tr>
        <td><b>balance_sys_no</b></td>
        <td>нет резерва для создания выплаты в данном направлении</td>
    </tr>
</table>    

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=payoutChecking&ps=1136053&sumIn=1&curIn=USD&curOut=RUB&param_ACCOUNT_NUMBER=P1000001
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":null,"outputParams":{"sumIn":1,"curIn":"USD","curOut":"RUB","ps":1136053,"sumOut":"61.47"}}
      
                
## Выплата

Выплата на любую платежную систему, которую поддерживает Payeer. Список платежных систем Вы можете посмотреть на соответствующей вкладке в настройках апи-пользователя или с помощью специального метода <code>getPaySystems</code>.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: выплата</td>
        <td>payout</td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>id выбранной платежной системы</td>
        <td>1136053</td>
    </tr>
    <tr>
        <td><b>sumIn</b></td>
        <td>сумма списания (сумма зачисления будет рассчитана автоматически с вычетом всех комиссий с получателя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>валюта, с которой будет производиться списание</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>сумма зачисления (сумма списания будет рассчитана автоматически с вычетом всех комиссий с отправителя)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>валюта зачисления (в случае отличия валюты списания от валюты зачисления конвертация будет произведена автоматически по курсу системы Payeer на момент перевода)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>param_ACCOUNT_NUMBER</b></td>
        <td>номер счета получателя в выбранной платежной системе <code>ps</code></td>
        <td>P1000441</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

### Выплата [POST]
Список возможных ошибок <code>errors</code>:

<table>
    <tr>
        <th>Ошибка</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>curIn invalid</b></td>
        <td>валюта списания не поддерживается</td>
    </tr>
    <tr>
        <td><b>curOut invalid</b></td>
        <td>валюта получения не поддерживается</td>
    </tr>
    <tr>
        <td><b>This type of exchange is not possible</b></td>
        <td>автоматический обмен валюты curIn на curOut временно запрещен</td>
    </tr>
    <tr>
        <td><b>invalid format</b></td>
        <td>неверный формат параметра</td>
    </tr>
    <tr>
        <td><b>pay_sys_no_isset</b></td>
        <td>платежное направление отключено или не существует</td>
    </tr>
    <tr>
        <td><b>cur_no_pay_sys</b></td>
        <td>несуществующая валюта получения</td>
    </tr>
    <tr>
        <td><b>sum_less_min</b></td>
        <td>сумма получения меньше минимальной</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>сумма получения больше максимальной</td>
    </tr>
    <tr>
        <td><b>output_block</b></td>
        <td>У нас возникли вопросы к Вашей деятельности. Пожалуйста, обратитесь в Службу поддержки</td>
    </tr>
    <tr>
        <td><b>balans_no</b></td>
        <td>недостаточно средств для вывода</td>
    </tr>
    <tr>
        <td><b>balans_no_hold</b></td>
        <td>недостаточно средств для вывода с учетом ограничений по аккаунту</td>
    </tr>
    <tr>
        <td><b>balance_sys_no</b></td>
        <td>нет резерва для создания выплаты в данном направлении</td>
    </tr>
</table>    

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=payout&ps=1136053&sumIn=1&curIn=USD&curOut=RUB&param_ACCOUNT_NUMBER=P1000001
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":false,"outputParams":{"sumIn":1,"curIn":"RUB","curOut":"RUB","ps":1136053,"sumOut":0},"historyId":1975716}


## Получение доступных платежных систем

Метод возвращает список доступных для вывода платежных систем.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: получение доступных платежных систем</td>
        <td>getPaySystems</td>
    </tr>
</table>

### Получение доступных платежных систем [POST]

Массив <code>list</code> содержит список платежных систем для вывода:

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>id платежной системы (который нужно использовать для Выплаты)</td>
    </tr>
    <tr>
        <td><b>name</b></td>
        <td>название платежной системы</td>
    </tr>
    <tr>
        <td><b>gate_commission</b></td>
        <td>комиссия шлюза для каждой поддерживаемой валюты</td>
    </tr>
    <tr>
        <td><b>gate_commission_min</b></td>
        <td>минимальная комиссия шлюза для каждой валюты</td>
    </tr>
    <tr>
        <td><b>gate_commission_max</b></td>
        <td>максимальная комиссия шлюза для каждой валюты</td>
    </tr>
    <tr>
        <td><b>currencies</b></td>
        <td>поддерживаемые валюты</td>
    </tr>
    <tr>
        <td><b>commission_site_percent</b></td>
        <td>комиссия Payeer в процентах</td>
    </tr>
    <tr>
        <td><b>r_fields</b></td>
        <td>список параметров для создания выплаты</td>
    </tr>
</table> 

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=getPaySystems
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"list":{"1136053":{"id":"1136053","name":"Payeer","gate_commission":[],"gate_commission_min":[],"gate_commission_max":[],"currencies":["USD","RUB","EUR"],"commission_site_percent":0,"r_fields":{"ACCOUNT_NUMBER":{"name":"\u041d\u043e\u043c\u0435\u0440 \u0441\u0447\u0435\u0442\u0430","reg_expr":"#^P[0-9]+$#","example":"P1000000"}}}}}
        

## Информация о платеже

Метод возвращает информацию о платеже.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: информация о платеже</td>
        <td>paymentDetails</td>
    </tr>
    <tr>
        <td><b>merchantId</b></td>
        <td>id мерчанта (m_shop)</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>referenceId</b></td>
        <td>id транзакции в системе учета продавца (m_orderid)</td>
        <td>117d3e31-f950-4766-a63d-5b7785aa51b6</td>
    </tr>
</table>

### Информация о платеже [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>массив ошибок</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td>items</td>
        <td>массив платажей (последние 10 платежей)</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>сумма счета (m_amount)</td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>currency</b></td>
        <td>валюта счета (m_curr)</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>description</b></td>
        <td>описание (m_desc)</td>
        <td>Test payment</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>referenceId</b></td>
        <td>id счета в системе учета продавца (m_orderid)</td>
        <td>117d3e31-f950-4766-a63d-5b7785aa51b6</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>created</b></td>
        <td>дата создания счета</td>
        <td>2020-01-20 14:05:30</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>processed</b></td>
        <td>дата оплаты счета</td>
        <td>2020-01-20 14:05:48</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>paySystemName</b></td>
        <td>название платежного метода</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>paySystemId</b></td>
        <td>id платежного метода</td>
        <td>2609</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientAmount</b></td>
        <td>сумма, оплачиваемая клиентом</td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientCurrency</b></td>
        <td>валюта</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientAccount</b></td>
        <td>номер Payeer кошелька плательщика</td>
        <td>P1000001</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientEmail</b></td>
        <td>email плательщика</td>
        <td>client@payeer.com</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitAmount</b></td>
        <td>сумма зачисления на счет мерчанта</td>
        <td>0.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitCurrency</b></td>
        <td>валюта зачисления</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitTransactionId</b></td>
        <td>id транзакции зачисления</td>
        <td>928562504</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantTransactionId</b></td>
        <td>id транзакции оплаты</td>
        <td>928562360</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>created_ts</b></td>
        <td>дата создания счета в формате unixtime</td>
        <td>1579518330</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>processed_ts</b></td>
        <td>дата оплаты счета в формате unixtime</td>
        <td>1579518348</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantFee</b></td>
        <td>сумма комиссии мерчанта</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantCurrency</b></td>
        <td>валюта комиссии мерчанта</td>
        <td>USD</td>
    </tr>
</table>

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=paymentDetails&merchantId=12345&referenceId=117d3e31-f950-4766-a63d-5b7785aa51b6
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"success":true,"items":[{"amount":1,"currency":"USD","description":"Test payment","referenceId":"117d3e31-f950-4766-a63d-5b7785aa51b6","created":"2020-01-20 14:05:30","processed":"2020-01-20 14:05:48","paySystemName":"Payeer","paySystemId":"2609","clientAmount":1,"clientCurrency":"USD","clientAccount":"P1000001","clientEmail":"client@payeer.com","profitAmount":0.99,"profitCurrency":"USD","profitTransactionId":"928562504","merchantTransactionId":"928562360","created_ts":1579518330,"processed_ts":1579518348,"merchantFee":0.01,"merchantCurrency":"USD"}]}


## История операций

Получение истории операций по счету.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: получение истории операций по счету</td>
        <td>history</td>
    </tr>
    <tr>
        <td>sort</td>
        <td>сортировка по дате (по-умолчанию desc)</td>
        <td>asc, desc</td>
    </tr>
    <tr>
        <td>count</td>
        <td>количество возвращаемых записей (максимум 1000)</td>
        <td>10</td>
    </tr>
    <tr>
        <td>from</td>
        <td>начало периода</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td>to</td>
        <td>конец периода</td>
        <td>2016-03-31 13:47:06</td>
    </tr>
    <tr>
        <td>type</td>
        <td>тип операций</td>
        <td><b>incoming</b> - входящие платежи<br><b>outgoing</b> - исходящие платежи</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id предыдущей операции</td>
        <td>197941396</td>
    </tr>
    <tr>
        <td>id</td>
        <td>получение истории по id транзакции</td>
        <td>197941396</td>
    </tr>
</table>

### История операций [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>id операции</td>
        <td>197941397</td>
    </tr>
    <tr>
        <td><b>date</b></td>
        <td>дата/время создания в формате ГГГГ-ММ-ДД ЧЧ:ММ:СС</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>тип операции</td>
        <td><b>transfer</b> - перевод<br><b>deposit</b> - пополнение<br><b>withdrawal</b> - вывод<br><b>sci</b> - оплата в магазин</td>
    </tr>
    <tr>
        <td><b>status</b></td>
        <td>статус операции</td>
        <td><b>success</b> - выполнен (конечный статус)<br><b>processing</b> - в процессе выполнения (изменится на success, canceled или pending)<br><b>canceled</b> - отменен (конечный статус)<br><b>waiting</b> - в ожидании (например в ожидании оплаты) (изменится на success, canceled или pending)<br><b>pending</b> - приостановлен (изменится на success, canceled)</td>
    </tr>
    <tr>
        <td><b>from</b></td>
        <td>номер счета отправителя</td>
        <td>P1000000, @anonum, @merchant, null</td>
    </tr>
    <tr>
        <td><b>debitedAmount</b></td>
        <td>сумма списания</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>debitedCurrency</b></td>
        <td>валюта списания</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>номер счета получателя</td>
        <td>P1000001, @sms, null</td>
    </tr>
    <tr>
        <td><b>creditedAmount</b></td>
        <td>сумма получения</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>creditedCurrency</b></td>
        <td>валюта получения</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>payeerFee</b></td>
        <td>комиссия Payeer</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>gateFee</b></td>
        <td>комиссия шлюза</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>comment</b></td>
        <td>комметарий</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>paySystem</b></td>
        <td>наименование платежного метода</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td><b>shopId</b></td>
        <td>id магазина, по которому произошло зачисление</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>shopOrderId</b></td>
        <td>id номера счета в системе учета продавца</td>
        <td>123</td>
    </tr>
    <tr>
        <td><b>exchangeRate</b></td>
        <td>курс конвертации</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>protect</b></td>
        <td>протекция сделки</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>protectDay</b></td>
        <td>количество дней протекции сделки</td>
        <td>1-30, null</td>
    </tr>
    <tr>
        <td><b>isApi</b></td>
        <td>операция совершена через API</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>shop</b></td>
        <td>массив параметров магазина, в который произведена оплата через sci</td>
        <td><b>id</b> - id магазина<br><b>domain</b> - домен магазина<br><b>orderid</b> - номер счета в системе учета продавца<br><b>description</b> - описание счета</td>
    </tr>
    <tr>
        <td>apiId</td>
        <td>ID API-пользователя, через которого была совершена транзакция</td>
        <td>12345</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=history
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"params":{},"history":{}}
            

## Информация по выплате

Получение информации по выплате с использованием Reference ID.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: получение информации по выплате</td>
        <td>payoutDetails</td>
    </tr>
    <tr>
        <td><b>referenceId</b></td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
    <tr>
        <td>referenceApiId</td>
        <td>id апи-пользователя, через которого производилась выплата referenceId (по-умолчанию используется apiId)</td>
        <td>123456</td>
    </tr>
</table>

### Информация по выплате [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>id операции</td>
        <td>197941397</td>
    </tr>
    <tr>
        <td><b>date</b></td>
        <td>дата/время создания в формате ГГГГ-ММ-ДД ЧЧ:ММ:СС</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>тип операции</td>
        <td><b>transfer</b> - перевод<br><b>withdrawal</b> - вывод</td>
    </tr>
    <tr>
        <td><b>status</b></td>
        <td>статус операции</td>
        <td><b>success</b> - выполнен (конечный статус)<br><b>processing</b> - в процессе выполнения (изменится на success, canceled или pending)<br><b>canceled</b> - отменен (конечный статус)</td>
    </tr>
    <tr>
        <td><b>from</b></td>
        <td>номер счета отправителя</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>debitedAmount</b></td>
        <td>сумма списания</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>debitedCurrency</b></td>
        <td>валюта списания</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>номер счета получателя</td>
        <td>P1000001</td>
    </tr>
    <tr>
        <td><b>creditedAmount</b></td>
        <td>сумма получения</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>creditedCurrency</b></td>
        <td>валюта получения</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>payeerFee</b></td>
        <td>комиссия Payeer</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>gateFee</b></td>
        <td>комиссия шлюза</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>comment</b></td>
        <td>комметарий</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>paySystem</b></td>
        <td>наименование платежного метода</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td><b>exchangeRate</b></td>
        <td>курс конвертации</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>protect</b></td>
        <td>протекция сделки</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>protectDay</b></td>
        <td>количество дней протекции сделки</td>
        <td>1-30, null</td>
    </tr>
    <tr>
        <td><b>isApi</b></td>
        <td>операция совершена через API</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>ID API-пользователя, через которого была совершена транзакция</td>
        <td>12345</td>
    </tr>
    <tr>
        <td>referenceId</b></td>
        <td>идентификационный номер в вашей системе учета</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=payoutDetails&referenceId=08d7f228-5a03-4dc7-b13a-7c14146df8f7
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"params":{},"history":{}}


## Создание счета для оплаты

Создание счета для оплаты.

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: создания счета</td>
        <td>invoiceCreate</td>
    </tr>
    <tr>
        <td><b>m_shop</b></td>
        <td>id мерчанта</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>m_orderid</b></td>
        <td>id счета в системе учета продавца</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>m_amount</b></td>
        <td>сумма счета</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>m_curr</b></td>
        <td>валюта счета</td>
        <td>USD</td>
    </tr>
    <tr>
        <td><b>m_desc</b></td>
        <td>описание счета</td>
        <td>Test Invoice</td>
    </tr>
</table>

### Создание счета для оплаты [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>массив ошибок</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>url</td>
        <td>ссылка для оплаты</td>
    </tr>
</table> 

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=invoiceCreate&m_shop=12345&m_orderid=test&m_amount=10&m_curr=USD&m_desc=Test%20Invoice
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"success":true,"url":"https:\/\/payeer.com\/merchant\/?m_shop=12345&m_orderid=test&m_amount=10.00&m_curr=USD&m_desc=VGVzdCBJbnZvaWNl&m_sign=123FB8F0B661B63F30B673443582D855447E62E90631988811AC9F4F147BCE97"}

                                            
## API Merchant

С помощью данного метода можно автоматически создать счет для оплаты в нужной платежной системе.

<b>Метод включается только магазинам с высоким оборотом по предварительной заявке</b>

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
        <th>Пример</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Ваш номер счета в системе Payeer</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>id апи-пользователя, выдается при добавлении API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>секретный ключ апи-пользователя</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>тип действия: выставление счета по API</td>
        <td>merchant</td>
    </tr>
    <tr>
        <td>processUrl</td>
        <td>url для переадресации к оплате счета</td>
        <td>https://payeer.com/api/merchant/process.php</td>
    </tr>
    <tr>
        <td>merchantUrl</td>
        <td>url ожидания оплаты</td>
        <td>https://payeer.com/merchant/</td>
    </tr>
    <tr>
        <td>lang</td>
        <td>язык интерфейса</td>
        <td>ru, en</td>
    </tr>
    <tr>
        <td><b>shop</b></td>
        <td>json данные</td>
        <td>
            <b>m_shop</b> - id магазина<br>
            <b>m_orderid</b> - id счета в Вашей системе учета<br>
            <b>m_amount</b> - сумма счета<br>
            <b>m_curr</b> - валюта счета<br>
            <b>m_desc</b> - комментарий к счету закодированный base64<br>
            m_params - массив дополнительных параметров<br>
            <b>m_sign</b> - подпись
        </td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>json данные</td>
        <td>
            <b>id</b> - id платежной системы, который может посмотреть в настройках магазина на вкладке "Внешний вид"<br>
            <b>curr</b> - валюта платежной системы
        </td>
    </tr>
    <tr>
        <td><b>form</b></td>
        <td>json данные</td>
        <td>
            <b>order_email</b> - email клиента, который будет оплачивать счет
        </td>
    </tr>
    <tr>
        <td>ip</td>
        <td>ip клиента</td>
        <td>8.8.8.8</td>
    </tr>
</table>

Массив дополнительных параметров (m_params):
<table>
    <tr>
        <th>Название</th>
        <th>Ключ</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td>Success URL</td>
        <td><i>success_url</i></td>
        <td>на данный адрес клиент будет перенаправлен после успешной оплаты</td>
    </tr>
    <tr>
        <td>Fail URL</td>
        <td><i>fail_url</i></td>
        <td>этот адрес используется для перенаправления в случае ошибки в процессе оплаты или отмены платежа</td>
    </tr>
    <tr>
        <td>Status URL</td>
        <td><i>status_url</i></td>
        <td>адрес обработчика платежа, на данной странице заказ должен помечаться как оплаченный или, например, происходить зачисление денег на счет клиента на Вашем сайте</td>
    </tr>
    <tr>
        <td>Дополнительные поля</td>
        <td><i>reference</i></td>
        <td>Массив дополнительных полей, которые необходимо вернуть на обработчик платежа</td>
    </tr>
    <tr>
        <td>Домен сабмерчанта</td>
        <td><i>submerchant</i></td>
        <td>Домен сабмерчанта (только для агрегаторов)</td>
    </tr>
</table>    

### API Merchant [POST]

<table>
    <tr>
        <th>Параметр</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>location</b></td>
        <td>url для переадресации пользователя для оплаты или ожидания оплаты</td>
    </tr>
    <tr>
        <td><b>orderid</b></td>
        <td>номер счета в системе учета Payeer</td>
    </tr>
    <tr>
        <td><b>historyid</b></td>
        <td>номер операции в системе учета Payeer</td>
    </tr>
    <tr>
        <td><b>historytm</b></td>
        <td>timestamp создания операции в Payeer</td>
    </tr>
</table>

Список возможных ошибок <code>errors</code>:

<table>
    <tr>
        <th>Ошибка</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><b>Shop API is disabled</b></td>
        <td>апи-мерчант для указанного магазина выключен (по-умолчанию для всех магазинов)</td>
    </tr>
    <tr>
        <td><b>The payment system is forbidden</b></td>
        <td>выбранная платежная система недоступна для данного магазина</td>
    </tr>
    <tr>
        <td><b>PS currency is not supported</b></td>
        <td>указанная валюта не поддерживается для выбранной ПС</td>
    </tr>
    <tr>
        <td><b>Currency is not supported</b></td>
        <td>валюта не поддерживается Payeer</td>
    </tr>
    <tr>
        <td><b>Amount is too large</b></td>
        <td>сумма счета слишком велика</td>
    </tr>
    <tr>
        <td><b>Summa less min</b></td>
        <td>сумма оплаты меньше минимальной</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>сумма больше максимальной для выбранной ПС</td>
    </tr>
    <tr>
        <td><b>Invalid sign</b></td>
        <td>неверно сформирована подпись m_sign</td>
    </tr>
    <tr>
        <td><b>field_required</b></td>
        <td>указанное поле обязательно</td>
    </tr>
    <tr>
        <td><b>field_invalid_regexp</b></td>
        <td>указанное поле несоответствует регулярному выражению</td>
    </tr>
</table>    

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwertyaction=merchant&lang=ru&shop={"m_shop":"1234567","m_orderid":"12345","m_amount":"1.00","m_curr":"USD","m_desc":"VGVzdA==","m_sign":"2F1C3BF2B64229E59AAA4CDE746DC8E99E6E001AA721C07269832B9EDF9DBDD8"}&ps={"id":"2609","curr":"USD"}&form={"order_email":"test@payeer.com"}
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"location":"https:\/\/payeer.com\/api\/merchant\/process.php?m_historyid=1980775&m_historytm=1000919709&m_curorderid=254431&lang=ru&referer=https%3A%2F%2Fpayeer.com%2Fmerchant%2F%3Fm_historyid%3D1980775%26m_historytm%3D1000919709%26m_curorderid%3D254431%26lang%3Dru","orderid":254431,"historyid":1980775,"historytm":1000919709}

            

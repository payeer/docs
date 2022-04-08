# Payeer API (en)

API URL: https://payeer.com/ajax/api/api.php

* [Authorization Check](#authorization-check)
* [Balance Check](#balance-check)
* [Transferring Funds](#transferring-funds)
* [Checking Existence of Account](#checking-existence-of-account)
* [Automatic Conversion Rates](#automatic-conversion-rates)
* [Checking Possibility of Payout](#checking-possibility-of-payout)
* [Payout](#payout)
* [Getting Available Payment Systems](#getting-available-payment-systems)
* [Payment details](#payment-details)
* [History of transactions](#history-of-transactions)
* [Payout details](#payout-details)
* [Creating an invoice for payment](#creating-an-invoice-for-payment)
* [Merchant API](#merchant-api)

Payeer provides external developers with options for interacting with the service via software. One such interaction method is the use of an API. 
The API determines a set of functions for facilitating interaction on HTTPs protocols. Using these functions, it is possible to gain access to various system resources such as information about payout methods in the Payeer system, exchange rates, or information on specific payments in the system, as well as to generate a withdrawal. 
This interaction is performed using POST queries in UTF-8 encoding to the URL: https://payeer.com/ajax/api/api.php. 
Each response from the Payeer server must contain the <code>auth_error</code> field, which shows whether or not the authentication parameters have been entered correctly. 
The response to the query will also contain the array <code>errors</code>, which refers to the presence of errors when fulfilling the query. An "errors" field that is blank or contains the values "false", "null", or "0" indicates that there were no errors whatsoever when fulfilling the query; otherwise a text description of the error that occurred is returned.

## Starting to Work with the API
In order to work with the API, you must in the profile at https://payeer.com/en/account/?tab=api click on the button "Activation" in the right column.

Enter the name of the user, generate a secret key for working with the API, and, in order to provide additional security, enter the IP address from which you will be sending queries to the Payeer server.

Warning! Never share your secret API key with anyone.

Once you have added the API you will receive a user ID that you will have to use along with a key and an account number for authentication in queries.

## Frequently Asked Questions
* When I send a query to the API I get this error message: "Auth error: account or apiId or apiPass is incorrect or api-user was blocked." 
<br><br><code>Check to make sure your account number, API user ID, and secret key have been entered correctly and that the user has not been blocked in the "Mass Payouts" settings</code>

* I can’t get rid of the the error message "IP 1.2.3.4 does not satisfy the security settings." 
<br><br><code>Enter the IP that was returned in the error message in the "Mass Payouts" settings and repeat the query.</code>

* After a while the error message "Auth error: account or apiId or apiPass is incorrect or api-user was blocked” appears again, although no settings have been changed. 
<br><br><code>After a certain number of login attempts with an invalid secret key an API user will be blocked. The user can be unblocked in the "Mass Payouts" settings.  It is possible that you have changed the secret key everywhere on your website or that someone is attempting to acquire the password to your API. <br>1) Delete the old API user <br>2) Create a new one with IP protection<br>3) Register the new access information on your website</code>

* Someone hacked my website and withdrew all the money through the API 
<br><br><code>For additional security you can use a second account for payouts where you keep the amount necessary for short-term transfers. Please do not neglect the performance of safety checks for your website.</code>

## Authorization Check
The simplest query is checking authorization. It does not have to be fulfilled separately from primary actions.

To check authorization, send a POST query to the URL https://payeer.com/ajax/api/api.php with three parameters:
<code>account</code>, <code>apiId</code> и <code>apiPass</code>:

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>the API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>the API user's secret key</td>
        <td>qwerty</td>
    </tr>
</table>

### Authorization Check [POST]

Two parameters will be returned in the body of the response: <code>auth_error</code> и <code>errors</code>:

<table>
    <tr>
        <th>auth_error</th>
        <th>errors</th>
        <th></th>
    </tr>
    <tr>
        <td><b>0</b></td>
        <td></td>
        <td><b>Authorization successful</b></td>
    </tr>
    <tr>
        <td><b>1</b></td>
        <td>Auth error: account or apiId or apiPass is incorrect or api-user was blocked</td>
        <td><b>Authorization error</b><br>Account number, apiID, or apiPass entered incorrectly or the API user has been blocked in the settings</td>
    </tr>
    <tr>
        <td><b>1</b></td>
        <td>IP 1.2.3.4 does not satisfy the security settings</td>
        <td><b>Authorization error</b><br>Queries come from the IP 1.2.3.4, although another one has been registered in the API user's settings</td>
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

## Balance Check

Getting a wallet balance.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: balance check</td>
        <td>getBalance</td>
    </tr>
    <tr>
        <td>type</td>
        <td>return type</td>
        <td>object (default), array</td>
    </tr>
</table>

### Balance Check [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>currency</b></td>
        <td>currency code</td>
    </tr>
    <tr>
        <td><b>total</b></td>
        <td>total balance</td>
    </tr>
    <tr>
        <td><b>available</b></td>
        <td>available balance for payouts</td>
    </tr>
    <tr>
        <td><b>hold</b></td>
        <td>amount of frozen funds</td>
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
        
## Transferring Funds

Transfer funds within the Payeer system.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: transfer</td>
        <td>transfer</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>currency with which the withdrawal will be performed</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>sum</b></td>
        <td>amount withdrawn (the amount deposited will be calculated automatically, factoring in all fees from the recipient)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>deposit currency (if the withdrawal currency is different from the deposit currency, the conversion will be performed automatically based on the Payeer system’s exchange rate when the transfer takes place)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>amount deposited (the amount withdrawn will be calculated automatically, factoring in all fees from the sender)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>user’s Payeer account number or email address (if the email address was not registered before the transfer, the registration will be performed automatically)</td>
        <td>P1000000, test@mail.com</td>
    </tr>
    <tr>
        <td>comment</td>
        <td>comments on the transfer (this should preferably show the transaction ID in your accounting system)</td>
        <td>Transfer #1365</td>
    </tr>
    <tr>
        <td>protect</td>
        <td>activation of transaction protection (to have the transfer deposited into the recipient’s account you will have to enter the protectCode protection code within protectCode days; otherwise the transfer will be canceled and the funds will remain in the sender's account)</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>protectPeriod</td>
        <td>protection period: 1–30 days</td>
        <td>1–30</td>
    </tr>
    <tr>
        <td>protectCode</td>
        <td>protection code</td>
        <td>12345</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>identification number in your accounting system</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

required parameters are in <b>bold</b>

### Transferring Funds [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>array of errors or false</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>historyId</td>
        <td>transaction number</td>
    </tr>
</table> 

List of potential <code>errors</code>:

<table>
    <tr>
        <th>Error</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>balanceError</b></td>
        <td>insufficient funds for transfer</td>
    </tr>
    <tr>
        <td><b>balanceError000</b></td>
        <td>insufficient funds for transfer given limitations on the account</td>
    </tr>
    <tr>
        <td><b>transferHimselfForbidden</b></td>
        <td>you cannot transfer funds to yourself (when the withdrawal currency and deposit currency are the same)</td>
    </tr>
    <tr>
        <td><b>transferError</b></td>
        <td>transfer error, try again later</td>
    </tr>
    <tr>
        <td><b>convertForbidden</b></td>
        <td>automatic exchange of curIn for curOut is temporarily prohibited</td>
    </tr>
    <tr>
        <td><b>protectDay_1_30</b></td>
        <td>error entering transfer protection period</td>
    </tr>
    <tr>
        <td><b>protectCodeNotEmpty</b></td>
        <td>the protection code cannot be blank when protection is active</td>
    </tr>
    <tr>
        <td><b>sumNotNull</b></td>
        <td>the amounts sent and received cannot be zero</td>
    </tr>
    <tr>
        <td><b>sumInNotMinus</b></td>
        <td>the amount sent cannot be negative</td>
    </tr>
    <tr>
        <td><b>sumOutNotMinus</b></td>
        <td>the amount received cannot be negative</td>
    </tr>
    <tr>
        <td><b>fromError</b></td>
        <td>sender not found</td>
    </tr>
    <tr>
        <td><b>toError</b></td>
        <td>recipient entered incorrectly</td>
    </tr>
    <tr>
        <td><b>curInError</b></td>
        <td>withdrawal currency not supported</td>
    </tr>
    <tr>
        <td><b>curOutError</b></td>
        <td>reception currency not supported</td>
    </tr>
    <tr>
        <td><b>outputHold</b></td>
        <td>Issues have arisen regarding your activity. Please contact Technical Support</td>
    </tr>
    <tr>
        <td><b>transferToForbiddenCountry</b></td>
        <td>Transfer to customers in certain countries is prohibited</td>
    </tr>
</table> 

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=transfer&curIn=USD&sum=1&curOut=USD&to=P1000001
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":false,"success":true,"historyId":925947210}
        

## Checking Existence of Account

You can check the existence of an account number prior to transfer in the Payeer system..

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: check existence of account</td>
        <td>checkUser</td>
    </tr>
    <tr>
        <td><b>user</b></td>
        <td>user’s account number in the format P1000000</td>
        <td>P1000000</td>
    </tr>
</table>

### Checking Existence of Account [POST]

If the <code>errors</code> array is blank, this means that the user exists.

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=checkUser&user=P1000441
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[]}
        

## Automatic Conversion Rates

If during deposit/transfer/withdrawal the withdrawal currency <code>curIn</code> is different from the receiving currency <code>curOut</code>, the amount will be converted based on Payeer’s internal echange rates, which can be obtained using the following method.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: obtain automatic conversion rates</td>
        <td>getExchangeRate</td>
    </tr>
    <tr>
        <td><b>output</b></td>
        <td>select currencies for conversion rates</td>
        <td><b>N</b> - get deposit rates<br><b>Y</b> - get withdrawal rates</td>
    </tr>
</table>

### Automatic Conversion Rates [POST]

+ Request
    
    + Headers

            Content-Type: application/x-www-form-urlencoded

    + Body
    
            account=P1000000&apiId=12345&apiPass=qwerty&action=getExchangeRate&output=Y
            
+ Response 200 (application/json)

    + Body
    
            {"auth_error":"0","errors":[],"rate":{"RUB\/USD":"0.01477541","RUB\/RUB":1,"RUB\/EUR":"0.01308525","USD\/USD":1,"USD\/RUB":"61.47230000","USD\/EUR":"0.85757203","EUR\/USD":"1.09716731","EUR\/RUB":"69.53140000"}}
        
## Checking Possibility of Payout

This method allows you to check the possibility of a payout without actually creating a payout (you can get the withdrawal/reception amount or check errors in parameters)

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: checking possibility of payout</td>
        <td>payoutChecking</td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>ID of selected payment system</td>
        <td>1136053</td>
    </tr>
    <tr>
        <td><b>sumIn</b></td>
        <td>amount withdrawn (the amount deposited will be calculated automatically, factoring in all fees from the recipient)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>currency with which the withdrawal will be performed</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>amount deposited (the amount withdrawn will be calculated automatically, factoring in all fees from the sender)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>deposit currency (if the withdrawal currency is different from the deposit currency, the conversion will be performed automatically based on the Payeer system’s exchange rate when the transfer takes place)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>param_ACCOUNT_NUMBER</b></td>
        <td>recipient's account number in the selected payment system <code>ps</code></td>
        <td>P1000441</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>identification number in your accounting system</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

### Checking Possibility of Payout [POST]
List of potential <code>errors</code>:

<table>
    <tr>
        <th>Error</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>curIn invalid</b></td>
        <td>withdrawal currency not supported</td>
    </tr>
    <tr>
        <td><b>curOut invalid</b></td>
        <td>deposit currency not supported</td>
    </tr>
    <tr>
        <td><b>This type of exchange is not possible</b></td>
        <td>automatic exchange of curIn for curOut is temporarily prohibited</td>
    </tr>
    <tr>
        <td><b>invalid format</b></td>
        <td>invalid parameter format</td>
    </tr>
    <tr>
        <td><b>pay_sys_no_isset</b></td>
        <td>payout method is disabled or does not exist</td>
    </tr>
    <tr>
        <td><b>cur_no_pay_sys</b></td>
        <td>non-existent deposit currency</td>
    </tr>
    <tr>
        <td><b>sum_less_min</b></td>
        <td>amount received is below the minimum</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>amount received is above the maximum</td>
    </tr>
    <tr>
        <td><b>output_block</b></td>
        <td>Issues have arisen regarding your activity. Please contact Technical Support</td>
    </tr>
    <tr>
        <td><b>balans_no</b></td>
        <td>insufficient funds for withdrawal</td>
    </tr>
    <tr>
        <td><b>balans_no_hold</b></td>
        <td>insufficient funds for withdrawal given limitations on the account</td>
    </tr>
    <tr>
        <td><b>balance_sys_no</b></td>
        <td>no reserve for the creation of a payout</td>
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
      
                
## Payout

A payout to any payment system that supports Payeer. You can find a list of payment systems on the corresponding tab in the API user settings or using the special <code>getPaySystems</code> method.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: payout</td>
        <td>payout</td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>ID of selected payment system</td>
        <td>1136053</td>
    </tr>
    <tr>
        <td><b>sumIn</b></td>
        <td>amount withdrawn (the amount deposited will be calculated automatically, factoring in all fees from the recipient)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curIn</b></td>
        <td>currency with which the withdrawal will be performed</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td>sumOut</td>
        <td>amount deposited (the amount withdrawn will be calculated automatically, factoring in all fees from the sender)</td>
        <td>1.00</td>
    </tr>
    <tr>
        <td><b>curOut</b></td>
        <td>deposit currency (if the withdrawal currency is different from the deposit currency, the conversion will be performed automatically based on the Payeer system’s exchange rate when the transfer takes place)</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>param_ACCOUNT_NUMBER</b></td>
        <td>recipient's account number in the selected payment system <code>ps</code></td>
        <td>P1000441</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>identification number in your accounting system</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
</table>

### Payout [POST]

List of potential <code>errors</code>:

<table>
    <tr>
        <th>Error</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>curIn invalid</b></td>
        <td>withdrawal currency not supported</td>
    </tr>
    <tr>
        <td><b>curOut invalid</b></td>
        <td>deposit currency not supported</td>
    </tr>
    <tr>
        <td><b>This type of exchange is not possible</b></td>
        <td>automatic exchange of curIn for curOut is temporarily prohibited</td>
    </tr>
    <tr>
        <td><b>invalid format</b></td>
        <td>invalid parameter format</td>
    </tr>
    <tr>
        <td><b>pay_sys_no_isset</b></td>
        <td>payout method is disabled or does not exist</td>
    </tr>
    <tr>
        <td><b>cur_no_pay_sys</b></td>
        <td>non-existent deposit currency</td>
    </tr>
    <tr>
        <td><b>sum_less_min</b></td>
        <td>amount received is below the minimum</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>amount received is above the maximum</td>
    </tr>
    <tr>
        <td><b>output_block</b></td>
        <td>Issues have arisen regarding your activity. Please contact Technical Support</td>
    </tr>
    <tr>
        <td><b>balans_no</b></td>
        <td>insufficient funds for withdrawal</td>
    </tr>
    <tr>
        <td><b>balans_no_hold</b></td>
        <td>insufficient funds for withdrawal given limitations on the account</td>
    </tr>
    <tr>
        <td><b>balance_sys_no</b></td>
        <td>no reserve for the creation of a payout</td>
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


## Getting Available Payment Systems

This method returns a list of payment systems that are available for payout.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: get available payment systems</td>
        <td>getPaySystems</td>
    </tr>
</table>

### Getting Available Payment Systems [POST]

The <code>list</code> array contains a list of payment systems for payout:

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>payment system ID (which is required to use for payout)</td>
    </tr>
    <tr>
        <td><b>name</b></td>
        <td>payment system name</td>
    </tr>
    <tr>
        <td><b>gate_commission</b></td>
        <td>gateway fee for each supported currency</td>
    </tr>
    <tr>
        <td><b>gate_commission_min</b></td>
        <td>minimum gateway fee for each currency</td>
    </tr>
    <tr>
        <td><b>gate_commission_max</b></td>
        <td>maximum gateway fee for each currency</td>
    </tr>
    <tr>
        <td><b>currencies</b></td>
        <td>supported currencies</td>
    </tr>
    <tr>
        <td><b>commission_site_percent</b></td>
        <td>Payeer’s fee in percent</td>
    </tr>
    <tr>
        <td><b>r_fields</b></td>
        <td>list of parameters for creating a payout</td>
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
        


## Payment details

Payment details.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: payment details</td>
        <td>paymentDetails</td>
    </tr>
    <tr>
        <td><b>merchantId</b></td>
        <td>merchant id (m_shop)</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>referenceId</b></td>
        <td>invoice id in the seller's accounting system (m_orderid)</td>
        <td>117d3e31-f950-4766-a63d-5b7785aa51b6</td>
    </tr>
</table>

### Payment details [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>array of errors</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td>items</td>
        <td>array of payments (last 10 payments)</td>
        <td>&nbsp;</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>amount</b></td>
        <td>invoice amount (m_amount)</td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>currency</b></td>
        <td>invoice currency (m_curr)</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>description</b></td>
        <td>description (m_desc)</td>
        <td>Test payment</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>referenceId</b></td>
        <td>invoice id in the seller's accounting system (m_orderid)</td>
        <td>117d3e31-f950-4766-a63d-5b7785aa51b6</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>created</b></td>
        <td>creation date</td>
        <td>2020-01-20 14:05:30</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>processed</b></td>
        <td>payment date</td>
        <td>2020-01-20 14:05:48</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>paySystemName</b></td>
        <td>name of the payment method</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>paySystemId</b></td>
        <td>id of the payment method</td>
        <td>2609</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientAmount</b></td>
        <td>amount to be paid by the client</td>
        <td>1</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientCurrency</b></td>
        <td>currency</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientAccount</b></td>
        <td>Payeer number of the payer's wallet</td>
        <td>P1000001</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>clientEmail</b></td>
        <td>payer's email address</td>
        <td>client@payeer.com</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitAmount</b></td>
        <td>amount credited to the merchant's account</td>
        <td>0.99</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitCurrency</b></td>
        <td>currency</td>
        <td>USD</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>profitTransactionId</b></td>
        <td>profit transaction id</td>
        <td>928562504</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantTransactionId</b></td>
        <td>payment transaction id</td>
        <td>928562360</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>created_ts</b></td>
        <td>creation date in unixtime format</td>
        <td>1579518330</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>processed_ts</b></td>
        <td>payment date in unixtime format</td>
        <td>1579518348</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantFee</b></td>
        <td>amount of the merchant fee</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>&nbsp;&nbsp;&nbsp;<b>merchantCurrency</b></td>
        <td>currency of the merchant fee</td>
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


## History of transactions

Get the history of account transactions.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: get the history of account transactions</td>
        <td>history</td>
    </tr>
    <tr>
        <td>sort</td>
        <td>sorting by date (default desc)</td>
        <td>asc, desc</td>
    </tr>
    <tr>
        <td>count</td>
        <td>count of records (max 1000)</td>
        <td>10</td>
    </tr>
    <tr>
        <td>from</td>
        <td>begin of the period</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td>to</td>
        <td>end of the period</td>
        <td>2016-03-31 13:47:06</td>
    </tr>
    <tr>
        <td>type</td>
        <td>transaction type</td>
        <td><b>incoming</b> - incoming payments<br><b>outgoing</b> - outgoing payments</td>
    </tr>
    <tr>
        <td>append</td>
        <td>id of the previous transaction</td>
        <td>197941396</td>
    </tr>
    <tr>
        <td>id</td>
        <td>getting history by transaction id</td>
        <td>197941396</td>
    </tr>
</table>

### History of transactions [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>transaction ID</td>
        <td>197941397</td>
    </tr>
    <tr>
        <td><b>date</b></td>
        <td>Date/time of creation in the format YYYY-MM-DD HH:MM:SS</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>transaction type</td>
        <td><b>transfer</b> - transfer<br><b>deposit</b> - deposit<br><b>withdrawal</b> - withdrawal<br><b>sci</b> - payment to the store</td>
    </tr>
    <tr>
        <td><b>status</b></td>
        <td>transaction status</td>
        <td><b>success</b> - complete (final status)<br><b>processing</b> - in progress (changes to success, canceled, or pending)<br><b>canceled</b> - canceled (final status)<br><b>waiting</b> - expected (for example, when waiting for a payment) (changes to success, canceled, or pending)<br><b>pending</b> - pending (changes to success, canceled)</td>
    </tr>
    <tr>
        <td><b>from</b></td>
        <td>sender's account number</td>
        <td>P1000000, @anonum, @merchant, null</td>
    </tr>
    <tr>
        <td><b>debitedAmount</b></td>
        <td>amount withdrawn</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>debitedCurrency</b></td>
        <td>withdrawal currency</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>recipient's account number</td>
        <td>P1000001, @sms, null</td>
    </tr>
    <tr>
        <td><b>creditedAmount</b></td>
        <td>amount received</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>creditedCurrency</b></td>
        <td>deposit currency</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>payeerFee</b></td>
        <td>Payeer’s fee</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>gateFee</b></td>
        <td>gateway fee</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>comment</b></td>
        <td>comments</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>paySystem</b></td>
        <td>name payment method</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td><b>shopId</b></td>
        <td>merchant id</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>shopOrderId</b></td>
        <td>id invoice of merchant</td>
        <td>123</td>
    </tr>
    <tr>
        <td><b>exchangeRate</b></td>
        <td>exchange rate for auto-conversion</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>protect</b></td>
        <td>transaction protection for transfer</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>protectDay</b></td>
        <td>number of days for protection</td>
        <td>1-30, null</td>
    </tr>
    <tr>
        <td><b>isApi</b></td>
        <td>via API</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>shop</b></td>
        <td>sci parameters</td>
        <td><b>id</b> - store id<br><b>domain</b> - domain store<br><b>orderid</b> - invoice id of store<br><b>description</b> - description</td>
    </tr>
    <tr>
        <td>apiId</td>
        <td>API user ID the transaction was made through</td>
        <td>12345</td>
    </tr>
    <tr>
        <td>referenceId</td>
        <td>identification number in your accounting system</td>
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


## Payout details

Getting payout information using Reference ID.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: getting payout information</td>
        <td>payoutDetails</td>
    </tr>
    <tr>
        <td><b>referenceId</b></td>
        <td>identification number in your accounting system</td>
        <td>08d7f228-5a03-4dc7-b13a-7c14146df8f7</td>
    </tr>
    <tr>
        <td>referenceApiId</td>
        <td>id of the api-user (apiId is used by default)</td>
        <td>123456</td>
    </tr>
</table>

### Payout details [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>id</b></td>
        <td>transaction ID</td>
        <td>197941397</td>
    </tr>
    <tr>
        <td><b>date</b></td>
        <td>Date/time of creation in the format YYYY-MM-DD HH:MM:SS</td>
        <td>2016-03-02 15:35:17</td>
    </tr>
    <tr>
        <td><b>type</b></td>
        <td>transaction type</td>
        <td><b>transfer</b> - transfer<br><b>withdrawal</b> - withdrawal</td>
    </tr>
    <tr>
        <td><b>status</b></td>
        <td>transaction status</td>
        <td><b>success</b> - complete (final status)<br><b>processing</b> - in progress (changes to success, canceled, or pending)<br><b>canceled</b> - canceled (final status)</td>
    </tr>
    <tr>
        <td><b>from</b></td>
        <td>sender's account number</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>debitedAmount</b></td>
        <td>amount withdrawn</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>debitedCurrency</b></td>
        <td>withdrawal currency</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>to</b></td>
        <td>recipient's account number</td>
        <td>P1000001</td>
    </tr>
    <tr>
        <td><b>creditedAmount</b></td>
        <td>amount received</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>creditedCurrency</b></td>
        <td>deposit currency</td>
        <td>USD, EUR, RUB</td>
    </tr>
    <tr>
        <td><b>payeerFee</b></td>
        <td>Payeer’s fee</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>gateFee</b></td>
        <td>gateway fee</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td><b>comment</b></td>
        <td>comments</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>paySystem</b></td>
        <td>name payment method</td>
        <td>Payeer</td>
    </tr>
    <tr>
        <td><b>exchangeRate</b></td>
        <td>exchange rate for auto-conversion</td>
        <td>1</td>
    </tr>
    <tr>
        <td><b>protect</b></td>
        <td>transaction protection for transfer</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>protectDay</b></td>
        <td>number of days for protection</td>
        <td>1-30, null</td>
    </tr>
    <tr>
        <td><b>isApi</b></td>
        <td>via API</td>
        <td>Y, N</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>API user ID the transaction was made through</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>referenceId</b></td>
        <td>identification number in your accounting system</td>
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


## Creating an invoice for payment

Creating an invoice for payment.

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: creating an invoice</td>
        <td>invoiceCreate</td>
    </tr>
    <tr>
        <td><b>m_shop</b></td>
        <td>merchant id</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>m_orderid</b></td>
        <td>invoice id in the seller's accounting system</td>
        <td>test</td>
    </tr>
    <tr>
        <td><b>m_amount</b></td>
        <td>invoice amount</td>
        <td>10</td>
    </tr>
    <tr>
        <td><b>m_curr</b></td>
        <td>currency amount</td>
        <td>USD</td>
    </tr>
    <tr>
        <td><b>m_desc</b></td>
        <td>description</td>
        <td>Test Invoice</td>
    </tr>
</table>

### Creating an invoice for payment [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>errors</b></td>
        <td>array of errors</td>
    </tr>
    <tr>
        <td><b>success</b></td>
        <td>true, false</td>
    </tr>
    <tr>
        <td>url</td>
        <td>the link for payment</td>
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
            
                                            
## Merchant API

You can use this method for automatic invoicing in a necessary payment system.

<b>The method is enabled only for stores with high turnover by prior request</b>

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td><b>account</b></td>
        <td>Your account number in the Payeer system</td>
        <td>P1000000</td>
    </tr>
    <tr>
        <td><b>apiId</b></td>
        <td>The API user’s ID; given out when adding the API</td>
        <td>12345</td>
    </tr>
    <tr>
        <td><b>apiPass</b></td>
        <td>The API user's secret key</td>
        <td>qwerty</td>
    </tr>
    <tr>
        <td><b>action</b></td>
        <td>action type: invoice via API</td>
        <td>merchant</td>
    </tr>
    <tr>
        <td>processUrl</td>
        <td>URL for redirection to invoice payment</td>
        <td>https://payeer.com/api/merchant/process.php</td>
    </tr>
    <tr>
        <td>merchantUrl</td>
        <td>URL of the payment expectation page</td>
        <td>https://payeer.com/merchant/</td>
    </tr>
    <tr>
        <td>lang</td>
        <td>interface language</td>
        <td>ru, en</td>
    </tr>
    <tr>
        <td><b>shop</b></td>
        <td>JSON data</td>
        <td>
            <b>m_shop</b> - merchant ID<br>
            <b>m_orderid</b> - invoice ID in your accounting system<br>
            <b>m_amount</b> - amount due<br>
            <b>m_curr</b> - invoice currency<br>
            <b>m_desc</b> - comments to invoice, encoded in base64<br>
            m_params - array of data of additional parameters<br>
            <b>m_sign</b> - signature
        </td>
    </tr>
    <tr>
        <td><b>ps</b></td>
        <td>JSON data</td>
        <td>
            <b>id</b> - payment system ID, which can be viewed in the merchant settings in the "Appearance" tab<br>
            <b>curr</b> - payment system currency
        </td>
    </tr>
    <tr>
        <td><b>form</b></td>
        <td>JSON data</td>
        <td>
            <b>order_email</b> - email address of the customer who will pay the invoice
        </td>
    </tr>
    <tr>
        <td>ip</td>
        <td>customer IP</td>
        <td>8.8.8.8</td>
    </tr>
</table>

Array of data of additional parameters (m_params):
<table>
    <tr>
        <th>Name</th>
        <th>Key</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Success URL</td>
        <td><i>success_url</i></td>
        <td>This is the address the customer will be redirected to once payment has been completed successfully</td>
    </tr>
    <tr>
        <td>Fail URL</td>
        <td><i>fail_url</i></td>
        <td>This is the address the customer will be redirected to in the event of an error in the payment process or if payment is cancelled</td>
    </tr>
    <tr>
        <td>Status URL</td>
        <td><i>status_url</i></td>
        <td>The address of the payment processor. This page is where orders can be marked as paid or, for example, funds can be sent to the customer’s account on your website</td>
    </tr>
    <tr>
        <td>Additional fields</td>
        <td><i>reference</i></td>
        <td>An array of additional fields that must be returned to the payment processor</td>
    </tr>
    <tr>
        <td>Domain of sub-merchant</td>
        <td><i>submerchant</i></td>
        <td>Domain of sub-merchant (only for aggregators)</td>
    </tr>
</table>   

### Merchant API [POST]

<table>
    <tr>
        <th>Parameter</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>location</b></td>
        <td>URL for redirecting the user for payment or expectation of payment</td>
    </tr>
    <tr>
        <td><b>orderid</b></td>
        <td>invoice number in the Payeer accounting system</td>
    </tr>
    <tr>
        <td><b>historyid</b></td>
        <td>transaction number in the Payeer accounting system</td>
    </tr>
    <tr>
        <td><b>historytm</b></td>
        <td>timestamp of the creation of the transaction in Payeer</td>
    </tr>
</table>

List of potential <code>errors</code>:

<table>
    <tr>
        <th>Error</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>Shop API is disabled</b></td>
        <td>merchant API disabled for the selected merchant (by default for all merchants)</td>
    </tr>
    <tr>
        <td><b>The payment system is forbidden</b></td>
        <td>the selected payment system is currently unavailable</td>
    </tr>
    <tr>
        <td><b>PS currency is not supported</b></td>
        <td>the selected currency is not supported for the selected PSС</td>
    </tr>
    <tr>
        <td><b>Currency is not supported</b></td>
        <td>this currency is not supported by Payeer</td>
    </tr>
    <tr>
        <td><b>Amount is too large</b></td>
        <td>amount due is too high</td>
    </tr>
    <tr>
        <td><b>Summa less min</b></td>
        <td>payment amount is below the minimum</td>
    </tr>
    <tr>
        <td><b>sum_more_max</b></td>
        <td>the amount exceeds the maximum for the selected PS</td>
    </tr>
    <tr>
        <td><b>Invalid sign</b></td>
        <td>signature m_sign is invalid</td>
    </tr>
    <tr>
        <td><b>field_required</b></td>
        <td>this field is mandatory</td>
    </tr>
    <tr>
        <td><b>field_invalid_regexp</b></td>
        <td>this field does not correspond to a regular expression</td>
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

            

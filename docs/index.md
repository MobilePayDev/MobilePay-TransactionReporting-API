---
layout: default
---

# <a name="overview"/> Overview

*Purpose:* The main objective is to make it easier for you to do reconciliation on your side. We at MobilePay will provide details of transactions to help you with that, with the products that MobilePay offers. This will be done by summarizing payment, fees and refunds etc. These are summarized in further details below. 


MobilePay TransactionReporting API allows to query all activities taking place at any of your MobilePay payment locations.

This document explains how to make a technical integration to the MobilePay Transaction Reporting product. The intended audience is technical integrators from merchant itself, or from Partner Banks.

Please note: Our reconciliation API is not available in the sandbox environment.



### Merchant onboarding
As an integrator, you automatically have access to the API. As a merchant, you get access to the MobilePay Production environment, where you can test the technical integration. 

The Reconciliation API consists of GET calls, which is why it not available in the sandbox environment.

The environment is located on [The Developer Portal](https://developer.mobilepay.dk/) 

In order to call our APIs from your systems you might need to whitelist our endpoints:

| Service        | Production  |
| ------------- |:-----:|
| API Gateway  | https://api.mobilepay.dk |

# <a name="authentication"/> Authentication
### OpenID Connect
When the merchant is onboarded, he has a user in MobilePay that is able to manage which products the merchant wishes to use. The OpenID Connect protocol is a simple identity layer on top of the OAuth 2.0 protocol.

Client: In order for this to work, the merchant must grant consent to an application(Client). The client is the application that is attempting to get access to the user’s account. This consent is granted through mechanism in the [OpenID Connect](http://openid.net/connect/) protocol suite.

Client must initiate the [hybrid flow](http://openid.net/specs/openid-connect-core-1_0.html#HybridFlowAuth) specified in OpenID connect. For Transaction Reporting product the Client must request consent from the merchant using the `transactionreporting` scope. You also need to specify `offline_access` scope, in order to get the refresh token. The authorization server is [located here](https://api.mobilepay.dk/merchant-authentication-openidconnect).

If the merchant grants consent, an authorization code is returned which the Client must exchange for an id token, an access token and a refresh token. The refresh token is used to refresh ended sessions without asking for merchant consent again. This means that if the Client receives an answer from the api gateway saying that the access token is invalid, the refresh token is exchanged for a new access token and refresh token. 

An example of how to use OpenID connect in C# [can be found here](https://github.com/MobilePayDev/MobilePay-Invoice/tree/master/ClientExamples).

### Supported Endpoints
Find the supported endpoints in the links below

| Environment   | OpenID Connect Discovery |
| ------------- | ------------- |
| Production | https://api.mobilepay.dk/merchant-authentication-openidconnect/.well-known/openid-configuration |

In order to authenticate to the API, all requests to the API must contain at least three authentication headers:
1. `x-ibm-client-id`
2. `x-ibm-client-secret`
3. `Authorization`

```
$ curl 
    --header 'Authorization: Bearer <token>' 
    --header 'x-ibm-client-id: client-id' 
    --header 'x-ibm-client-secret: client-secret' 
    --url https://<mobile-pay-root>/api/merchants/me/resource
```

### Implementing OpenID Connect protocol
Although the protocol is not that complicated, there is no need to implement it yourself! There are many OpenID Connect certified libraries for different platforms, so you just have to chose the one, that suits you best [from this list](http://openid.net/developers/certified/#RPLibs).

# <a name="general_notes"/> General notes

MobilePay Transaction Reporting is a full-fledged HTTPS REST api using JSON as request/response communication media.

All dates and time-stamps use the ISO 8601 format: date format - `YYYY-MM-DD`, date-time format - `YYYY-MM-DDTHH:mm:ssZ`.

When submitting requests, `Content-Type: application/json` HTTP header must be provided. UTF-8 character encoding should be used.

`$ curl --request GET --header 'Content-Type: application/json' --url https://<mobile-pay-root>/resource`

# <a name="payment_point_id"/> PaymentPointId 

When using the Transaction Reporting API, you will be introduced to the term 'paymentPointId'. paymentPointId - GUID assigned to payment pointAs MobilePay has different API product, and each product has a payment point. The payment point is named differently across products.  

Subscriptions API  

* PaymentPointId = `SubscriptionProviderId`
 
* The individual transactions will contain the same `external_id`, that you have assigned, and that `external_id` will be returned through the API.  `external_id` is the identifier of a specific payment in merchant's system, and can be used for this purpose. You generate this ID. 


Invoice API

 * PaymentPointId = `Invoice Issuer ID` 

*  If you fill out the field `PaymentReference`, then the ID follows, that you’ve written as a  `PaymentReference`, for the payment. If you do not fill out  `PaymentReference`, then it would be the input from the field  `InvoiceNumber`,  that will be the reference on the payment.


### Result paging

Some endpoint queries can return a large number of results. In order to deliver them efficiently over the network, data pagination is used. Query responses which have more than **1000 transactions** are automatically split into pages of 1000 records each.

* A query of 859 transactions would produce a single page
* A query of 1547 transactions would produce 2 pages
* A query of 50000 transactions would produce 50 pages

When data pagination is used `NextPageToken` property is returned inside of request body. You should append the value of this property to query url `pageToken` parameter in order to retrieve the next page. If returned `NextPageToken` property is missing or null then the last page had been reached.

**Important**: NextPageToken value can contain any base64 characters, including `/`, `+` and `=`. Be sure to properly escape the value when submitting request for the next page.

#### Paging example

Initial query url is:
`https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z`

Returned response is:   
   ```javascript
  {
      "merchantId": "123456789",
      "merchantName": "Acme Ltd",
      "paymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "paymentPointName" : "snack kiosk",
      "receiverAccount" : "DK123456789",
      "transactions": [
          {
              "type": "Payment",
              "amount": 81.00,
              "currencyCode": "EUR",
              "timestamp": "2018-06-13T04:44:06Z",
              "paymentTransactionId" : "AABBCCDD11223344",
              "transferReference" : "00020180624123456789",
              "transferReferenceDate" : "2018-06-24",
              "senderComment" : "This is fun",
              "externalTransactionId": "QWERTY123456798",
              "externalBulkId": "ASDFG987654321",
          },
          ...
      ],
      "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```
In order to retrieve the next page, you shuould call:
`https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA`

### Error codes

In general the following error codes are possible. Error messages from the API are always in English, and are intended for developers, rather than to be shown to end users.

 * 400 for input validation errors. These will normally give detailed information including the specific parameters which were incorrect and examples of valid values where applicable. It is also returned if the query would yield too many results.
 * 401 is returned when required authentication headers are missing from the request.
 * 403 is returned for two cases: either user is not authorized to access specified resource or user is disabled.
 * 404 is used in the case of trying to query non existing resource. 
 * 408 is returned when request timed out. The request should be resubmmitted later.
 * 412 is used to indicate that resource exists but is not yet ready for use (for long running reports).
 * 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.
 
 
# <a name="payment_points_endpoint"/> Payment Points Endpoint

Returns Payment Points owned by a given Merchant based on the access token acquired from Merchant Authentication.

### URL

  `/transaction-reporting/api/merchant/v1/paymentpoints`
  
### Method

  `GET`
  
### Success Response

   HTTP 200
  ```javascript
{
    "merchantId": "E0225FF2-A1E6-4032-97BD-C6E90026C66B",
    "merchantName": "John's Flowershop",
    "paymentPoints": [
        {
          "paymentPointId": "55ADC672-83FB-408E-826A-1F95E732C5EC",
          "paymentPointName": "John's Flowershop Payment Point"
        }
    ]
}
  ```

Name | Type | Detail
----- |:-----:|:-----:| -----
merchantId | string | The id of the Merchant owning the Payment Points. 
merchantName | string | The name of the Merchant owning the Payment Points. 
paymentPoints | json array | Contains a list of Payment Points owned by a Merchant. 
paymentPointId | string | Id of the Payment Point. For more information about the context of the Payment Point Id, see the section regarding the Payment Point Id. 
paymentPointName | string | Name of the Payment Point. 
    
### Error Response

   * 401 when required authentication headers are missing/invalid in the request
   * 403 when the access token doesn't contain a merchant_id claim.
   * 408 is returned when request timed out. The request should be resubmmitted later.
   * 409 when the merchant could not be found.
   
### Example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints
  ```
 
# <a name="transfer_references_endpoint"/> Transfer References Endpoint

Returns a list of completed transfer references for a payment point.

**Note:** usually accumulated payment point balance is transferred once per day to a specified merchant account. You might have to wait until next day to get transfer reference and for the funds to appear in the bank account.

### URL

  `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transfer-references?from={fromDate}&to={toDate}`
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](types.md#guid) | Yes | Unique identifier for a payment point. 
fromDate | [Date](types.md#date) | Yes | Date to filter transfer reference results from (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
toDate | [Date](types.md#date) | Yes | Date to filter transfer reference results to (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
  
### Success Response

   HTTP 200
  ```javascript
  {
      "transferReferences": [
          {
              "transferReference": "00020180624123456789",
              "transferReferenceDate" : "2018-06-24",
              "totalTransferredAmount": 195.00,
              "currencyCode": "DKK"
          },
          ...
      ]
  }
  ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
transferReferences | json array | Yes | A collection of transfer reference lines (details below)
transferReference | [Transfer reference](types.md#transfer-reference) | Yes | Bank transfer reference number. Exact format can vary according to country's banking infrastructure regulations. The reference is considered unique for a duration of 1 year.
transferReferenceDate | [Date](types.md#date) | Yes | Transfer reference date. Corresponds to URL filter parameters "dateFrom" and "dateTo".
totalTransferredAmount | [Amount](types.md#amount) | Yes | Transferred amount.
currencyCode | [Currency](types.md#currency) | Yes | Transfer currency.
    
### Error Response

   * 400 when there was a validation problem with the request
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 408 is returned when request timed out. The request should be resubmmitted later.
   
### Example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfer-references?from=2018-09-18&to=2018-09-23
  ```
   
# <a name="transferred_transactions_v1_endpoint"/> Transferred Transactions V1 Endpoint

When a payment point transfer has been completed, you can retrieve a list of transactions that were included in the transfer. If you do not know the transfer reference, it can be obtained from the endpoint above.

### URL

 `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transfers/{transferReference}?pageToken={pageToken}`
  
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](types.md#guid) | Yes | Unique identifier for a payment point.
transferReference | [Transfer reference](types.md#transfer-reference) | Yes | Bank reference number used for aggregated transfer to receiver account.
pageToken | [Page token](types.md#page-token) | No | Specifies which result data page to retrieve if there are more than one page
  
### Success Response

   HTTP 200
   ```javascript
  {
      "merchantId": "123456789",
      "merchantName": "Acme Ltd",
      "paymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "paymentPointName" : "snack kiosk",
      "transferReference" : "00020180624123456789",
      "transferReferenceDate" : "2018-06-24",
      "receiverAccount" : "DK123456789",
      "transactions": [
          {
              "type": "Payment",
              "amount": 81.00,
              "currencyCode": "EUR",
              "timestamp": "2018-06-13T04:44:06Z",
              "paymentTransactionId" : "AABBCCDD11223344",
              "senderComment" : "This is fun",
              "externalTransactionId": "QWERTY123456798",
              "externalBulkId": "ASDFG987654321",
          },
          ...
      ],
      "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
merchantId | string | Yes | Public merchant identifier (usually VAT or CVR code)
merchantName | string | Yes | Merchant legal name (as registered with MobilePay)
paymentPointId | [Guid](types.md#guid) | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
paymentPointName | string | Yes | The registered name of a payment point.
transferReference | [Transfer reference](types.md#transfer-reference) | Yes | Bank reference number used for aggregated transfer to receiver account. Corresponds to url parameter.
transferReferenceDate | [Date](types.md#date) | Yes | Date used for aggregated transfer reference. Might be different from the date when transfer actually was made.
receiverAccount | string | Yes | Account number where funds have been transferred to. IBAN or regular account number.
transactions | json array | Yes | A collection of transactions (see below for details)
type | [Transaction type](types.md#transaction-type) | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, ReturnedTransaction, Adjustment
amount | [Amount](types.md#amount) | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
currencyCode | [Currency](types.md#currency) | Yes | Transaction currency.
timestamp | [Timestamp](types.md#timestamp) | Yes | Timestamp when transaction has been completed.
paymentTransactionId | string | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
senderComment | string | No | Free-form text message provided by payment sender.
externalTransactionId | string | No | ExternalTransactionId is ID that could be provided by merchant / payment integrator when initiating payments. In general, it can be used for correlating transactions between MobilePay and external (merchant/integrator) system.
externalBulkId | string | No | Similar purpose as ExternalTransactionId but used for correlating transactions with bulk/group id from external (merchant/integrator) system.
nextPageToken | [Page token](types.md#page-token) | No | A token used to retrieve next page of results. Null if this is the last page.
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 408 is returned when request timed out. The request should be resubmmitted later.
   * 412 when transfer reference is being processed and will become available later

### Example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```   

# <a name="transferred_transactions_v2_endpoint"/> Transferred Transactions V2 Endpoint

Receive a full report of the content of settlement transfers. Includes a line by line specification of your sales, fees, refunds etc. that make up settlement transfers 
### URL

 `/transaction-reporting/api/merchant/v2/paymentpoints/{paymentPointID}/transfers/{transferReference}?pageToken={pageToken}`
  
  
### Method

  `GET`

### URL Params

Name              | Type                                              | Required | Detail
-----             | -----                                             | -----    | -----
paymentPointID    | [Guid](types.md#guid)                             | Yes      | Unique identifier for a payment point.
transferReference | [Transfer reference](types.md#transfer-reference) | Yes      | Bank reference number used for aggregated transfer to receiver account.
pageToken         | [Page token](types.md#page-token)                 | No       | Specifies which result data page to retrieve if there are more than one page.

### Header Params

Name                | Type    | Required | Detail
-----               | -----   | -----    | -----
Authorization       | string  | Yes      | Authorization OpenId reference token - "Bearer xxxx".
x-ibm-client-id     | string  | Yes      | Api Gateway client Id.
x-ibm-client-secret | string  | Yes      | Api Gateway client Secrete.
Accept              | string  | No       | A value used to negotiate response 'Content-Type'. Available values `application/json` and `text/csv`. If header value contains `text/csv` endpoint returns csv file, otherwise will default to JSON format.

### JSON request example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890'
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Accept: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v2/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```

### CSV request example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890'
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Accept: text/csv'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v2/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```
  
### Success JSON Response

   HTTP 200
   ```javascript
{
    "companyRegNo": "123456789",
    "merchantName": "Acme Ltd",
    "paymentPointId": "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
    "paymentPointName": "snack kiosk",
    "transferReference": "02020180624123456789",
    "transferReferenceDate": "2020-10-14",
    "receiverAccount": "DK123400000567891231",
    "transactions": [
        {
            "type": "Payment",
            "amount": 88,00,
            "currencyCode": "DKK",
            "timestamp": "2020-10-13T04:44:06Z",
            "message": "Flowers for Anafora" || NULL,
            "merchantReference": "QWERTY123456798" || NULL,
            "merchantPaymentLabel": "Ref: AB22" || NULL,
            "paymentTransactionId": "AABBCCDD11223344",
            "userIndicator": "****1234" || NULL,
            "loyaltyId": "22499111" || NULL,
            "myshopNumber": "77643" || NULL,
            "brandName": "Acme Inc" || NULL,
            "brandId": "1234567" || NULL,
            "locationId": "56789" || NULL,
            "posName": "bakery" || NULL,
            "beaconId": "AAV2f28f-3322-4416-ab5a-BB98511c82AD" || NULL,
            "merchantPayerReference": "22449122" || NULL,
            "agreementId": "57b2f28f-9e5c-4416-ab5a-633345678ad1" || NULL,
        }
    ],
    "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
}
   ```

Name                 | Type                                                 | Required | Detail
-----                |-----                                                 |-----| -----
companyRegNo         | string                                               | Yes | Public merchant identifier (usually VAT or CVR code)
merchantName         | string                                               | Yes | Merchant legal name (as registered with MobilePay)
paymentPointId       | [Guid](types.md#guid)                                | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
paymentPointName     | string                                               | Yes | The registered name of a payment point.
transferReference    | [Transfer reference](types.md#transfer-reference)    | Yes | Bank reference number used for aggregated transfer to receiver account. Corresponds to url parameter.
transferReferenceDate| [Date](types.md#date)                                | Yes | Date used for aggregated transfer reference. Might be different from the date when transfer actually was made.
receiverAccount      | string                                               | Yes | Account number where funds have been transferred to. IBAN or regular account number.
transactions         | object[]                                             | Yes | A collection of transactions (see below for details)
type                 | [Transaction type](types.md#transaction-type)        | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, ReturnedTransaction, Adjustment
amount               | [Amount](types.md#amount)                            | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
currencyCode         | [Currency](types.md#currency)                        | Yes | Transaction currency.
timestamp            | [Timestamp](types.md#timestamp)                      | Yes | Timestamp when transaction has been completed.
message              | string                                               | No  | Free-form text message provided by payment sender.
merchantReference    | string                                               | No  | MerchantReference is ID that could be provided by merchant / payment integrator when initiating payments. In general, it can be used for correlating transactions between MobilePay and external (merchant/integrator) system.
merchantPaymentLabel | string                                               | No  | Similar purpose as MerchantReference but used for correlating transactions with bulk/group id from external (merchant/integrator) system.
paymentTransactionId | string                                               | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
userIndicator        | string                                               | No  | The payer's (user) phone number, semi masked
loyaltyId            | string                                               | No  | Value entered into the MP app for the specific merchant under "memberships"
myshopNumber         | string                                               | No  | Myshop number
brandName            | string                                               | No  | POS brand name
brandId              | string                                               | No  | POS legacy merchant Id
locationId           | string                                               | No  | POS merchant location Id
posName              | string                                               | No  | POS name
beaconId             | string                                               | No  | POS beacon Id
merchantPayerReference| string                                              | No  | Customer number/reference of payer in merchant's system, created by merchant. 
agreementId          | string                                               | No  | MobilePay Subscriptions generated agreement Id
nextPageToken        | [Page token](types.md#page-token)                    | No  | A token used to retrieve next page of results. Null if this is the last page.

### Success CSV Response

A `text/csv` *Content-Type* response with `;` seperated CSV file. A sample Transferred Transactions file can be [downloaded from here.](assets/files/transferredtransactions/f8eb42ae-a39f-4524-bb21-92f2097570d8.txt)
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 408 is returned when request timed out. The request should be resubmmitted later.
   * 412 when transfer reference is being processed and will become available later
   * 415 when Accept header contains unsupported media type


# <a name="transactions_v1_endpoint"/> Transactions V1 Endpoint

Returns a list of all transactions that took place during specified time period for a payment point.

**Note:** data provided by this endpoint represents the latest known state at the time of query. Resubmitting your request might yeld different results if additional transactions have occured during the time between requests.

### URL

  `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transactions?from={fromTimestamp}&to={toTimestamp}&pageToken={pageToken}`
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](types.md#guid) | Yes | Unique identifier for a payment point.
fromTimestamp | [Timestamp](types.md#timestamp) | Yes | Timestamp to filter transactions from (inclusive). Refers to transaction timestamp.
toTimestamp | [Timestamp](types.md#timestamp) | Yes |Timestamp to filter transactions to (inclusive). Refers to transaction timestamp.
pageToken | [Page token](types.md#page-token) | No | Specifies which result data page to retrieve if there are more than one page
  
### Success Response

   HTTP 200
   ```javascript
  {
      "merchantId": "123456789",
      "merchantName": "Acme Ltd",
      "paymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "paymentPointName" : "snack kiosk",
      "receiverAccount" : "DK123456789",
      "transactions": [
          {
              "type": "Payment",
              "amount": 81.00,
              "currencyCode": "EUR",
              "timestamp": "2018-06-13T04:44:06Z",
              "paymentTransactionId" : "AABBCCDD11223344",
              "transferReference" : "00020180624123456789",
              "transferReferenceDate" : "2018-06-24",
              "senderComment" : "This is fun",
              "externalTransactionId": "QWERTY123456798",
              "externalBulkId": "ASDFG987654321",
          },
          ...
      ],
      "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
merchantId | string | Yes | Public merchant identifier (usually VAT or CVR code)
merchantName | string | Yes | Merchant legal name (as registered with MobilePay)
paymentPointId | [Guid](types.md#guid) | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
paymentPointName | string | Yes | The registered name of a payment point.
receiverAccount | string | Yes | Account number where funds have been transferred to. IBAN or regular account number.
transactions | json array | Yes | A collection of transactions (see below for details)
type | [Transaction type](types.md#transaction-type) | Yes | Specifies transaction type. Possible values are: Payment, Refund, transactionFee, ServiceFee, Transfer, ReturnedTransaction, Adjustment
amount | [Amount](types.md#amount) | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
currencyCode | [Currency](types.md#currency) | Yes | Transaction currency.
timestamp | [Timestamp](types.md#timestamp) | Yes | Timestamp when transaction has been completed. Corresponds to url parameters "fromDateTimeOffset" and "toDateTimeOffset".
paymentTransactionId | string | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
transferReference | [Transfer reference](types.md#transfer-reference) | No | Bank reference number used for aggregated transfer to receiver account. Null if this transaction has not been transferred yet.
transferReferenceDate | [Date](types.md#date) | No | Date used for aggregated transfer reference. Null if this transaction has not been transferred yet.
senderComment | string | No | Free-form text message provided by payment sender.
externalTransactionId | string | No | ExternalTransactionId is ID that could be provided by merchant / payment integrator when initiating payments. In general, it can be used for correlating transactions between MobilePay and external (merchant/integrator) system.
externalBulkId | string | No | Similar purpose as ExternalTransactionId but used for correlating transactions with bulk/group id from external (merchant/integrator) system.
nextPageToken | [Page token](types.md#page-token) | No | A token used to retrieve next page of results. Null if this is the last page.
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 408 is returned when request timed out. The request should be resubmmitted later.

### Example
`/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transactions?from={fromTimestamp}&to={toTimestamp}`
  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```

# <a name="transactions_v2_endpoint"/> Transactions V2 Endpoint

Returns a report of all sales transactions for a payment point for the queried period.
**Note:** data provided by this endpoint represents the latest known state at the time of query. Resubmitting your request might yeld different results if additional transactions have occured during the time between requests. 


### URL

`/transaction-reporting/api/merchant/v2/paymentpoints/{paymentPointId}/transactions?from={from}&to={to}&pageToken={pageToken}`

### Method

`GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointId | [Guid](types.md#guid) | Yes | Unique identifier for a payment point.
from | [Timestamp](types.md#timestamp) | Yes | Timestamp to filter transactions from (inclusive). Refers to transaction timestamp.
to | [Timestamp](types.md#timestamp) | Yes |Timestamp to filter transactions to (inclusive). Refers to transaction timestamp.
pageToken | [Page token](types.md#page-token) | No | Specifies which result data page to retrieve if there are more than one page

### Header params

Name                | Type    | Required | Detail
-----               | -----   | -----    | -----
Authorization       | string  | Yes      | Authorization OpenId reference token - "Bearer xxxx".
x-ibm-client-id     | string  | Yes      | Api Gateway client Id.
x-ibm-client-secret | string  | Yes      | Api Gateway client Secrete.

### Success Response

HTTP 200
   ```javascript
    {
        "companyRegNo": "123456789",                                            
        "merchantName": "Acme Ltd",
        "paymentPointId": "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
        "paymentPointName": "snack kiosk",
        "receiverAccount": "DK123400000567891231",
        "transactions": [
            {
                "type": "Payment",
                "amount": 88,00
                "currencyCode": "DKK",
                "timestamp": "2020-10-13T04:44:06Z",
                "message": "Flowers for Anafora",                               
                "merchantReference": "QWERTY123456798"                          
                "merchantPaymentLabel": "Ref: AB22",                            
                "paymentTransactionId": "AABBCCDD11223344",
                "transferReference": "02020180624123456789",                    
                "transferReferenceDate": "2020-10-14",                          
                "userIndicator": "****1234" || NULL,                            
                "loyaltyId": "22499111" || NULL,                                
                "myshopNumber": "77643" || NULL,                                
                "brandName": "Acme Inc" || NULL,                                
                "brandId": "1234567" || NULL,                                   
                "locationId": "56789" || NULL,                                  
                "posName": "bakery" || NULL,                                    
                "beaconId": "AAV2f28f-3322-4416-ab5a-BB98511c82AD" || NULL,     
                "merchantPayerReference": "22449122" || NULL,                   
                "agreementId": "57b2f28f-9e5c-4416-ab5a-633345678ad1" || NULL,  
            }
        ],
        "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
    }
   ```

Name                 | Type                                                 | Required | Detail
-----                |-----                                                 |-----| -----
companyRegNo         | string                                               | Yes | Public merchant identifier (usually VAT or CVR code)
merchantName         | string                                               | Yes | Merchant legal name (as registered with MobilePay)
paymentPointId       | [Guid](types.md#guid)                                | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
paymentPointName     | string                                               | Yes | The registered name of a payment point.
receiverAccount      | string                                               | Yes | Account number where funds have been transferred to. IBAN or regular account number.
transactions         | object[]                                             | Yes | A collection of transactions (see below for details)
type                 | [Transaction type](types.md#transaction-type)        | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, ReturnedTransaction, Adjustment
amount               | [Amount](types.md#amount)                            | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
currencyCode         | [Currency](types.md#currency)                        | Yes | Transaction currency.
timestamp            | [Timestamp](types.md#timestamp)                      | Yes | Timestamp when transaction has been completed.
message              | string                                               | No  | Free-form text message provided by payment sender.
merchantReference    | string                                               | No  | MerchantReference is ID that could be provided by merchant / payment integrator when initiating payments. In general, it can be used for correlating transactions between MobilePay and external (merchant/integrator) system.
merchantPaymentLabel | string                                               | No  | Similar purpose as MerchantReference but used for correlating transactions with bulk/group id from external (merchant/integrator) system.
paymentTransactionId | string                                               | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
transferReference    | [Transfer reference](types.md#transfer-reference)    | Yes | Bank reference number used for aggregated transfer to receiver account. Corresponds to url parameter.
transferReferenceDate| [Date](types.md#date)                                | Yes | Date used for aggregated transfer reference. Might be different from the date when transfer actually was made.
userIndicator        | string                                               | No  | The payer's (user) phone number, semi masked
loyaltyId            | string                                               | No  | Value entered into the MP app for the specific merchant under "memberships"
myshopNumber         | string                                               | No  | Myshop number
brandName            | string                                               | No  | POS brand name
brandId              | string                                               | No  | POS legacy merchant Id
locationId           | string                                               | No  | POS merchant location Id
posName              | string                                               | No  | POS name
beaconId             | string                                               | No  | POS beacon Id
merchantPayerReference| string                                              | No  | Customer number/reference of payer in merchant's system, created by merchant.
agreementId          | string                                               | No  | MobilePay Subscriptions generated agreement Id
nextPageToken        | [Page token](types.md#page-token)                    | No  | A token used to retrieve next page of results. Null if this is the last page.

### Error Response

* 400 when there was a validation problem with the request.
* 401 when required authentication headers are missing/invalid in the request
* 403 when user is not authorized to access the resource or user account is disabled
* 404 when payment point does not exist
* 408 is returned when request timed out. The request should be resubmmitted later.
* 415 when Accept header contains unsupported media type
* 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.

### Example
`/transaction-reporting/api/merchant/v2/paymentpoints/{paymentPointId}/transactions?from={from}&to={to}&pageToken={pageToken}`
  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/api/merchant/v2/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```   

# <a name="transferred_transactions_by_merchant_endpoint"/> Transferred Transactions By Merchant Endpoint

Returns a list of all transactions that took place during specified time period for a merchant.

**Note:** data provided by this endpoint represents the latest known state at the time of query. Resubmitting your request might yeld different results if additional transactions have occured during the time between requests.

### URL

  `/transaction-reporting/api/merchant/v2/paymentpoints/transfers?from={fromDate}&to={toDate}&pageToken={pageToken}`
  
### Method

  `GET`

### URL Params

Name      | Type                              | Required  | Detail
-----     | -----                             | -----     | -----
from      | [Date](types.md#date)             | Yes       | Date to filter transferred transactions results (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
to        | [Date](types.md#date)             | Yes       | Date to filter transferred transactions results (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
pageToken | [Page token](types.md#page-token) | No        | Specifies which result data page to retrieve if there are more than one page

### Header params
  
Name                | Type    | Required | Detail
-----               | -----   | -----    | -----
Authorization       | string  | Yes      | Authorization OpenId reference token - "Bearer xxxx".
x-ibm-client-id     | string  | Yes      | Api Gateway client Id.
x-ibm-client-secret | string  | Yes      | Api Gateway client Secrete.
Accept              | string  | No       | A value used to negotiate response 'Content-Type'. Available values `application/json` and `text/csv`. If header value contains `text/csv` endpoint returns csv file, otherwise will default to JSON format.

### Success Response

   HTTP 200
   ```javascript
  {
    "transactions": [
      {
        "companyRegNo": "string",
        "merchantName": "string",
        "paymentPointId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "paymentPointName": "string",
        "receiverAccount": "string",
        "type": "string",
        "amount": 0,
        "currencyCode": "string",
        "timestamp": "2021-01-11T20:32:15.649Z",
        "message": "string",
        "merchantReference": "string",
        "merchantPaymentLabel": "string",
        "paymentTransactionId": "string",
        "transferReference": "string",
        "transferReferenceDate": "string",
        "userIndicator": "string",
        "loyaltyId": "string",
        "myShopNumber": "string",
        "brandName": "string",
        "brandId": "string",
        "locationId": "string",
        "posName": "string",
        "beaconId": "string",
        "merchantPayerReference": "string",
        "agreementId": "string"
      }
    ],
    "nextPageToken": "string"
  }
   ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
transactions            | json array                                           | Yes | A collection of transactions (see below for details)
companyRegNo            | string                                               | Yes | Public merchant identifier (usually VAT or CVR code)
merchantName            | string                                               | Yes | Merchant legal name (as registered with MobilePay)
paymentPointId          | [Guid](types.md#guid)                                | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
paymentPointName        | string                                               | Yes | The registered name of a payment point.
receiverAccount         | string                                               | Yes | Account number where funds have been transferred to. IBAN or regular account number.
type                    | [Transaction type](types.md#transaction-type)        | Yes | Specifies transaction type. Possible values are: Payment, Refund, transactionFee, ServiceFee, Transfer, ReturnedTransaction, Adjustment
amount                  | [Amount](types.md#amount)                            | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
currencyCode            | [Currency](types.md#currency)                        | Yes | Transaction currency.
timestamp               | [Timestamp](types.md#timestamp)                      | Yes | Timestamp when transaction has been completed. Corresponds to url parameters "from" and "to".
senderComment           | string                                               | No  | Free-form text message provided by payment sender.
paymentTransactionId    | string                                               | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
message                 | string                                               | No  | Free-form text message provided by payment sender.
merchantReference       | string                                               | No  | MerchantReference is ID that could be provided by merchant / payment integrator when initiating payments. In general, it can be used for correlating transactions between MobilePay and external (merchant/integrator) system.
merchantPaymentLabel    | string                                               | No  | Similar purpose as MerchantReference but used for correlating transactions with bulk/group id from external (merchant/integrator) system.
paymentTransactionId    | string                                               | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
transferReference       | [Transfer reference](types.md#transfer-reference)    | No  | Bank reference number used for aggregated transfer to receiver account. Null if this transaction has not been transferred yet.
transferReferenceDate   | [Date](types.md#date)                                | No  | Date used for aggregated transfer reference. Null if this transaction has not been transferred yet.
userIndicator           | string                                               | No  | The payer's (user) phone number, semi masked
loyaltyId               | string                                               | No  | Value entered into the MP app for the specific merchant under "memberships"
myshopNumber            | string                                               | No  | Myshop number
brandName               | string                                               | No  | POS brand name
brandId                 | string                                               | No  | POS legacy merchant Id
locationId              | string                                               | No  | POS merchant location Id
posName                 | string                                               | No  | POS name
beaconId                | string                                               | No  | POS beacon Id
merchantPayerReference  | string                                               | No  | Customer number/reference of payer in merchant's system, created by merchant. 
agreementId             | string                                               | No  | MobilePay Subscriptions generated agreement Id
nextPageToken           | [Page token](types.md#page-token)                    | No  | A token used to retrieve next page of results. Null if this is the last page.
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 408 is returned when request timed out. The request should be resubmmitted later.
   * 415 when Accept header contains unsupported media type
   * 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.

### JSON request example
`/transaction-reporting/api/merchant/v2/paymentpoints/transfers?from={fromDate}&to={toDate}&pageToken={pageToken}`
  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.mobilepay.dk/transaction-reporting/merchant/v2/paymentpoints/transfers?from=2020-06-13&to=2020-06-14&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```   
### CSV request example
`/transaction-reporting/api/merchant/v2/paymentpoints/transfers?from={fromDate}&to={toDate}&pageToken={pageToken}`
  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Accept: text/csv'
  --url https://api.mobilepay.dk/transaction-reporting/merchant/v2/paymentpoints/transfers?from=2020-06-13&to=2020-06-14&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```  

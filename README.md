## Overview
MobilePay Transaction Reporting allows to query all activities taking place at any of your MobilePay payment locations.

This document explains how to make a technical integration to the MobilePay Transaction Reporting product. The intended audience is technical integrators from merchant itself, or from Partner Banks.

### Merchant onboarding
You enroll to the Transaction Reporting via www.MobilePay.dk or the MobilePay Business Administration portal. Then you get access to the MobilePay Sandbox environment, where you can test the technical integration. The environment is located on [The Developer Portal](https://sandbox-developer.mobilepay.dk/) 

In order to call our APIs from your systems you might need to whitelist our endpoints:

| Service        | Sandbox           | Production  |
| ------------- |:-------------:|:-----:|
| API Gateway | https://api.sandbox.mobilepay.dk | https://api.mobilepay.dk |

## Authentication
### OpenID Connect
When the merchant is onboarded, he has a user in MobilePay that is able to manage which products the merchant wishes to use. Not all merchants have the technical capabilities to make integrations to MobilePay, instead they may need to go through applications with these capabilities. The OpenID Connect protocol is a simple identity layer on top of the OAuth 2.0 protocol.

Client: In order for this to work, the merchant must grant consent to an application(Client) with these capabilities. The client is the application that is attempting to get access to the user’s account. The client needs to get consent from the user before it can do so. This consent is granted through mechanism in the [OpenID Connect](http://openid.net/connect/) protocol suite.

Integrators are the same as Clients in the OAuth 2.0 protocol. The first thing that must be done as a Client is to go and [register here](https://www.mobilepay.dk/da-dk/Erhverv/Pages/MobilePay-integrator.aspx). Once this is done the Client must initiate the [hybrid flow](http://openid.net/specs/openid-connect-core-1_0.html#HybridFlowAuth) specified in OpenID connect. For Transaction Reporting product the Client must request consent from the merchant using the `transaction_reporting` scope. Scopes are like permissions or delegated rights that the Resource Owner wishes the client to be able to do on their behalf. You also need to specify `offline_access` scope, in order to get the refresh token. The authorization server in sandbox is [located here](https://api.sandbox.mobilepay.dk/merchant-authentication-openidconnect).

If the merchant grants consent, an authorization code is returned which the Client must exchange for an id token, an access token and a refresh token. The refresh token is used to refresh ended sessions without asking for merchant consent again. This means that if the Client receives an answer from the api gateway saying that the access token is invalid, the refresh token is exchanged for a new access token and refresh token. 

An example of how to use OpenID connect in C# [can be found here](https://github.com/MobilePayDev/MobilePay-Invoice/tree/master/ClientExamples).

### Supported Endpoints
Find the supported endpoints in the links below

| Environment   | Link |
| ------------- | ------------- |
| Sandbox | https://api.sandbox.mobilepay.dk/merchant-authentication-openidconnect/.well-known/openid-configuration |
| Production | https://api.mobilepay.dk/merchant-authentication-openidconnect/.well-known/openid-configuration |

In order to authenticate to the API, all requests to the API must contain at least three authentication headers:
1. `x-ibm-client-id`
2. `x-ibm-client-secret`
3. `Authorization`

`$ curl --header "Authorization: Bearer <token>" --header 'x-ibm-client-id: client-id' --header 'x-ibm-client-secret: client-secret' --url https://<mobile-pay-root>/api/merchants/me/resource`

### Implementing OpenID Connect protocol
Although the protocol is not that complicated, there is no need to implement it yourself! There are many OpenID Connect certified libraries for different platforms, so you just have to chose the one, that suits you best [from this list](http://openid.net/developers/certified/#RPLibs).

## General notes

MobilePay Transaction Reporting is a full-fledged HTTPS REST api using JSON as request/response communication media.

All dates and time-stamps use the ISO 8601 format: date format - `YYYY-MM-DD`, date-time format - `YYYY-MM-DDTHH:mm:ssZ`.

When submitting requests, `Content-Type: application/json` HTTP header must be provided. UTF-8 character encoding should be used.

`$ curl --request GET --header 'Content-Type: application/json' --url https://<mobile-pay-root>/resource`

### Result paging

Some endpoint queries can return a large number of results. In order to deliver them efficiently over the network, data pagination is used. Query responses which have more than **1000 transactions** are automatically split into pages of 1000 records each.

* A query of 859 transactions would produce a single page
* A query of 1547 transactions would produce 2 pages
* A query of 50000 transactions would produce 50 pages

When data pagination is used `NextPageToken` property is returned inside of request body. You should append the value of this property to query url `pageToken` parameter in order to retrieve the next page. If returned `NextPageToken` property is missing or null then the last page had been reached.

**Important**: NextPageToken value can contain any base64 characters, including `/`, `+` and `=`. Be sure to properly escape the value when submitting request for the next page.

#### Paging example

Initial query url is:
`https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z`

Returned response is:   
   ```
  {
      "MerchantId": "123456789",
      "MerchantName": "Acme Ltd",
      "PaymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "PaymentPointName" : "snack kiosk",
      "ReceiverAccount" : "DK123456789",
      "Transactions": [
          {
              "Type": "Payment",
              "Amount": 81.00,
              "CurrencyCode": "EUR",
              "CustomBulkId" : null,
              "Timestamp": "2018-06-13T04:44:06Z",
              "PaymentTransactionId" : "AABBCCDD11223344",
              "TransferReference" : "00020180624123456789",
              "TransferReferenceDate" : "2018-06-24",
              "SenderComment" : "This is fun",
              "CustomPaymentId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1"
          },
          ...
      ],
      "NextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```
In order to retrieve the next page, you shuould call:
`https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA`

### Error codes

In general the following error codes are possible. Error messages from the API are always in English, and are intended for developers, rather than to be shown to end users.

 * 400 for input validation errors. These will normally give detailed information including the specific parameters which were incorrect and examples of valid values where applicable. It is also returned if the query would yield too many results.
 * 401 is returned when required authentication headers are missing from the request.
 * 403 is returned for two cases: either user is not authorized to access specified resource or user is disabled.
 * 404 is used in the case of trying to query non existing resource. 
 * 412 is used to indicate that resource exists but is not yet ready for use (for long running reports).
 * 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.

## Transfer References Endpoint

Returns a list of completed transfer references for a payment point.

**Note:** usually accumulated payment point balance is transferred once per day to a specified merchant account. You might have to wait until next day to get transfer reference and for the funds to appear in the bank account.

### URL

  `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transfer-references?from={fromDate}&to={toDate}`
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](docs/api/types.md#guid) | Yes | Unique identifier for a payment point. 
fromDate | [Date](docs/api/types.md#date) | Yes | Date to filter transfer reference results from (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
toDate | [Date](docs/api/types.md#date) | Yes | Date to filter transfer reference results to (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
  
### Success Response

   HTTP 200
  ```javascript
  {
      "TransferReferences": [
          {
              "TransferReference": "00020180624123456789",
              "TransferReferenceDate" : "2018-06-24",
              "TotalTransferredAmount": 195.00,
              "CurrencyCode": "DKK"
          },
          ...
      ]
  }
  ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
TransferReferences | json array | Yes | A collection of transfer reference lines (details below)
TransferReference | [Transfer reference](docs/api/types.md#transfer-reference) | Yes | Bank transfer reference number. Exact format can vary according to country's banking infrastructure regulations. The reference is considered unique for a duration of 1 year.
TransferReferenceDate | [Date](docs/api/types.md#date) | Yes | Transfer reference date. Corresponds to URL filter parameters "dateFrom" and "dateTo".
TotalTransferredAmount | [Amount](docs/api/types.md#amount) | Yes | Transferred amount.
CurrencyCode | [Currency](docs/api/types.md#currency) | Yes | Transfer currency.
    
### Error Response

   * 400 when there was a validation problem with the request
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   
### Sandbox example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfer-references?from=2018-09-18&to=2018-09-23
  ```
   
## Transferred Transactions Endpoint

When a payment point transfer has been completed, you can retrieve a list of transactions that were included in the transfer. If you do not know the transfer reference, it can be obtained from the endpoint above.

### URL

 `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transfers/{transferReference}?pageToken={pageToken}`
  
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](docs/api/types.md#guid) | Yes | Unique identifier for a payment point.
transferReference | [Transfer reference](docs/api/types.md#transfer-reference) | Yes | Bank reference number used for aggregated transfer to receiver account.
pageToken | [Page token](docs/api/types.md#page-token) | No | Specifies which result data page to retrieve if there are more than one page
  
### Success Response

   HTTP 200
   ```javascript
  {
      "MerchantId": "123456789",
      "MerchantName": "Acme Ltd",
      "PaymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "PaymentPointName" : "snack kiosk",
      "TransferReference" : "00020180624123456789",
      "TransferReferenceDate" : "2018-06-24",
      "ReceiverAccount" : "DK123456789",
      "Transactions": [
          {
              "Type": "Payment",
              "Amount": 81.00,
              "CurrencyCode": "EUR",
              "CustomBulkId" : null,
              "Timestamp": "2018-06-13T04:44:06Z",
              "PaymentTransactionId" : "AABBCCDD11223344",
              "SenderComment" : "This is fun",
              "CustomPaymentId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1"
          },
          ...
      ],
      "NextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
MerchantId | string | Yes | Public merchant identifier (usually VAT or CVR code)
MerchantName | string | Yes | Merchant legal name (as registered with MobilePay)
PaymentPointId | [Guid](docs/api/types.md#guid) | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
PaymentPointName | string | Yes | The registered name of a payment point.
TransferReference | [Transfer reference](docs/api/types.md#transfer-reference) | Yes | Bank reference number used for aggregated transfer to receiver account. Corresponds to url parameter.
TransferReferenceDate | [Date](docs/api/types.md#date) | Yes | Date used for aggregated transfer reference. Might be different from the date when transfer actually was made.
ReceiverAccount | string | Yes | Account number where funds have been transferred to. IBAN or regular account number.
Transactions | json array | Yes | A collection of transactions (see below for details)
Type | [Transaction type](docs/api/types.md#transaction-type) | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, ReturnedTransaction, Payout, Adjustment, Chargeback
Amount | [Amount](docs/api/types.md#amount) | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
CurrencyCode | [Currency](docs/api/types.md#currency) | Yes | Transaction currency.
CustomBulkId | string | No | Pass through reference provided by merchant for the transaction.
Timestamp | [Timestamp](docs/api/types.md#timestamp) | Yes | Timestamp when transaction has been completed.
PaymentTransactionId | string | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.  
SenderComment | string | No | Free-form text message provided by payment sender.
CustomPaymentId | string | No | Custom payment id provided by merchant / payment integrator.
NextPageToken | [Page token](docs/api/types.md#page-token) | No | A token used to retrieve next page of results. Null if this is the last page.
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 412 when transfer reference is being processed and will become available later

### Sandbox example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```   

## Transactions Endpoint

Returns a list of all transactions that took place during specified time period for a payment point.

**Note:** data provided by this endpoint represents the latest known state at the time of query. Resubmitting your request might yeld different results if additional transactions have occured during the time between requests.

### URL

  `/transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transactions?from={fromTimestamp}&to={toTimestamp}&pageToken={pageToken}`
  
### Method

  `GET`

### URL Params

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
paymentPointID | [Guid](docs/api/types.md#guid) | Yes | Unique identifier for a payment point.
fromTimestamp | [Timestamp](docs/api/types.md#timestamp) | Yes | Timestamp to filter transactions from (inclusive). Refers to transaction timestamp.
toTimestamp | [Timestamp](docs/api/types.md#timestamp) | Yes |Timestamp to filter transactions to (inclusive). Refers to transaction timestamp.
pageToken | [Page token](docs/api/types.md#page-token) | No | Specifies which result data page to retrieve if there are more than one page
  
### Success Response

   HTTP 200
   ```javascript
  {
      "MerchantId": "123456789",
      "MerchantName": "Acme Ltd",
      "PaymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "PaymentPointName" : "snack kiosk",
      "ReceiverAccount" : "DK123456789",
      "Transactions": [
          {
              "Type": "Payment",
              "Amount": 81.00,
              "CurrencyCode": "EUR",
              "CustomBulkId" : null,
              "Timestamp": "2018-06-13T04:44:06Z",
              "PaymentTransactionId" : "AABBCCDD11223344",
              "TransferReference" : "00020180624123456789",
              "TransferReferenceDate" : "2018-06-24",
              "SenderComment" : "This is fun",
              "CustomPaymentId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1"
          },
          ...
      ],
      "NextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
  }
   ```

Name | Type | Required | Detail
----- |:-----:|:-----:| -----
MerchantId | string | Yes | Public merchant identifier (usually VAT or CVR code)
MerchantName | string | Yes | Merchant legal name (as registered with MobilePay)
PaymentPointId | [Guid](docs/api/types.md#guid) | Yes | Unique identifier for a payment point. Corresponds to the provided url parameter.
PaymentPointName | string | Yes | The registered name of a payment point.
ReceiverAccount | string | Yes | Account number where funds have been transferred to. IBAN or regular account number.
Transactions | json array | Yes | A collection of transactions (see below for details)
Type | [Transaction type](docs/api/types.md#transaction-type) | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, Transfer, ReturnedTransaction, Payout, Adjustment, Chargeback
Amount | [Amount](docs/api/types.md#amount) | Yes | Transaction amount. Positive for debit transactions, negative for credit transactions.
CurrencyCode | [Currency](docs/api/types.md#currency) | Yes | Transaction currency.
CustomBulkId | string | No | Pass through reference provided by merchant for the transaction.
Timestamp | [Timestamp](docs/api/types.md#timestamp) | Yes | Timestamp when transaction has been completed. Corresponds to url parameters "fromDateTimeOffset" and "toDateTimeOffset".
PaymentTransactionId | string | Yes | Unique payment provider transaction id used when collecting funds for this individual transaction.
TransferReference | [Transfer reference](docs/api/types.md#transfer-reference) | No | Bank reference number used for aggregated transfer to receiver account. Null if this transaction has not been transferred yet.
TransferReferenceDate | [Date](docs/api/types.md#date) | No | Date used for aggregated transfer reference. Null if this transaction has not been transferred yet.
SenderComment | string | No | Free-form text message provided by payment sender.
CustomPaymentId | string | No | Custom payment id provided by merchant / payment integrator.
NextPageToken | [Page token](docs/api/types.md#page-token) | No | A token used to retrieve next page of results. Null if this is the last page.
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist

### Sandbox example
`transaction-reporting/api/merchant/v1/paymentpoints/{paymentPointID}/transactions?from={fromTimestamp}&to={toTimestamp}`
  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890' 
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Content-Type: application/json'
  --url https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v1/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transactions?from=2018-06-13T04:44:06Z&to=2018-06-13T23:00:00Z&pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```   

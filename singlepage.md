## Overview
MobilePay Transaction Reporting allows to query all activities taking place at any of your MobilePay payment locations.

This document explains how to make a technical integration to the MobilePay Transaction Reporting product. The intended audience is technical integrators from merchant itself, or from Partner Banks.

### Merchant onboarding
You enroll to the Transaction Reporting via www.MobilePay.dk or the MobilePay Business Administration portal. Then you get access to the MobilePay Sandbox environment, where you can test the technical integration. The environment is located on [The Developer Portal](https://sandbox-developer.mobilepay.dk/) 

In order to call our APIs from your systems you might need to whitelist our endpoints:

| Service        | Sandbox           | Production  |
| ------------- |:-------------:| -----:|
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
x-ibm-client-id
x-ibm-client-secret
Authorization

#### Generate certificate
Certificates are used in our environments to provide an extra layer of security for an API. In order to be authenticated to our REST-services you have to provide a certificate, which can be self-signed if preferred. The certifcate can be generated either using makecert.exe or OpenSSL.  

For help on doing this, please see the [guide here](ClientCertificate.MD). Only the Sandbox certificate is needed for now, there's no need to create a production one at this time. Once you have created the certificate, you can move onto the next step and send us an email containing both items.

#### Email us the certificate and public key
When you have completed the step above, send an email to developer@mobilepay.dk using the template below with a zip containing both the certificate and the public key. Please zip it, as our e-mail server is quire sensitive. 

This email should also contain the **client id** from the API gateway step above. We don't want the client secret, that should be kept somewhere safe and not shared. 

If you don't know the two process payment URLs at this point, it's OK to leave these blank and we can continue the onboarding process without them. However you'll need to send a follow up with these before payments can be processed. 

    Subject: MobilePay Transaction Reporting Sandbox Registration
    
    Hi,

    Please find attached our SSL certificate and public key for the MobilePay Transaction Reporting Sandbox. Could 
    you register this with the API gateway and add us to the system?
   
    Here are our details for the registration: 
    
    Client-Id: b45f5873-cea6-40a2-82e5-81906817d680
    CVR/VATNumber: 1234567890
    PartnerID: <from existing integration if applicable>
    ContactEmail: enquiries@newpay.com
    TechnicalContactEmail: techsupport@newpay.com
    TechnicalContactPhone: +4512345678
        
    Thanks
    <Name>


## Transaction Reporting API 

### Getting a list of transactions
Once your certificate and public key have been uploaded, you are ready to query transactions as they arrive to MobilePay. Simply issue a [transactions GET request](docs/api/resources/transactions.md) 

https://api.sandbox.mobilepay.dk/payment-transactionreporting-restapi/api/v1/{paymentPointId}/transactions?from={fromTimestamp}&to={toTimestamp} with the following headers

     accept: application/json
     content-type: application/json
     x-ibm-client-id: REPLACE_THIS_KEY
     x-ibm-client-secret: REPLACE_THIS_KEY
   
You will need to supply your client certificate each time you make a call for authentication. If successful you will receive an HTTP `200 OK` response with a JSON representation of the list of transactions.

### Retrieving a list of transfer references

Usually accumulated payment point balance is transferred once per day to a specified merchant account. You might have to wait until next day to get transfer reference and the funds to appear in the bank account. Once the transfer has been made issue a [transfer reference GET request](docs/api/resources/transfer-references.md) with the same headers as above.

https://api.sandbox.mobilepay.dk/payment-transactionreporting-restapi/api/v1/{paymentPointID}/transfer-references?from={fromDate}&to={toDate}

If successful you will receive an HTTP `200 OK` response with a JSON representation of the list of transfer references. If the list is empty, it means the transfer has not been executed yet.

### Retrieving a list of transferred transactions

When a payment point transfer has been completed, you can retrieve a list of transactions that were included in the transfer. Submit a [transferred transactions GET request](docs/api/resources/transferred-transactions.md) with the same headers as above. If you do not know the transfer references, it can be obtained from the endppoint above.

https://api.sandbox.mobilepay.dk/payment-transactionreporting-restapi/api/v1/{paymentPointID}/transfer/{transferReference}

If successful you will receive an HTTP `200 OK` response with a JSON representation of the list of transactions.

# Mobile Pay Transaction Reporting API

This document contains details on aspects of the MobilePay Transaction Reporting API which are common to all endpoints/resources.

## Headers

All requests to the API should have the following HTTP headers:

    accept: application/json
    content-type: application/json
    x-ibm-client-id: REPLACE_THIS_KEY
    x-ibm-client-secret: REPLACE_THIS_KEY

## Request bodies

All request bodies are a JSON object, the parameters described in tables in the documentation for each resource are properties of the top-level object. UTF-8 character encoding should be used. 

Many of the properties are enums or other specific types, and have strict validation rules applied to them by MobilePay. For more info see [API Types](types.md).

## Error codes

In general the following error codes are possible. Error messages from the API are always in English, and are intended for developers, rather than to be shown to end users.

 * 400 for input validation errors. These will normally give detailed information including the specific parameters which were incorrect and examples of valid values where applicable. It is also returned if the query would yield too many results.
 * 401 is returned when required authentication headers are missing from the request.
 * 403 is returned for two cases: either user is not authorized to access specified resource or user is disabled.
 * 404 is used in the case of trying to query non existing resource. 
 * 412 is used to indicate that resource exists but is not yet ready for use (for long running reports).
 
  * 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.

# Transfer References Query Endpoint

Returns a list of completed transfer references for a payment point.

* **URL**

  /{paymentPointID}/transfer-references?from={fromDate}&to={toDate}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    paymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit)
    fromDate | [Date](../types.md#date) | Date to filter transfer reference results from (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
    toDate | [Date](../types.md#date) | Date to filter transfer reference results to (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
  
* **Success Response:**

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

  Name | Type | Detail
  ----- | ------ | ------
  TransferReferences | json array | A collection of transfer reference lines (details below)
  TransferReference | [Transfer reference](../types.md#transfer-reference) | Bank transfer reference number. Exact format can vary according to country's banking infrastructure regulations. The reference is considered unique for a duration of 1 year.
  TransferReferenceDate | [Date](../types.md#date) | Transfer reference date. Corresponds to URL filter parameters "dateFrom" and "dateTo".
  TotalTransferredAmount | [Amount](../types.md#amount) | Transferred amount.
  CurrencyCode | [Currency](../types.md#currency) | Transfer currency.
    
* **Error Response:**

   * 400 when there was a validation problem with the request
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   
 # Transferred Transactions Query Endpoint

Returns a list of transferred transactions that belong to a specified transfer reference.

* **URL**

  /{paymentPointID}/transfer/{transferReference}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    paymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit).
    transferReference | [Transfer reference](../types.md#transfer-reference) | Bank reference number used for aggregated transfer to receiver account.
  
* **Success Response:**

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
              "Type": "<Payment|Refund|Fee|SentBackTransfer|Payout>",
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
      "NumberOfEntries": 4,
      "TotalValueOfTransactions": 200.00,
      "TotalValueOfFees": 5.00,
      "TotalTransferredAmount": 195.00
  }
    ```

  Name | Type | Detail
  ----- | ------ | ------
  MerchantId | string | Public merchant identifier (usually VAT or CVR code)
  MerchantName | string | Merchant legal name (as registered with MobilePay)
  PaymentPointId | [Guid](../types.md#guid) | Unique identifier for a payment point. Corresponds to the provided url parameter.
  PaymentPointName | string | The registered name of a payment point.
  TransferReference | [Transfer reference](../types.md#transfer-reference) | Bank reference number used for aggregated transfer to receiver account. Corresponds to url parameter.
  TransferReferenceDate | [Date](../types.md#date) | Date used for aggregated transfer reference. Might be different from the date when transfer actually was made.
  ReceiverAccount | string | Account number where funds have been transferred to. IBAN or regular account number.
  Transactions | json array | A collection of transactions (see below for details)
  Type | [Transaction type](../types.md#transaction-type) | Specifies transaction type. Possible values are: Payment, Refund, Fee, SentBackTransfer, Payout
  Amount | [Amount](../types.md#amount) | Transaction amount. Positive for debit transactions, negative for credit transactions.
  CurrencyCode | [Currency](../types.md#currency) | Transaction currency.
  CustomBulkId | string | Pass through reference provided by merchant for the transaction (optional).
  Timestamp | [Timestamp](../types.md#timestamp) | Timestamp when transaction has been completed.
  PaymentTransactionId | string | Unique payment provider transaction id used when collecting funds for this individual transaction.  
  SenderComment | string | Free-form text message provided by payment sender (optional).
  CustomPaymentId | string | Custom payment id provided by merchant / payment integrator (optional).
  NumberOfEntries | integer | Displays how many transactions have been included in this transfer.
  TotalValueOfTransactions | [Amount](../types.md#amount) | Total value of transactions excluding fees.
  TotalValueOfFees | [Amount](../types.md#amount) | Total value of included transaction fees.
  TotalTransferredAmount | [Amount](../types.md#amount) | Amount that was sent to receiver's account (total value - total fees)
    
* **Error Response:**

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 412 when transfer reference is being processed and will become available later
   
 # Transactions Query Endpoint

Returns a list of all transactions that took place during specified time period for a payment point.

* **URL**

  /{paymentPointID}/transactions?from={fromTimestamp}&to={toTimestamp}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    paymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit)
    fromTimestamp | [Timestamp](../types.md#timestamp) | Timestamp to filter transactions from (inclusive). Refers to transaction timestamp.
    toTimestamp | [Timestamp](../types.md#timestamp) | Timestamp to filter transactions to (inclusive). Refers to transaction timestamp.
  
* **Success Response:**

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
      ]
  }
    ```

  Name | Type | Detail
  ----- | ------ | ------
  MerchantId | string | Public merchant identifier (usually VAT or CVR code)
  MerchantName | string | Merchant legal name (as registered with MobilePay)
  PaymentPointId | [Guid](../types.md#guid) | Unique identifier for a payment point. Corresponds to the provided url parameter.
  PaymentPointName | string | The registered name of a payment point.
  ReceiverAccount | string | Account number where funds have been transferred to. IBAN or regular account number.
  Transactions | json array | A collection of transactions (see below for details)
  Type | [Transaction type](../types.md#transaction-type) | Specifies transaction type. Possible values are: Payment, Refund, Fee, Transfer, SentBackTransfer, Payout
  Amount | [Amount](../types.md#amount) | Transaction amount. Positive for debit transactions, negative for credit transactions.
  CurrencyCode | [Currency](../types.md#currency) | Transaction currency.
  CustomBulkId | string | Pass through reference provided by merchant for the transaction (optional).
  Timestamp | [Timestamp](../types.md#timestamp) | Timestamp when transaction has been completed. Corresponds to url parameters "fromDateTimeOffset" and "toDateTimeOffset".
  PaymentTransactionId | string | Unique payment provider transaction id used when collecting funds for this individual transaction.
  TransferReference | [Transfer reference](../types.md#transfer-reference) | Bank reference number used for aggregated transfer to receiver account. Null if this transaction has not been transferred yet.
  TransferReferenceDate | [Date](../types.md#date) | Date used for aggregated transfer reference. Null if this transaction has not been transferred yet.
  SenderComment | string | Free-form text message provided by payment sender (optional).
  CustomPaymentId | string | Custom payment id provided by merchant / payment integrator (optional).
    
* **Error Response:**

   * 400 when there was a validation problem with the request. Also when result count exceeds 20000 records.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist

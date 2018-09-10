
# Transactions Query Endpoint

Returns a list of all transactions that took place during specified time period for a payment point..

* **URL**

  /{externalPaymentPointID}/transactions?from={fromDateTimeOffset}&to={toDateTimeOffset}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    externalPaymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit)
    fromDateTimeOffset | [Timestamp](../types.md#timestamp) | Timestamp to filter transactions from (inclusive). Refers to transaction timestamp.
    toDateTimeOffset | [Timestamp](../types.md#timestamp) | Timestamp to filter transactions to (inclusive). Refers to transaction timestamp.
  
* **Success Response:**

   HTTP 200
   ```javascript
  {
      "MerchantId": "123456789",
      "MerchantName": "Acme Ltd",
      "PaymentPointId" : "08b2f28f-9e5c-4416-ab5a-6338511c8ad1",
      "PaymentPointName" : "snack kiosk",
      "ReceiverAccount" : "DK123456789", // IBAN or regular account number
      "Transactions": [
          {
              "Type": "<Payment|Refund|Fee|SentBackTransfer|Payout>",
              "Amount": 81.00,
              "CurrencyCode": "EUR",
              "CustomBulkId" : null, // pass through reference (provided by clients), a.k.a. "Merchant free text field"
              "Timestamp": "2018-06-13T04:44:06+01:00", // timestamp in merchant's timezone
              "PaymentTransactionId" : "AABBCCDD11223344",
              "TransferReference" : "FI20180624123456789", // bank ref number, incl. FI ref number
              "TransferReferenceDate" : "2018-06-24", // reference date, not the actual transfer time
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
  ReceiverAccount | string | Account number where funds have been transferred to.
    
* **Error Response:**

   * 400 when there was a validation problem with the request
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist

## Edit Online Merchant

Make changes to an existing merchant 

* **URL**

  /online-merchants/:id
  
* **Method**

  PUT

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    id | [GUID](../types.MD#guid) | The OnlineMerchantId of the merchant to edit 
  
* **Data Params**
  
  All parameters are optional, but at least one must be specified

  Name | Type | Detail
  ----- | ------ | ------
  *Name* | string | Merchant name
  *LogoUrl* | [HttpsURL](../types.MD#httpsurl) | URL for image, should be in PNG format. The image is shown in the MobilePay App when the user is prompted to approve the payment
  *Onboard* | boolean | Whether merchant is ready to make live payments. Onboard merchants are elligible for subscription fees. Defaults to false

* **Success Response:**

   HTTP 200
   ```javascript
    {
      Name: null, 
      LogoUrl: "https://sunset-boulevard.dk/logo.png",
      Onboard: false
    }
    ```
    
* **Error Response:**

   * 401 if there was an authentication problem
   * 400 if there was a validation problem with the request
   * 404 if the merchant could not be found

* **Sample Call:**

   ```javascript
    {
      LogoUrl: "https://sunset-boulevard.dk/logo.png",
      Onboard: false
    }
   ```

* **Notes**
   
   At the time of writing, only the Name, LogoUrl and Onboard parameters can be changed. If something else needs to be changed a new Merchant must be created. The response that comes back will contain a copy of the values that were changed, and it will contain null values for other merchant properties which can be changed but were not in this request.

# Transfer References Query Endpoint

Returns a list of completed transfer references for a payment point.

* **URL**

  /{externalPaymentPointID}/transfer-references?from={fromDate}&to={toDate}
  
* **Method**

  POST

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    externalPaymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit)
    fromDate | [Date](../types.md#Date) | Date to filter transfer reference results from (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
    toDate | [Date](../types.md#Date) | Date to filter transfer reference results to (inclusive). Value refers to transfer reference date field, not the actual date / time when the transfer has been made
  
* **Data Params**

  None

* **Success Response:**

   HTTP 200
   ```javascript
{
    "TransferReferences": [
        {
            "TransferReference": "00020180624123456789",
            "TransferReferenceDate" : "2018-06-24", // reference date, not the actual transfer time
            "TotalTransferredAmount": 195.00,
            "CurrencyCode": "DKK"
        },
        ...
    ]
}
    ```

  Name | Type | Detail
  ----- | ------ | ------
  TransferReferences | json array | A collection of transfer reference lines (detailed below)
  TransferReference | [Transfer reference](../types.md#transfer reference) | Bank transfer reference number. For transfers made in Finland corresponds to 20 digit Finland bank transfer reference. Format can vary according to country's banking infrastructure regulations. The reference is considered unique for a duration of 1 year.
  MerchantName | string | Merchant legal name (as registered with MobilePay)
  PaymentPointId | [Guid](../types.md#guid) | Unique identifier for a payment point. Corresponds to the provided url parameter.
  PaymentPointName | string | The registered name of a payment point
  ReceiverAccount | string | Account number where funds were transfered to
  Transactions | json array | A collection of 
  *Onboard* | boolean | Whether merchant is ready to make live payments. Onboard merchants are elligible for subscription fees. Defaults to false

    
* **Error Response:**

   * 401 if there was an authentication problem
   * 400 if there was a validation problem with the request
   * 409 if there was a logic error such as attempting to create duplicate merchant

* **Sample Call:**

   ```javascript
    {
      PspMerchantId: "1",
      Name: "Benie's Sunset",
      CountryCode: "DK",
      SubscriptionBillingCurrency: "DKK",
      LogoUrl: "https://sunset-boulevard.dk/logo.png",
      VatNumber: "99999999"
    }
   ```

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

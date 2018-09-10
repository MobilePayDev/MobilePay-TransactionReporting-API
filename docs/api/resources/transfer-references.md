# Transfer References Query Endpoint

Returns a list of completed transfer references for a payment point.

* **URL**

  /{externalPaymentPointID}/transfer-references?from={fromDate}&to={toDate}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    externalPaymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit)
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

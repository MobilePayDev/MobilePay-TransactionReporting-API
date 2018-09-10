# Transfer References Query Endpoint

Returns a list of completed transfer references for a payment point.

* **URL**

  /{externalPaymentPointID}/transfer-references?from={fromDate}&to={toDate}
  
* **Method**

  POST

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    externalPaymentPointID | [GUID](../types.MD#guid) | Guid identifier for a payment point (not to confuse with payment point alias which is a digit)
  
* **Data Params**

  Name | Type | Detail
  ----- | ------ | ------
  PspMerchantId | string | Equivalent of ShopId in existing system, this is the Psp's identifier for the merchant. Must be unique
  Name | string | Merchant name
  CountryCode | [Country](../types.MD#countrycode) | Country where merchant is based
  VatNumber | string | CVR in Denmark, equivalent in other countries
  SubscriptionBillingCurrency | [Currency](../types.MD#currency) | Which currency the Psp subscription fee will be received in
  LogoUrl | [HttpsURL](../types.MD#httpsurl) | URL for image, should be in PNG format. The image is shown in the MobilePay App when the user is prompted to approve the payment
  *Onboard* | boolean | Whether merchant is ready to make live payments. Onboard merchants are elligible for subscription fees. Defaults to false

* **Success Response:**

   HTTP 200
   ```javascript
    {
      OnlineMerchantId: "b1a816b9-a36c-435b-a65b-0a608b3ff0bc",
      PspId: "6f9a71c5-4a8b-4309-a286-e16879fb73c9",
      PspMerchantId: "1",
      Name: "Benie's Sunset",
      CountryCode: "DK",
      SubscriptionBillingCurrency: "DKK",
      LogoUrl: "https://sunset-boulevard.dk/logo.png",
      VatNumber: "99999999",
      Onboard: false
    }
    ```
    
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

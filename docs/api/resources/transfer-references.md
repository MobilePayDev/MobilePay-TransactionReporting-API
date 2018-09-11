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

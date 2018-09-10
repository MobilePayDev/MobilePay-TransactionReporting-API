
# Transferred Transactions Query Endpoint

Returns a list of transferred transactions that belong to a specified transfer reference.

* **URL**

  /{externalPaymentPointID}/transfer/{transferReference}
  
* **Method**

  GET

*  **URL Params**

    Name | Type | Detail
    ----- | ------ | ------
    externalPaymentPointID | [Guid](../types.md#guid) | Unique identifier for a payment point (not to confuse with payment point alias which is a digit).
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

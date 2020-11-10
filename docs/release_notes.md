---
layout: default
---

# Under development

## Transferred Transactions V2 API

In order to retrieve transactions from completed transfer with MobilePay products specific data points a new endpoint can be used: 

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

### Sandbox JSON request example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890'
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Accept: application/json'
  --url https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v2/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
  ```

### Sandbox CSV request example

  ```
$ curl 
  --header "Authorization: Bearer abcd1234567890" 
  --header 'x-ibm-client-id: abcd1234567890'
  --header 'x-ibm-client-secret: abcd1234567890'
  --header 'Accept: text/csv'
  --url https://api.sandbox.mobilepay.dk/transaction-reporting/api/merchant/v2/paymentpoints/37b8450b-579b-489d-8698-c7800c65934c/transfers/00020180624123456789?pageToken=CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA
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
type                 | [Transaction type](types.md#transaction-type)        | Yes | Specifies transaction type. Possible values are: Payment, Refund, TransactionFee, ServiceFee, ReturnedTransaction, Payout, Adjustment, Chargeback
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

A `text/csv` *Content-Type* response with `;` seperated CSV file. A sample Transferred Transactions file can be [downloaded from here.](assets/files/transferredtransactions/f8eb42ae-a39f-4524-bb21-92f2097570d8.csv)
    
### Error Response

   * 400 when there was a validation problem with the request.
   * 401 when required authentication headers are missing/invalid in the request
   * 403 when user is not authorized to access the resource or user account is disabled
   * 404 when payment point does not exist
   * 412 when transfer reference is being processed and will become available later


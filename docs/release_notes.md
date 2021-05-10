---
layout: default
---

# Under development

### Personal Access Token Authentication

Personal Access Token is an implemention of OAuth 2.0 JWT Access Reference Token. More info about reference tokens can be found [here](http://docs.identityserver.io/en/latest/topics/reference_tokens.html). Reference token simplifies API authentication and removes a necessity of granting a consent for merchants to use API themselfes. 

In order to use reference token authorization on Transaction Reporting API A Super Manager User in MobilePay Portal unders Settings, Api menu item can generate reference token. Token will expire after 5 years. Generated token is shown ONCE and must be stored in a safe place. Compromised tokens can be revoked and regenerated if neccessary.

Authentication to API is done in same principle as with self-contained JWT token. All requests to the API must contain at least three headers:
1. `x-ibm-client-id`
2. `x-ibm-client-secret`
3. `Authorization`

```
$ curl 
    --header 'Authorization: Bearer <reference-token>' 
    --header 'x-ibm-client-id: client-id' 
    --header 'x-ibm-client-secret: client-secret' 
    --url https://<mobile-pay-root>/transaction-reporting/api/merchant/v1/paymentpoints
```

Currentlly only `transactionreporting` scope resources can be accessed using personal access token.

# Released 2021 01

* [Transferred Transactions By Merchant](index#transferred_transactions_by_merchant_endpoint)
* [Transactions V2](index#transactions_v2_endpoint)

# Released 2020 12

* [Transferred Transactions V2 API](index#transferred_transactions_v2_endpoint)
---
layout: default
---

# 25 May 2021

### Reference Access Token Authentication

Reference Access Token simplifies API authentication, and removes a necessity of granting a consent for merchants to use API themselves. Reference Access Token is an implementation of OAuth 2.0 JWT Access Token. More info about reference tokens can be found [here](http://docs.identityserver.io/en/latest/topics/reference_tokens.html).

Reference tokens can be generated in the MobilePay Portal by users with Super Manager role. Log on to the portal, go to Settings, click on the API tab, and follow the instructions. The generated token is shown only once, and must be stored in a safe place. It is recommended to copy/paste the created token directly into the integration/system used to authenticate. Tokens expire 5 years after creation.
Tokens can be revoked any time in the MobilePay Portal, and regenerated if necessary. When revoked, reference tokens immediately stop working.

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

Currentlly only Transaction Reporting API  can be accessed using reference access token.

# Released 2021 01

* [Transferred Transactions By Merchant](index#transferred_transactions_by_merchant_endpoint)
* [Transactions V2](index#transactions_v2_endpoint)

# Released 2020 12

* [Transferred Transactions V2 API](index#transferred_transactions_v2_endpoint)

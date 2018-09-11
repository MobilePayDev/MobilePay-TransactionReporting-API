# MobilePay Transaction Reporting

### Overview
MobilePay Transaction Reporting allows to query customer transactions made to a payment point and it's outgoing transfers to receiver's account. 

This document explains how to make a technical integration to the MobilePay Transaction Reporting product. The intended audience is technical integrators either from merchant itself, or from Payment Service Providers or from Partner Banks.

## Integration
Integrating to MobilePay Transaction Reporting is a multistep process involving creating an application interacting with our systems via our API gateway and calling the provided RESTful APIs. In the sections below, the following steps will be explained.

### Sandbox
The Sandbox is designed to give developers a shielded environment for testing and integration purposes. Sandboxes are isolated from your production organization and give you the ability to safely explore the API platform. 

#### Onboarding

- Contact us at help@mobilepay.dk to request an invite to the Sandbox environment. 
- Once you have received the email invite, go to sandbox-developer.mobilepay.dk and log in with your credentials
- Next you select your account > my Apps > Create new App to register a new application

> **IMPORTANT**: Please make a note of your Client Secret as you'll only see this once

> **TIP**: You should always store the Client Secret in a secure location, and never reveal it publicly. If you suspect that the secret key has been compromised, you may reset it immediately by clicking the link on the application details page.

To implement MobilePay Transaction Reporting, go to APIs and subscribe to "Transaction Reporting for Partners".

From the API-page you're able to call the api and get the appropiate responses.

In order to call our APIs from your systems you might need to whitelist our endpoints:

| Service        | Sandbox           | Production  |
| ------------- |:-------------:| -----:|
| API Gateway | api.sandbox.mobilepay.dk | TBC |
| Process payment callback |  TBC | TBC |

#### Generate certificate
Certificates are used in our environments to provide an extra layer of security for an API. In order to be authenticated to our REST-services you have to provide a certificate, which can be self-signed if preferred. The certifcate can be generated either using makecert.exe or OpenSSL.  

For help on doing this, please see the [guide here](ClientCertificate.MD). Only the Sandbox certificate is needed for now, there's no need to create a production one at this time. Once you have created the certificate, you can move onto the next step and send us an email containing both items.

#### Email us the certificate and public key
When you have completed the step above, send an email to help@mobilepay.dk using the template below with a zip containing both the certificate and the public key. 

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

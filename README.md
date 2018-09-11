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

Usually accumulated payment point balance is transferred once per day to a specified merchant account. You might have to wait until next day to get transfer reference and the funds to appear in the bank account. Once the transfer has been made issue a transfer reference GET request with the same headers as above.

a transfer is started. transfer happen Once you have created an Online Merchant, payments can be created for this merchant using the URL https://api.sandbox.mobilepay.dk/online-restapi/api/v1/online-payments. Send a POST request with the same headers as above, and with a body like this:

    {
      "OnlineMerchantId": "9a97de32-35e8-4f78-8f14-c94787f7c40e",
      "OrderId": "1",
      "Amount": {
        "Value": "1",
        "Currency": "DKK"
      },
      "IsTest": true,
      "BillingCurrency": "DKK",
      "ReturnUrl": "https://www.yourmerchantreturnurl.com",
      "CustomerLanguage": "da",
      "AllowedCardTypes": [
        {
          "CardType": "VisaDebit",
          "CardCountries": [
            {
              "CountryCode": "dk",
              "Fee": {
                "Value": "1",
                "Currency": "dkk"
              }
            }
          ]
        }
      ]
    }

You'll need to replace some of the parameters with your own values, in particular the OnlineMerchantId should be the Id of the merchant created in the step above. The ReturnURL is the location where the user will be redirected to after the payment is approved by the MobilePay user, any valid HTTPS URL can be used for the purposes of testing.

For more information on creating a payment see [Online Payment API Docs](docs/api/resources/online-payment.md)

### Redirecting to the MPO website

Once a payment has been created, the next step is to Request it from the MobilePay user, which requires the phone number of the user. In the Sandbox environment there are no real MobilePay users, so it's possible to test payments using some mock phone numbers which produce different behaviour. The most useful numbers for developing the integration are:

 - +4511111111 - User approves payment
 - +4522222222 - User declines payment
 - +4555555555 - User times out after 5 mins
 
 To request the payment and input the phone number, a redirect needs to be carried out to the URL https://sandprod-online.mobilepay.dk/online-website/start, the redirect url is included in the response from the create payment request as RedirectToMpoUrl. The redirect should post the PaymentToken property that was returned by the API as part of the payment creation step. As a form, the redirect should look like the below example:
 
      <form method="post" action="https://sandprod-online.mobilepay.dk/online-website/start">
                <input name="PaymentToken" type="text" value="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJjYzUyNTFjYy01MjM4LTQwNTEtYjYxYy00YWVmOTFmNmVkZWUiLCJtZXJjaGFudElkIjoiOWE5N2RlMzItMzVlOC00Zjc4LThmMTQtYzk0Nzg3ZjdjNDBlIiwibWVyY2hhbnROYW1lIjoiTWVyY2hhbnQgdG8gdGVzdCBvbmJvYXJkaW5nIiwibWVyY2hhbnRDb3VudHJ5Q29kZSI6IkRLIiwib3JkZXJJZCI6IjcxNyIsImFtb3VudCI6eyJ2YWx1ZSI6IjEiLCJjdXJyZW5jeSI6ImRrayJ9LCJhbGxvd2VkQ2FyZFR5cGVzIjpbeyJjYXJkVHlwZSI6IlZpc2FEZWJpdCIsImNhcmRDb3VudHJpZXMiOlt7ImNvdW50cnlDb2RlIjoiZGsiLCJmZWUiOnsidmFsdWUiOiIxIiwiY3VycmVuY3kiOiJka2sifX1dfV0sInJldHVyblVybCI6Imh0dHBzOi8vc3lzdC1wdWJsaWNtb2JpbGVwYXkuZGFuc2tlYmFuay5jb20vcDJvL01lcmNoYW50LzEvTWVyY2hhbnRSZXR1cm5QYWdlLmFzcHgiLCJjdXN0b21lckxhbmd1YWdlIjoiREEiLCJzdGF0dXMiOiJOZXciLCJ0aW1lb3V0SW5TZWNvbmRzIjozMDAsImV4cCI6MTUwNjMzNTY0MH0.PY_J7WkI-9b69VRLlGVeVLwlRrNuK8IcVKtt7YflhgqSroVmRFt3S4g0qkEjLT7c54BQyog-0JEscoCekb-ngY_V2da3gbLV-YEZR_Bbom5peyeuptMjhhx7dOMKtjL3N5SkT9UTYYtfBUSEdR3GItD4-AB1RpcOXOmI4ifqLrGcqWmhR_dRfPciz9Ti2QNsvlcPWg0Mur4co0w-DJnZYvr6dLhv4LJWSTQeEGCRvzjFFOxgvtVgUQkTIy_06fF06a4PyMWEgRNJy-zFrMExL92-r8paV3M-sT1DsJw1dVMMgoNwzF-N2mrVk2-EBBnRR7F8NoIfJHr9ZhY5oM0OEg">
      </form>
      
The resulting payment window shown in the browser will allow the phone number to be entered. One of the two mock numbers above can be entered for testing. 

An alternative if you want to test this without building a website to do the redirect is to use the Sample page we created on Sandbox. This allows you to copy+paste the whole Payment JSON object into a webpage, and will perform the POST and redirect for you:

https://sandprod-online.mobilepay.dk/online-website/sample

Paste the entire JSON response from the create payment API call into the text box, not just the payment token.

> **INFO**: The examples here are for the scenario where the user is buying through a website, typically either on a Tablet or Desktop/Laptop computer. There are two other flows where the user is buying using the browser on their phone or a merchant app on their phone. These flows are harder to test on Sandbox as they involve closer integration with the mobile apps, it's recommended to get the browser flow working first before looking into these
 
### Handling the process payment callback

If you've followed the above steps, creating and requesting a payment using the phone number +4511111111, should result in one of two things happening:

  1. Either the payment fails at this point and you see **Teknisk fejl** 
  2. or a 5 minute countdown is shown, which ticks down until it expires. 
  
At this point, the next step is to implement the **process payment callback**. This is an HTTP service available at the ProcessPaymentUrl (and/or TestProcessPaymentUrl) value which is provided by the PSP during onboarding. If you didn't provide URLs for this at the time of onboarding, you can do it now by email help@mobilepay.dk. 

MobilePay Online will make a POST request to the provided URL, with a request body in the following format:

    {
        OnlinePaymentId: "8ed06ba4-d467-4180-bf2d-3e212699460f",
        Type: "OnlinePaymentApproved",
        EncryptedCardData: "TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsIGNvbn...",
        ValidUntil: "2017-01-01T18:25:43.511Z",
        CardType: "Dankort",
        Date: "2017-01-01T18:25:13.511Z"
     }
     
The expected response should be an HTTP 202 Accepted, with a body like:

    {
     CardType: "Dankort"
    }

The CardType property in the response should be the card type that the PSP calculates from the provided encrypted card data. The reason for returning this is to allow MobilePay to detect any mismatches and automatically alert our teams to these for investigation. If the card type is missing or doesn't match what was provided in the request, the payment will be allowed to continue anyway. 

### Sending the authorized event
Once the process payment callback is implemented to the point where it's returning a sucessful 202 response to MobilePay, you should see that payments which are approved by the user no longer error with **Teknisk fejl**, but instead show the 5 minute countdown.

At the moment the countdown will tick down to 0 and expire for all payments. In order to stop this happening, implement the **Authorized event**. This should be sent to MobilePay Online by calling our payment events API within **30 seconds** of the process payment callback. If we receive the Authorized event within 30 seconds, the user will be shown a receipt screen indicating that the payment was successful. If we don't receive it, or it comes too late, the user will be shown an error screen.

To send the authorized event, make a POST to https://api.sandbox.mobilepay.dk/online-restapi/api/v1/online-payments/events with the same headers as the payment creation, and with a body similar to this:

    {
        OnlinePaymentId: "8ed06ba4-d467-4180-bf2d-3e212699460f",
        Date: "2017-01-01T18:37:43.221Z",
        Type: "Authorized",
        Amount: "1"
    }

For more information on posting payment events, see [Payment Events API Docs](docs/api/resources/online-payment.md#payment-events)

### Handling the authorize confirmation
When MobilePay Online receives the authorized event for a payment, it will either send an OnlinePaymentAuthorizationSucceeded event, or an OnlinePaymentAuthorizationFailed event. Which of these is received indicates whether the user was shown a receipt screen or an error screen. 

The two events will be sent via POST requests to the ProcessPaymentURL/TestProcessPaymentURL, the same as the original process payment callback.

#### OnlinePaymentAuthorizationSucceeded
This means the user was informed in the MobilePay App that the payment suceeded. After receiving this event the PSP is free to capture the funds whenever required. The capture can happen immediately on some time in the future, there is currently no time limit on when a capture can be received for an authorized payment. 

     {
        OnlinePaymentId: "8ed06ba4-d467-4180-bf2d-3e212699460f",
        Type: "OnlinePaymentAuthorizationSucceeded",
        Date: "2017-01-01T10:25:43.511Z"
     }

#### OnlinePaymentAuthorizationFailed
This means either the authorized was received too late by MobilePay Online, or some other error happened and the user was informed that the payment failed. In this case authorized funds should be voided/cancelled and not captured. The PSP should send a Cancelled event once this is done. 

     {
        OnlinePaymentId: "8ed06ba4-d467-4180-bf2d-3e212699460f",
        Type: "OnlinePaymentAuthorizationFailed",
        Date: "2017-01-01T10:25:43.511Z"
     }

> **Note**: The OnlinePaymentAuthorizationFailed event is sent at the point that the 30 seconds elapses and the payment is marked as expired. I.e. we don't wait for a delayed Authorized event to be received before sending this. If an Authorized event is received later for an expired payment it will be ignored by the system.

### Sending captured events

Once the funds are captured, an event should be sent to MobilePay to indicate this. At the time of writing this event can be sent any time after the OnlinePaymentAuthorizationSucceeded event is sent by MobilePay. There is not a time-limit. Capture events will be used by MobilePay for billing purposes and in the future to show more information to the user within the App. 

Capture events are very similar to Authorized events, they are sent by posting to the same URL: https://api.sandbox.mobilepay.dk/online-restapi/api/v1/online-payments/events

    {
        OnlinePaymentId: "8ed06ba4-d467-4180-bf2d-3e212699460f",
        Date: "2017-01-01T18:37:43.221Z",
        Type: "Captured",
        Amount: "1"
    }
   
It's possible to send multiple capture events for a payment, if for example the goods were shipping in separate batches.

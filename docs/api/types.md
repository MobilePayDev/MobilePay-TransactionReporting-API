# MobilePay Online API Types

### GUID

JSON | Description | Examples
----------- | --------- | -------
string | A standard GUID, this is used as the main ID for all MobilePay Online resources | "64e23a20-966d-4406-a1dd-1aec15704228"

### Email

JSON | Description | Examples
----------- | --------- | -------
string | Normalised email string, e.g. with domain name in lower case | techsupport@newpay.com

### CountryCode

JSON | Description | Examples
----------- | --------- | -------
string | Enum representing a supported country | "DK", "FI", "NO"

### Currency

JSON | Description | Examples
----------- | --------- | -------
string | Enum reprenting a supported currency | "DKK", "NOK", "EUR"

### HttpsUrl

JSON | Description | Examples
----------- | --------- | -------
string | A URL which must use the HTTPS scheme. MobilePay Online does not support insecure HTTP urls in any APIs | https://www.mobilepay.dk

### Money

JSON | Description | Examples
----------- | --------- | -------
object | Used to represent amounts of currency, e.g. payment amounts and card fees. The value is always in minor currency units, i.e. a value of 10 means 10 øre in Denmark. Note that the value is always expressed as a string in Json, not a number | { Value: "10", Currency: "DKK" }

### MoneyAmount
JSON | Description | Examples
----------- | --------- | -------
string | Represents an amount of money where the currency is implicit. Used for payment events, where the currency is always implicitly the same as the currency of the payment amount. In minor currency units, but expressed as a string in Json, not a number | "10" 


### ReturnUrl

JSON | Description | Examples
----------- | --------- | -------
string | This is either an HttpsUrl, or a Custom URL for AppSwitching purposes | https://www.mobilepay.dk, newpay://launch

### Language

JSON | Description | Examples
----------- | --------- | -------
string | Enum representing a supported language | "DA", "FI", "NO"

### CardType

JSON | Description | Examples
----------- | --------- | -------
string | Enum representing supported card types | These are the possible values: "Dankort", "ElectronDebit", "MaestroDebit", "MasterCardCredit", "MasterCardDebit", "VisaCredit", "VisaDebit"

If a payment supports "Dankort", this means either a Dankort card or a dual-branded Visa-Dankort being used as a Dankort. Similiarly "VisaDebit" implies either an actual Visa debit card or a dual-branded Visa-Dankort being used as visa.

### Timestamp

JSON type | Description | Examples
----------- | --------- | -------
string | All dates and times should be in UTC. The representation is an ISO 8601 24 character String | "2007-04-05T24:00:17.154Z"

### PspEventType

JSON type | Description | Examples
----------- | --------- | -------
string | Enum of different Psp events MobilePay accepts | These are the possible values "Authorized", "Declined", "Captured", "Refunded", "Cancelled", "Errored", "Expired"

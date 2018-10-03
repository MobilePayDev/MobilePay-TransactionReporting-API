# MobilePay Transaction Reporting API Types

### GUID

JSON | Description | Examples
----------- | --------- | -------
string | A standard GUID, this is used as id for MobilePay Transaction Reporting resources | "64e23a20-966d-4406-a1dd-1aec15704228"

### Amount
JSON | Description | Examples
----------- | --------- | -------
decimal number | Represents an amount of money whose currency is explicitly specified in a separate field (see below). Can be positive (credit operations) or negative (debit operations). | 10.15

### Currency

JSON | Description | Examples
----------- | --------- | -------
string | Enum reprenting a supported currency | "DKK", "NOK", "EUR"

### Date

JSON type | Description | Examples
----------- | --------- | -------
string | ISO 8601 compatible date string | "2007-04-05"

### Timestamp

JSON type | Description | Examples
----------- | --------- | -------
string | All dates and times should be in UTC. The representation is an ISO 8601 24 character String | "2007-04-05T24:00:17.154Z"


### Transfer reference

JSON type | Description | Examples
----------- | --------- | -------
string | Bank transfer reference number. For transfers made in Finland corresponds to 20 digit Finland bank transfer reference. Format can vary according to country's banking infrastructure regulations. The reference is considered unique for a duration of 1 year. | "77060913004200000128"

### Transaction type

JSON | Description | Examples
----------- | --------- | -------
string | Enum representing supported transaction types | Payment - Incoming payment from a private customer. Includes all typs of payment sources (account, card, emoney).
||| Refund - Refund of payment or partial payment.
||| TransactionFee - Fee based on a transaction.
||| ServiceFee - Fee based on other services provided by MobilePay (onboarding fee, recurring charges etc.).
||| Transfer - Transfer of money from Merchant wallet to Merchant account.
||| ReturnedTransaction - Failed (sent back) transfers, refunds and payouts.
||| Payout - Payout to a customer without prior payment.
||| Adjustment - Manual corrections affecting wallet balance.
||| Chargeback - Reversal of a prior outbound transaction because of dispute.

### Page token

JSON type | Description | Examples
----------- | --------- | -------
string | Unique page identifier when there are multiple pages of results. Can contain any base64 characters, including `/`, `+` and `=`. | "CiAKGjBpNDd2Nmp2Zml2cXRwY+jBpOXA=="

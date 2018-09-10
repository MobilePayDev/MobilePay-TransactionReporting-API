# Mobile Pay Transaction Reporting API

This document contains details on aspects of the MobilePay Transaction Reporting API which are common to all endpoints/resources.

## Headers

All requests to the API should have the following HTTP headers:

    accept: application/json
    content-type: application/json
    x-ibm-client-id: REPLACE_THIS_KEY
    x-ibm-client-secret: REPLACE_THIS_KEY

## Request bodies

All request bodies are a JSON object, the parameters described in tables in the documentation for each resource are properties of the top-level object. UTF-8 character encoding should be used. 

Many of the properties are enums or other specific types, and have strict validation rules applied to them by MobilePay. For more info see [API Types](types.md).

## Error codes

In general the following error codes are possible. Error messages from the API are always in English, and are intended for developers, rather than to be shown to end users.

 * 400 for input validation errors. These will normally give detailed information including the specific parameters which were incorrect and examples of valid values where applicable. It is also returned if the query would return too many results.
 * 401 is returned when required authentication headers are missing from the request.
 * 403 is returned for two cases: either user is not authorized to access specified resource or user is disabled.
 * 404 is used in the case of trying to query non existing resource. 
 * 412 is used to indicate that resource exists but is not yet ready for use (for long running reports).
 
  * 500 can happen if something unexpected goes wrong in the API, e.g. an unhandled exception. There is likely to be quite limited error information available in this case and it's best to contact MobilePay, providing details of what request caused the problem and when it was done. One form of 500 error that may be observed is a TimeoutException, which can ocurr when the API server did not receive an expected event after sending a command into the system to be executed. This error should be treated like other unhandled exceptions and reported, rather than ignored like a network timeout might be.

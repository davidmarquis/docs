---
title: "Using REST"
excerpt: ""
---

## Authorization

The API uses the standard HTTP *Authorization* header to pass authentication information.

### Timestamp

The client must send a custom header *x-timestamp* (time) with each request that is validated by the server. This custom header is used to determine that the request is not too old. The timestamp is also part of the signature. The timestamp must be formatted to [ISO 8061](http://en.wikipedia.org/wiki/ISO_8601) specifications.

*Example*

    X-Timestamp: 2014-06-02T15:39:31.2729234Z

### Public resources

Public resources do not require a *signature* in the **Authorization** header. However, the client must send an HTTP *Authorization* header with the *ApplicationKey*. A client can also perform a signed request to the public resource.

    Authorization = “Application” + " " + ApplicationKey

### Protected resources

Protected resources require a **signed** request. The signature is used to validate the client and to check whether the client is authorized to perform the operation. The Api is using standard HTTP *Authorization* header to pass authorization information. The following schemes are supported:

  - **Application signed request** - Used to sign requests that are specific to a particular application. Most Sinch APIs require this type of authorization scheme.
  - **User signed request** - Used to create instances. It is mainly used to login with your email account in order to perform administrative tasks such as rented numbers administration.
  - **Instance signed request** - Used to sign requests that are relevant only to your account and not specific to an application, such as administering rented numbers.

#### Application signed request

Use the following syntax to sign a request for the Sinch Platform. The result should be included in the HTTP *Authorization* header sent with the HTTP Request.

    Authorization = Scheme + " " + applicationKey + “:” + Signature

    Scheme = “Application”

    Signature = Base64 ( HMAC-SHA256 ( applicationSecret, UTF8 (
    StringToSign ) ) );

    StringToSign = HTTP-Verb + “\n” +
    Content-MD5 + “\n” +
    Content-Type + “\n” +
    CanonicalizedHeaders + “\n” +
    CanonicalizedResource;

    Content-MD5 = Base64 ( MD5 ( UTF8 ( [BODY] ) ) )

*CanonicalizedHeaders*: Currently the only header required is “X-Timestamp”.

To get the *applicationKey* and *applicationSecret*, you should create an application in the Sinch dashboard. The dashboard will display your application key and secret pair that can be used to sign requests.

*Example*

    applicationKey: 5F5C418A0F914BBC8234A9BF5EDDAD97 applicationSecret:
    JViE5vDor0Sw3WllZka15Q==

> **Note**
>
> The application secret value must be decoded from Base64 before it is used for signing.

*Example of an application signed request to a protected resource*

For the following POST request to the protected resource /v1/sms/+46700000000,

    POST /v1/sms/+46700000000
    X-Timestamp: 2014-06-04T13:41:58Z
    Content-Type: application/json

    {“message”:“Hello world”}

the signature should be formed like this:

    Content-MD5 = Base64 ( MD5 ( UTF8 ( [BODY] ) ) )
        jANzQ+rgAHyf1MWQFSwvYw==

    StringToSign
    POST
    jANzQ+rgAHyf1MWQFSwvYw==
    application/json
    x-timestamp:2014-06-04T13:41:58Z
    /v1/sms/+46700000000

    Signature = Base64 ( HMAC-SHA256 ( Secret, UTF8 ( StringToSign ) ) )
        qDXMwzfaxCRS849c/2R0hg0nphgdHciTo7OdM6MsdnM=

    HTTP Authorization Header Authorization: Application
        5F5C418A0F914BBC8234A9BF5EDDAD97:qDXMwzfaxCRS849c/2R0hg0nphgdHciTo7OdM6MsdnM=

> **Note**
>
> For requests that don’t contain a body (like GET requests) or requests where the body is empty, the Content-MD5 value of StringToSign should be left empty.

    StringToSign = HTTP-Verb + “\n” +
        “\n” +
        Content-Type + “\n” +
        CanonicalizedHeaders + “\n” +
        CanonicalizedResource;

**CanonicalizedHeaders**: Currently the only header required is “X-Timestamp”.  

##### User signed request


    Authorization = “User” + " " + USER_AUTHORIZATION

> **Note**
>
> The USER\_AUTHORIZATION is received in its entiry from ‘\[POST\] /authentication’. It can be added to the header as it is.

*Example*

    Authorization: User eyJhcHBsaWNhdGlvbktleSI6IllPVVJfQVBQTElDQVRJT05fS0VZIiwiaWRlbnRpdHkiOnsidHlwZSI6ImVtYWlsIiwiZW5kcG9pbnQiOiJhZGRyZXNzQGV4YW1wbGUuY29tIn0sImNyZWF0ZWQiOiIyMDE1LTA2LTI0VDA4OjMyOjMyLjk0MTc2MDVaIn0=:Uc3UQ6tnextCCXiuieizBGNf16SDKFGFWMpu6LKbOwA=

#### Instance signed request

In order to increase security and minimize the risk of app secrets to be compromised requests can be signed. The signature is used to validate that the client and check if the client is authorized to perform the operation. Security is increased since the secret is not actually part on the message sent over the wire.

    Authorization = “Instance” + " " + INSTANCE_ID + “:” + INSTANCE_SIGNATURE

    INSTANCE_SIGNATURE = Base64 ( Hmac-Sha256 ( INSTANCE_SECRET, Utf8 (STRING_TO_SIGN ) ) )

    STRING_TO_SIGN = HTTP-Verb + “\n” +
        CONTENT_MD5 + “\n” +
        Content-Type + “\n” +
        CanonicalizedHeaders + “\n” +
        CanonicalizedResource;

    CONTENT_MD5 = Base64 ( Md5 ( [BODY] ) )

*INSTANCE\_SECRET*: INSTANCE\_SECRET is received from ‘\[POST\] /instances’ as a Base64 encoded byte array. It **must** be decoded from Base64 before being used for signing.

*CanonicalizedHeaders* - Currently the only header required is “X-Timestamp”.

#### Instance Signing Example 1: Reserve a number

**Request**

    INSTANCE_ID: 00a3ffb1-0808-4dd4-9c7d-e4383d82e445 ~
    INSTANCE_SECRET: bRo76GRddEyetgJDTgkLHA==

    PUT v1/organisations/id/8888123/numbers/shop HTTP/1.1
    Host: api.sinch.com
    X-Timestamp: 2015-06-20T11:43:10.944Z
    Content-Type: application/json

    {“groupId”:13,“quantity”:1}

**Content-MD5**

Base64 ( HMAC-MD5 ( \[Body\] ) )

    BKCnAAx1KstTZCD0hQLbkw==

**StringToSign**

    PUT
    BKCnAAx1KstTZCD0hQLbkw==
    application/json
    x-timestamp:2015-06-20T11:43:05.270Z
    /v1/organisations/id/8888123/numbers/shop

**Signature**

Base64 ( HMAC-SHA256 ( INSTANCE\_SECRET, UTF8( \[STRING\_TO\_SIGN\] ) )
);

    a6p7RYw8bMr3JuZh1LArvWTLJjIgCeQj5nsRZaXW7VQ=

**Authorization
    Header**

    Authorization: Instance 00a3ffb1-0808-4dd4-9c7d-e4383d82e445:rMc5t4BI62b3o7JhPWX6/CslXYdbkC7rs5dyqBj9MIA=

#### Instance Signing Example 2: Get numbers assigned to an application

**Request**

    INSTANCE_ID: 00a3ffb1-0808-4dd4-9c7d-e4383d82e445
    INSTANCE_SECRET: bRo76GRddEyetgJDTgkLHA==

    PUT v1/applications/key/bb7b4e39-4227-4913-8c81-2db4abb54fb3/numbers HTTP/1.1
    Host: api.sinch.com
    X-Timestamp: 2015-06-20T11:43:10.944Z
    Content-Type: application/json

    {}

**Content-MD5**

Base64 ( HMAC-MD5 ( \[Body\] ) )

    null

> **Note**
>
> The Content-MD5 does not need to be calculated when the body is empty.

**StringToSign**

    GET

    application/json
    x-timestamp:2015-06-20T11:43:10.944Z
    /v1/applications/key/bb7b4e39-4227-4913-8c81-2db4abb54fb3/numbers

> **Note**
>
> The newline character must still be included even when the request body is empty.

**Signature**

Base64 ( HMAC-SHA256 ( INSTANCE\_SECRET, UTF8( \[STRING\_TO\_SIGN\] ) )
);

    VE1UwyOa8r9DscyBWGVZ43qEDn+SGJGoNe2aN8WrR+8=

**Authorization Header**

    Authorization: Instance 00a3ffb1-0808-4dd4-9c7d-e4383d82e445:xz+CPft5te5h9bCFJAeCd1OKhSW2ZUFnmX4gcGuZqcY=

#### Callback request signing

The Sinch Platform can initiate callback requests to a URL you define on events like call initiation, call answer, and call disconnect. All callback requests are signed using your Application key and Secret pair. The signature is included in the Authorization header of the
   request.

    Authorization = “Application” + " " + PartnerApplicationKey + “:” + Signature

    Signature = Base64 ( HMAC-SHA256 ( PartnerApplicationSecret, UTF8 (StringToSign ) ) );

    StringToSign = HTTP-Verb + “\n” +
        Content-MD5 + “\n” +
        Content-Type + “\n” +
        CanonicalizedHeaders + “\n” +
        CanonicalizedResource;

    Content-MD5 = Base64 ( MD5 ( [BODY] ) )

*Example*

    PartnerApplicationKey = 669E367E-6BBA-48AB-AF15-266871C28135
    PartnerApplicationSecret = BeIukql3pTKJ8RGL5zo0DA==

    Body
        {“event”:“ace”,“callid”:“822aa4b7-05b4-4d83-87c7-1f835ee0b6f6_257”,“timestamp”:“2014-09-24T10:59:41Z”,“version”:1}

    Content-MD5 = Base64 ( MD5 ( [BODY] ) )
        REWF+X220L4/Gw1spXOU7g==

    StringToSign
        POST
        REWF+X220L4/Gw1spXOU7g==
        application/json
        x-timestamp:2014-09-24T10:59:41Z
        /sinch/callback/ace

    Signature = Base64 ( HMAC-SHA256 ( PartnerApplicationSecret, UTF8 (StringToSign ) ) )
        Tg6fMyo8mj9pYfWQ9ssbx3Tc1BNC87IEygAfLbJqZb4=

    HTTP Authorization Header
        Authorization: Application 669E367E-6BBA-48AB-AF15-266871C28135:Tg6fMyo8mj9pYfWQ9ssbx3Tc1BNC87IEygAfLbJqZb4=



#### Callback request validation

Your development platform that receives the callbacks can verify that the request originated from Sinch by re-signing the request and comparing the result with the value contained in the *Application* HTTP header.

#### Basic authorization


To get started quickly, applications are enabled to use basic authorization instead of signing messages. To use basic authorization, set the application key as the username and the secret from the portal as the password.

    //application call
    usernameAndPassword = “application" + ApplicationKey + ”:" + ApplicationSecret

To get the *applicationKey* and *applicationSecret*, you should create an application in the Sinch dashboard. The dashboard will display your application key and secret pair that can be used to sign requests.

By convention, the username and password need to be base64 encoded before being added to the header:

    Authorization = “basic” + " " + Base64 ( usernameAndPassword )

## Methods

The API is developed using the following REST style operations:

    [GET] - Fetch / Query
    [POST] - Create / New / Start
    [PUT] - 'Full' Update / Alter / Modify
    [PATCH] - 'Partially' Update / Alter / Modify
    [DELETE] - Remove / Stop / Cancel

### Errors

The API uses the HTTP status code to indicate failure and the body contains more information.

    [Error]
        int - errorCode
        string - message

*Example*

    HTTP Status Code: 401 (Unauthorized)
    {
        "errorCode":40102,
        "message":"Invalid Signature"
    }

### Error Codes

An error code contains five digits. The first three digits are the same as the HTTP Status Code.

    [BadRequest - 400]
    40001 - Parameter Validation

    [Unauthorized - 401]
    40100 - Authorization Header
    40101 - Timestamp Header
    40102 - Invalid Signature

    [InternalServerError - 500]
    50000 - Internal Server Error

    [ServiceUnavailable - 503]
    50300 - Temporary Down

## Types

There are standard types, array types, and custom types.

### Standard Types

    string - string
    int - signed 32 bit number
    long - signed 64 bit number
    bool - boolean (true/false)
    decimal - decimal

### Optional Values

An object can contain an optional value which is represented as the type with a trailing question mark.

*Example*

    string? - firstName

### Array Values

An object can contain an array of types which is represented as the type followed by square brackets.

*Example*

    string[] - locales

### Custom Types

#### Timestamp (time)

Timestamp is sent as an string ([ISO 8061](http://en.wikipedia.org/wiki/ISO_8601)).

    2014-06-02T15:39:31Z
    2014-06-02T15:39:31.2729234Z

#### Binary data (byte\[\])

A binary byte array is sent as a Base64-encoded string.

    WbUCsMDTfW7mjCiMpG4bmw==

#### Country Id (country)

Represents a country defined as a string ([ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)).

*Example*

    "US" - United States
    "SE" - Sweden
    "GB" - United Kingdom

#### Timezone (timezone)

Represents a timezone defined as a string ([Timezone](http://en.wikipedia.org/wiki/Time_zone)).

*Example*

    "UTC-04:30" - Venezuela
    "UTC-01:00" - Cape Verde
    "UTC+08:45" - Australia (Eucla)

#### Locale (locale)

Defined as a string with Language ([ISO 639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)) and Country Id.

*Example* ([Language Codes](http://www.lingoes.net/en/translator/langcode.htm))

    "sv-SE" - Swedish (Sweden)
    "en-US" - English (United States)
    "en-GB" - English (United Kingdom)
    "fr-FR" - French (France)

##### Identity (identity)

An endpoint identity is sent as an object containing type and identity.

    [Identity]
        string - type
        string - endpoint
        bool? - verified

    Type:
        "number" - Phone Number - in international format starting with a '+'
        "email" - Email Address
        "facebook" - Facebook Id
        "userName" - User name
        "conference" - The id of the conference that a call will be connected to.
        "did" - a Phone number for inbound voice rented from Sinch (DID means "direct inward dialing")

  - **verified** is a *readonly* Boolean sent when listing a user’s identities and specifies whether the identity has been validated.

*Example*

    {
        "type":"number",
        "endpoint":"+46707123456",
        "verified":true
    }

#### Currency Id (currency)

Represents a currency defined as a string ([ISO 4217](http://en.wikipedia.org/wiki/ISO_4217)).

*Example*

    "USD" - United States Dollar
    "GBP" - United Kingdom Pound
    "EUR" - Euro Member Countries

#### Money (money)

Represents a money object that is formatted based on the currency’s defined locale.

```
[Money]
    currency - currencyId
    decimal - amount
    string? – formatted     
```

  - **formatted** is a *readonly* string that can be sent from the server for display purposes.

*Example*

    {
        "currencyId": "USD",
        "amount":10.25,
        "formatted":"$10.25"
    }


<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/voice/using-rest.md"><span class="fab fa-github"></span>Edit on GitHub</a>

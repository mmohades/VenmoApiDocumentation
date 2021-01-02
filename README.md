# Venmo Unofficial API Documentation

This is an unofficial documentation of the Venmo iOS app API. The API base route is: `https://api.venmo.com/v1/`

A Python wrapper for the Venmo API, here: [GitHub.com/mmohades/venmo](https://github.com/mmohades/venmo)

## Table of Contents

- [API Overview](#api-overview)
  - [Public Endpoints](#public-endpoints)
  - [Authentication Required Endpoints](#authentication-required-endpoints)
  - [Parameters Description](#parameters)
  - [Request Header Schema](#request-header-schema)
  - [Request Body Schema](#request-body-JSON)
- [Public Endpoints](#public-endpoints-1)
  - [Login](#login)
- [Authentication Required Endpoints](#authentication-required-endpoints-1)
  - [Account](#account)
  - [User](#user)
  - [Transaction](#transaction)

# API Overview

The API base route is `https://api.venmo.com/v1`. In this section, you will find an overview of the API endpoints, parameter description, request body schemas, and request header schemas. You can read more and see examples of each one by clicking on the endpoint.

## Public Endpoints

| Resource                                                     | Description                                          | Request Header Id           | Request Body Id         |
| ------------------------------------------------------------ | :--------------------------------------------------- | --------------------------- | ----------------------- |
| [`POST` `/oauth/access_token`](#loginget-access-token)         | Login using credentials or 2-factor.                 | [1](#request-header-schema) | [1](#request-body-json) |
| [`GET` `/account/two-factor/token?client_id=1`](#two-factor-get-options) | Get the 2-factor authentication options. (sms, etc). | [2](#request-header-schema) |                         |
| [`POST` `account/two-factor/token`](#two-factor-ask-for-text-message-code) | Ask Venmo to send you an OTP as a Text.              | [3](#request-header-schema) | [2](#request-body-json) |

## Authentication Required Endpoints

| Resource                                                     | Description                                      | Request Header Id           | Request Body Id              |
| ------------------------------------------------------------ | :----------------------------------------------- | --------------------------- | ---------------------------- |
| [`DELETE` `/oauth/access_token`](#sign-out)                    | Sign-Out, revoke the provided Access Token.      | [4](#request-header-schema) |                              |
| [`GET` `/me`](#get-your-user-profile)                                  | Get your user profile.                   | [5](#request-header-schema) |                              |
| [`GET` `/users/{user-id}`](#get-users-profile)                 | Get a user's public profile.                     | [4](#request-header-schema) |                              |
| [`GET` `/users/{user-id}/friends`](#users-friends-list)         | Get a user's friend's list.                      | [4](#request-header-schema) |                              |
| [`GET` `/stories/{transaction-id}`](#transaction-info)         | Get a specific transaction information.          | [4](#request-header-schema) |                              |
| [`GET` `/stories/target-or-actor/{user-id}`](#users-transactions-list) | Get a list of the user's transactions.           | [4](#request-header-schema) |                              |
| [`GET` `/payment-methods`](#available-payment-methods)         | Get the payment methods list, Venmo Balance, etc | [4](#request-header-schema) |                              |
| [`POST` `/payments`](#make-a-payment-or-request-money)         | Make a payment or request money.                 | [5](#request-header-schema) | [3 or 4](#request-body-json) |

## Parameters

| Name                    | Possible Value                         | Description                                                  |
| ----------------------- | -------------------------------------- | ------------------------------------------------------------ |
| amount                  | 12.2                                   | \$12.20, the amount to charge or send in dollars. For charging, the number must be negative. |
| audience                | private                                | enum { private, friends, public } The privacy of your payment. |
| Authorization           | `Bearer <token>`                       | Your Access Token                                            |
| device-id               | `88884260-05O3-8U81-58I1-2WA76F357GR9` | Phone's unique identifier that never changes.                |
| funding_source_id       | 1513921002697097045                    | Payment Id, basically the source of the money that you are sending. For example, it can be your Venmo Balance or Bank Account. |
| note                    | Pizza 🍕                                | The note of your transaction.                                |
| Payment-id              | 1513921002697097045                    | Payment method's unique identifier, like your Venmo Balance Id or your Bank Checking account Id. |
| password                | password123456                         | The user's account password in plain-text.                   |
| phone_email_or_username | email@example.com                      | The user's account email address or username.                |
| transaction-id          | 4246290347126270993                    | Transaction unique identifier.                               |
| user-id                 | 4696228937479104362                    | User unique identifier.                                      |
| Venmo-Otp               | 123456                                 | Venmo OTP received by text.                                  |
| venmo-otp-secret        | `H02SO0WYEJKMLMC4...`                  | Temporary Identifier of a user required by all the 2-Factor requests. Expires in minutes. |

## Request Header Schema

| index | Keys                                               |
| ----- | -------------------------------------------------- |
| 1     | device-id, Content-Type                            |
| 2     | device-id, venmo-otp-secret                        |
| 3     | device-id, venmo-otp-secret, Content-Type          |
| 4     | device-id, venmo-otp-secret, Venmo-Otp, User-Agent |
| 5     | Authorization                                      |
| 6     | Authorization, Content-Type                        |

## Request Body (JSON)

| Index | Keys                                                                                                                                       |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | `{ "phone_email_or_username": "", "client_id": "1","password": ""}`                                                                        |
| 2     | `{ "via": "sms" }`                                                                                                                         |
| 3     | `{ "funding_source_id": 0, "metadata": { "quasi_cash_disclaimer_viewed": false }, "user_id": 0, "audience": "", "amount": 1, "note": "" }` |
| 4     | `{ "note": "", "metadata": { "quasi_cash_disclaimer_viewed": false }, "amount": -1, "user_id": 0, "audience": "" }`                        |

# Public Endpoints

Open endpoints do not require the Authentication Token. However, they can be used to get the Authentication Token required by other methods.

## Login

Each endpoint is related to the login process, including the two factor authentication and the final goal is to get the authorization token and use that in the routes that require Authentication. The token never expires, unless you logout.

### Login/Get Access Token
Login in using your username and password in plain-text. Remember, if the `device-id` is trusted by your account, it will work. Otherwise, you will have to follow the 2-factor auth. process.

`POST` `/oauth/access_token`


#### Request

_Header_

| Key          | Example Value                          | Required | Description             |
| ------------ | -------------------------------------- | -------- | ----------------------- |
| device-id    | `88884260-05O3-8U81-58I1-2WA76F357GR9` | True     | Mobile Unique Device Id |
| Content-Type | application/json                       | True     | Identifying the body    |
| Host         | api.venmo.com                          | False    |                         |

_Body_

```json
{
  "phone_email_or_username": "email@user.com",
  "client_id": "1",
  "password": "password123456"
}
```

#### Response

`Status: 400 Bad Request`

`Status: 401 Unauthorized`

This means username and password was correct, but additional verification is required since the device-id is unknown for your account. You will need to do two additional steps for logging in with 2-factor. 1. Request for the text message to be sent to your phone. 2. Send the `code` you received on your phone to the Venmo and the `venmo-otp-secret` using this route.

_Header_

| Key              | Value                                                              |
| ---------------- | ------------------------------------------------------------------ |
| venmo-otp        | required; two_factor                                               |
| venmo-otp-secret | `H02SO0WYEJKMLMC4TFKN5YZ7WHOJO4PAWCP8HFLP8NZANO2IDRCQJ5J1GGYNYXSP` |

_Body_

```json
{
  "error": {
    "url": "https://venmo.com/two-factor",
    "message": "Additional authentication is required",
    "code": 81109,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

_Body_

```json
{
  "access_token": "28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG",
  "balance": 200.2,
  "user": {
    "username": "user",
    "phone": "18558124430",
    "first_name": "Random",
    "last_name": "Name",
    "display_name": "Random Name",
    "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
    "id": "3020735175772533188",
    "email": "random@email.com"
  }
}
```

---

### Two-Factor, Get Options
Get the two-factor authentication options that you have for your account.

`GET` `/account/two-factor/token?client_id=1`


#### Request

_Request Header_

| Key              | Example Value                                                      | Required | Description                                                                                                 |
| ---------------- | ------------------------------------------------------------------ | :------- | ----------------------------------------------------------------------------------------------------------- |
| device-id        | 88884260-05O3-8U81-58I1-2WA76F357GR9                               | True     | Mobile Unique Device Id                                                                                     |
| venmo-otp-secret | `H02SO0WYEJKMLMC4TFKN5YZ7WHOJO4PAWCP8HFLP8NZANO2IDRCQJ5J1GGYNYXSP` | True     | The otp-secret that you receive in the response header when first try to login with username and password. |

#### Response

`Status: 400 Bad Request`

_Body_

```json
{
  "error": {
    "message": "Your code has expired. Sign in again and we'll send you a new code.",
    "code": 81110,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

_Body_

```json
{
  "data": {
    "questions": [
      {
        "value": "Capital One, N.a. Checking Account",
        "question_type": "bank"
      }
    ],
    "devices": [
      {
        "value": "(XXX) XXX - 1234",
        "device_type": "sms"
      }
    ],
    "braintree": {
      "braintree_merchant_id": "BVC6R3Z8ONVTWLAD",
      "braintree_use_prod": "true",
      "braintree_cse_public_key": "WUIUVNTLRCUODNBWHC3ICAQJ1SCLCJJRIMLACUKGJLYIOOAKIRN2ISTKDFLIM90WZYZANYRRZAPT9WO3OXZYTFN8REGK3ZFW76PNPQY9LXVQ5FBGQSAEEJDRHBHIQICZCXHKQ7QGQKATXUBIM8UPPXEFH4JRNWZZJNG0TR7U09EFAOF9OMTYPLEW1HUKRXIF6GH7RGAP7M1CUFVUFNMKAJYTMPBFYJZOMTMEBJNCVGOGAQWQFBZEI612BBWR7PG6DP68LZMSURVFITLFUQL6J9BZN8VEHZBIJG3X7UFG0K5VTZDJI7MLRIYCLEI4HGMRIPGZMC3VCF28NQUHOKAAMJ9ZHE1RITTXAYZUOAYD"
    }
  }
}
```

---

### Two-Factor, Ask for Text Message Code

Ask Venmo to text you the 2-factor code.

`POST` `account/two-factor/token`


#### Request

_Header_

| Key              | Example Value                                                      | Required | Description                                                                                                |
| ---------------- | ------------------------------------------------------------------ | :------- | ---------------------------------------------------------------------------------------------------------- |
| device-id        | 88884260-05O3-8U81-58I1-2WA76F357GR9                               | True     | Mobile Unique Device Id                                                                                    |
| venmo-otp-secret | `H02SO0WYEJKMLMC4TFKN5YZ7WHOJO4PAWCP8HFLP8NZANO2IDRCQJ5J1GGYNYXSP` | True     | The otp-secret that you receive in the response header when first try to login with username and password |
| Content-Type     | application/json                                                   | True     | Send the server the body content type that is Json.                                                        |

_Body_

```json
{ "via": "sms" }
```

#### Response

`Status: 400 Bad Request`

_Body_

```json
{
  "error": {
    "message": "Your code has expired. Sign in again and we'll send you a new code.",
    "code": 81110,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

_Body_

```json
{
  "data": {
    "status": "sent"
  }
}
```

---

### Two-Factor, Get Access Token

Login using your one-time password that received by text.

`POST` `/oauth/access_token?client_id=1`


#### Request

_Header_

| Key              | Example Value                                                      | Required | Description                                                                                                |
| ---------------- | ------------------------------------------------------------------ | :------- | ---------------------------------------------------------------------------------------------------------- |
| device-id        | 88884260-05O3-8U81-58I1-2WA76F357GR9                               | True     | Mobile Unique Device Id                                                                                    |
| venmo-otp-secret | `H02SO0WYEJKMLMC4TFKN5YZ7WHOJO4PAWCP8HFLP8NZANO2IDRCQJ5J1GGYNYXSP` | True     | The otp-secret that you receive in the response header when first try to login with username and password |
| Venmo-Otp        | 123456                                                             | True     | OTP received by text.                                                                                      |
| User-Agent       | Venmo/7.38.0 (iPhone; iOS 13.0; Scale/2.0)                         | False    |                                                                                                            |

#### Response

`Status: 400 Bad Request`

_Body_

```json
{
  "error": {
    "message": "Your code has expired. Sign in again and we'll send you a new code.",
    "code": 81110,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

_Body_

```json
{
  "access_token": "28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG",
  "balance": 200.2,
  "user": {
    "username": "user",
    "phone": "18558124430",
    "first_name": "Random",
    "last_name": "Name",
    "display_name": "Random Name",
    "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
    "id": "3020735175772533188",
    "email": "random@email.com"
  }
}
```

---

# Authentication Required Endpoints

Closed endpoints require a valid Authorization Token to be included in the header of the
request. A Token can be acquired from the Login view above.

## Account

### Sign-Out

Revoke your Access Token.

`DELETE` `/oauth/access_token`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "Your OAuth Token has been revoked.",
    "code": 262,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 204 No Content`

Your Token is revoked.

---

## User

### Get Your User Profile

`GET` `/me`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

Your user profile information.

```json
{
  "feature_groups": [],
  "is_goods_services_limited": false,
  "use_new_default_funding_source_logic": true,
  "is_suspended_for_disputes": false,
  "is_indebted": false,
  "cip_status": "passed",
  "is_balance_upgrade_user": false,
  "available_instant_transfer_capabilities": [
      "banks",
      "cards"
  ],
  "zendesk_identifier": "$$hjbvhjebvehjrbvehrjbvrehjvberhjvberh.ghfdjksghfdhjsghjfdkgyufgeryufbrefbreyubreyugregbreuygbreyugbreygbreyugbreyugberygergregreyugerygregregerghkjrhgulihgljkfdhgjghfjhgk.fghfverfgerlfbrehjkgbrebgerkgbrejkgbrehjgbe",
  "notifications": {
      "outgoing_count": {
          "outgoing_requests_count": 0,
          "outgoing_payments_count": 0
      },
      "incoming_count": 0
  },
  "qrc_rewards_enabled": true,
  "user": {
      "username": "JohnDoe",
      "last_name": "Doe",
      "friends_count": 100,
      "is_group": false,
      "is_active": true,
      "trust_request": null,
      "is_venmo_team": false,
      "phone": "15551234567",
      "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
      "is_payable": true,
      "is_blocked": false,
      "id": "5611330498099736263",
      "identity": {
          "has_submitted": false
      },
      "date_joined": "2017-12-31T23:50:17",
      "about": " ",
      "display_name": "John Doe",
      "identity_type": "personal",
      "first_name": "John",
      "friend_status": null,
      "email": "john@doe.com"
  },
  "is_limited_account": false,
  "is_web_whitelisted": false,
  "needs_verification": "not_required",
  "testing_bucket_id": "123",
  "balance": "0.00",
  "automatic_transfer_enabled": false,
  "is_recovery_exempted": false
}
```

---

### Get User's Profile

`GET` `/users/{user-id}`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

The user's profile information.

```json
{
  "username": "random-name",
  "last_name": "Name",
  "friends_count": null,
  "is_group": false,
  "is_active": true,
  "trust_request": null,
  "phone": null,
  "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
  "is_blocked": false,
  "id": "5611330498099736263",
  "identity": null,
  "date_joined": "2017-12-31T23:50:17",
  "about": " ",
  "display_name": "Random Name",
  "first_name": "Random",
  "friend_status": "not_friend",
  "email": null
}
```

---

### User's Friend's List

Revoke your Access Token.

`GET` `users/{user-id}/friends?limit=1337`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

_Parameters_

| Key     | Example Value | Description                                         |
| ------- | ------------- | --------------------------------------------------- |
| limit=  | 1337          | Max number of profile in every request is 1337     |
| offset= | 1337          | How many friends to offset. Can be used for paging. |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

A list of all the friend's profile info of the provided user.

```json
{
  "pagination": {
    "previous": null,
    "next": "https://api.venmo.com/v1/users/{user-id}/friends?limit=1337&offset=1337"
  },
  "data": [
    {
      "username": "random-name",
      "last_name": "Name",
      "friends_count": null,
      "is_group": false,
      "is_active": true,
      "trust_request": null,
      "phone": null,
      "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
      "is_blocked": false,
      "id": "5611330498099736263",
      "identity": null,
      "date_joined": "2017-12-31T23:50:17",
      "about": " ",
      "display_name": "Random Name",
      "first_name": "Random",
      "friend_status": "not_friend",
      "email": null
    }
  ]
}
```

---

## Transaction

### Transaction Info

`GET` `/stories/{transaction-id}`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

All the information about a transaction that can be find.

```json
{
  "date_updated": "2018-12-27T17:30:48",
  "transfer": null,
  "app": {
    "description": "Venmo for iPhone",
    "site_url": null,
    "image_url": "https://venmo.s3.amazonaws.com/oauth/no-image-100x100.png",
    "id": 1,
    "name": "Venmo for iPhone"
  },
  "comments": {
    "count": 0,
    "data": []
  },
  "payment": {
    "status": "settled",
    "id": 8579358424347419121,
    "date_authorized": null,
    "merchant_split_purchase": null,
    "date_completed": "2018-12-27T17:02:40",
    "target": {
      "merchant": null,
      "redeemable_target": null,
      "phone": null,
      "user": {
        "username": "random-name",
        "last_name": "Name",
        "friends_count": null,
        "is_group": false,
        "is_active": true,
        "trust_request": null,
        "phone": null,
        "profile_picture_url": "no-pic",
        "is_blocked": false,
        "id": "6088917218694187751",
        "identity": null,
        "date_joined": "2015-11-03T18:20:58",
        "about": " ",
        "display_name": "Random Name",
        "first_name": "Random",
        "friend_status": "not_friend",
        "email": null
      },
      "type": "user",
      "email": null
    },
    "audience": "public",
    "actor": {
      "username": "random-name2",
      "last_name": "Name2",
      "friends_count": null,
      "is_group": false,
      "is_active": true,
      "trust_request": null,
      "phone": null,
      "profile_picture_url": "no-pic",
      "is_blocked": false,
      "id": "6088917218694187752",
      "identity": null,
      "date_joined": "2015-11-03T18:20:58",
      "about": " ",
      "display_name": "Random Name2",
      "first_name": "Random",
      "friend_status": "not_friend",
      "email": null
    },
    "note": "Dude here is your money",
    "amount": null,
    "action": "pay",
    "date_created": "2016-11-27T10:09:31",
    "date_reminded": null
  },
  "note": "Dude here is your money",
  "audience": "public",
  "likes": {
    "count": 1,
    "data": [
      {
        "username": "random-name2",
        "last_name": "Name2",
        "friends_count": null,
        "is_group": false,
        "is_active": true,
        "trust_request": null,
        "phone": null,
        "profile_picture_url": "no-pic",
        "is_blocked": false,
        "id": "6088917218694187752",
        "identity": null,
        "date_joined": "2015-11-03T18:20:58",
        "about": " ",
        "display_name": "Random Name2",
        "first_name": "Random",
        "friend_status": "not_friend",
        "email": null
      }
    ]
  },
  "mentions": {
    "count": 0,
    "data": []
  },
  "date_created": "2018-12-27T17:30:48",
  "type": "payment",
  "id": "5296429055819060535",
  "authorization": null
}
```

---

### User's Transactions List

Revoke your Access Token.

`GET` `/stories/target-or-actor/{user-id}`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

_Parameters_

| Key                        | Example Value       | Description                                              |
| -------------------------- | ------------------- | -------------------------------------------------------- |
| before_id={transaction-id} | 6088917218694187751 | List of transactions before the provided transaction-id. |
| limit=                     | 50                  | Max number of transactions per request is 50.            |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

A list of all the user's transactions. Max 50 per request.

```json
{
  "pagination": {
    "previous": "https://api.venmo.com/v1/stories/target-or-actor/{user-id}?limit=1&after_id={transaction-id}",
    "next": "https://api.venmo.com/v1/stories/target-or-actor/{user-id}?before_id=transaction-id&limit=50"
  },
  "data": [
    {
      "date_updated": "2018-12-27T17:30:48",
      "transfer": null,
      "app": {
        "description": "Venmo for iPhone",
        "site_url": null,
        "image_url": "https://venmo.s3.amazonaws.com/oauth/no-image-100x100.png",
        "id": 1,
        "name": "Venmo for iPhone"
      },
      "comments": {
        "count": 0,
        "data": []
      },
      "payment": {
        "status": "settled",
        "id": 1513921002697097045,
        "date_authorized": null,
        "merchant_split_purchase": null,
        "date_completed": "2018-12-27T17:02:40",
        "target": {
          "merchant": null,
          "redeemable_target": null,
          "phone": null,
          "user": {
            "username": "random-name",
            "last_name": "Name",
            "friends_count": null,
            "is_group": false,
            "is_active": true,
            "trust_request": null,
            "phone": null,
            "profile_picture_url": "no-pic",
            "is_blocked": false,
            "id": "6088917218694187751",
            "identity": null,
            "date_joined": "2015-11-03T18:20:58",
            "about": " ",
            "display_name": "Random Name",
            "first_name": "Random",
            "friend_status": "not_friend",
            "email": null
          },
          "type": "user",
          "email": null
        },
        "audience": "public",
        "actor": {
          "username": "random-name2",
          "last_name": "Name2",
          "friends_count": null,
          "is_group": false,
          "is_active": true,
          "trust_request": null,
          "phone": null,
          "profile_picture_url": "no-pic",
          "is_blocked": false,
          "id": "6088917218694187752",
          "identity": null,
          "date_joined": "2015-11-03T18:20:58",
          "about": " ",
          "display_name": "Random Name2",
          "first_name": "Random",
          "friend_status": "not_friend",
          "email": null
        },
        "note": "Dude here is your money",
        "amount": null,
        "action": "pay",
        "date_created": "2016-11-27T10:09:31",
        "date_reminded": null
      },
      "note": "Dude here is your money",
      "audience": "public",
      "likes": {
        "count": 1,
        "data": [
          {
            "username": "random-name2",
            "last_name": "Name2",
            "friends_count": null,
            "is_group": false,
            "is_active": true,
            "trust_request": null,
            "phone": null,
            "profile_picture_url": "no-pic",
            "is_blocked": false,
            "id": "6088917218694187752",
            "identity": null,
            "date_joined": "2015-11-03T18:20:58",
            "about": " ",
            "display_name": "Random Name2",
            "first_name": "Random",
            "friend_status": "not_friend",
            "email": null
          }
        ]
      },
      "mentions": {
        "count": 0,
        "data": []
      },
      "date_created": "2018-12-27T17:30:48",
      "type": "payment",
      "id": "5296429055819060535",
      "authorization": null
    }
  ]
}
```

---

## Payment

### Available Payment Methods

`GET` `/payment-methods`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

A list of all the available payment methods.

```json
{
  "data": [
    {
      "top_up_role": "none",
      "default_transfer_destination": "none",
      "fee": null,
      "last_four": null,
      "id": "1208112335595142453",
      "card": null,
      "assets": null,
      "peer_payment_role": "default",
      "name": "Venmo balance",
      "image_url": null,
      "bank_account": null,
      "merchant_payment_role": "none",
      "type": "balance"
    },
    {
      "top_up_role": "eligible",
      "default_transfer_destination": "default",
      "fee": null,
      "last_four": "1234",
      "id": "7239467018896641105",
      "card": null,
      "assets": {
        "detail": "bank logo.jpg",
        "thumbnail": "bank logo.jpg"
      },
      "peer_payment_role": "backup",
      "name": "Capital One, Personal Checking",
      "image_url": "/static/images/banklogos/capitalone.png",
      "bank_account": {
        "is_verified": true,
        "id": "6203910C-50R3-78D0-844A-86R44J031368",
        "bank": {
          "asset_name": "capital_one",
          "name": "Capital One"
        }
      },
      "merchant_payment_role": "backup",
      "type": "bank"
    }
  ]
}
```

---

### Make A Payment or Request Money

In the request body, if the amount is positive, then you are sending the money. If you simply take the amount value to be negative, then you are requesting for money.

`POST` `/payments`

#### Request

_Header_

| Key           | Example Value                                                | Required |
| ------------- | ------------------------------------------------------------ | :------- |
| Authorization | Bearer 28735RJZ0MG3378R8HV6946Y64D077930MZO29REK1RQ7493966107H64P7764AG | True     |
| Content-Type  | application/json                                             | True     |

_Body, Sending money_

```json
{
  "funding_source_id": 1208112335595142453,
  "metadata": {
    "quasi_cash_disclaimer_viewed": false
  },
  "user_id": 4696228937479104362,
  "audience": "private",
  "amount": 20,
  "note": "The transaction note. Like, Last night's dinner."
}
```

_Body, Requesting Money_

```json
{
  "note": "The transaction note.",
  "metadata": {
    "quasi_cash_disclaimer_viewed": false
  },
  "amount": -18,
  "user_id": 4696228937479104362,
  "audience": "private"
}
```

#### Response

`Status: 401 Bad Request`

_Body_

```json
{
  "error": {
    "message": "You did not pass a valid OAuth access token.",
    "code": 261,
    "links": null,
    "title": "Error"
  }
}
```

`Status: 200 OK`

An example of making a charge request's response:

```json
{
  "data": {
    "balance": "122.96",
    "payment": {
      "status": "pending",
      "refund": null,
      "medium": "Venmo for iPhone",
      "id": "9252187857987647599",
      "date_authorized": null,
      "fee": null,
      "date_completed": null,
      "target": {
        "merchant": null,
        "redeemable_target": null,
        "phone": null,
        "user": {
          "username": "random-name",
          "last_name": "Name",
          "friends_count": 1000,
          "is_group": false,
          "is_active": true,
          "trust_request": null,
          "phone": null,
          "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
          "is_blocked": false,
          "id": "1357434613619042959",
          "identity": null,
          "date_joined": "2018-10-19T13:45:21",
          "about": " ",
          "display_name": "Random Name",
          "first_name": "Random",
          "friend_status": null,
          "email": null
        },
        "type": "user",
        "email": null
      },
      "audience": "private",
      "actor": {
        "username": "random-name",
        "last_name": "Name",
        "friends_count": 1000,
        "is_group": false,
        "is_active": true,
        "trust_request": null,
        "phone": null,
        "profile_picture_url": "https://s3.amazonaws.com/venmo/no-image.gif",
        "is_blocked": false,
        "id": "1357434613619042959",
        "identity": null,
        "date_joined": "2018-10-19T13:45:21",
        "about": " ",
        "display_name": "Random Name",
        "first_name": "Random",
        "friend_status": null,
        "email": null
      },
      "note": "the charge note",
      "amount": 2.0,
      "action": "charge",
      "date_created": "2018-11-17T07:07:57",
      "date_reminded": null
    },
    "payment_token": null
  }
}
```

---

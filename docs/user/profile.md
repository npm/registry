# User Profile

## Table of Contents

- [Routes](#routes)
- [Objects](#objects)
    - [Page]
    - [Error]
    - [Date]
    - [Profile]
    - [Profile Update Request]
- [API Methods](#api-methods)
    - [User Detail]
    - [User Update]
        - [Two Factor Authentication Flow]

* * *

## Routes

| Method      | Route                             | Name                      |
| ----------- | --------------------------------- | ------------------------- |
| `GET`       | `/-/npm/v1/user`                  | [User Detail]             |
| `POST`      | `/-/npm/v1/user`                  | [User Update]             |

* * *

## Objects

**Attributes marked with `// *` may not always appear in the object.**

### Page

A container object for pages of lists.

```javascript
// "*" indicates a field that may not always appear
{
  total: Number,    // *
  objects: [Object],
  urls: {
    next: String,   // *
    prev: String,   // *
  }
}
```

* * *

### Error

An error. The `message` property is contains a user-facing string.

```javascript
{
  message: String
}
```

* * *

### Date

All `Date` objects will be represented as ISO-8601 UTC datetime strings.

* * *

### Profile

A user's profile.

```
// "*" indicates a field that may not always appear
{
  tfa: null |
       false |
       {"mode": "auth-only", pending: Boolean} |
       ["recovery", "codes"] |
       "otpauth://...",
  name: String,
  email: String,
  email_verified: Boolean,
  created: Date,
  updated: Date,
  cidr_whitelist: null | ["192.168.1.1/32", ...],
  fullname: String, // *
  homepage: String, // *
  freenode: String, // *
  twitter: String,  // *
  github: String    // *
}
```

When `tfa` is an array, the array will contain a list of single use recovery codes.

When `tfa` is a string beginning with `otpauth://`, it represents a URI suitable
for consumption by Google Authenticator.

When `tfa` is an object, it represents a user who has begun the `tfa` flow. The
`mode` indicates the desired TFA mode. See [**Two Factor
Authentication**][ref-tfa] for an explanation of the `auth-only` and
`auth-and-writes` modes.

`tfa` will be `null` when a profile update was requested that did not affect the
TFA status of the target user.

### Profile Update Request

```javascript
// "*" indicates a field that may not always appear
{
  password: {               // *
    old: String,
    new: String
  },
  fullname: String,         // *
  homepage: String,         // *
  freenode: String,         // *
  twitter: String,          // *
  github: String,           // *
  email: String,            // *
  tfa: {                    // *
    password: String,
    mode: "disable" |
          "auth-only" |
          "auth-and-writes"
  } |
  ["otp"]
}
```

Note: all fields are optional.

* * *

## API Methods

### User Detail

```bash
curl -u user:pass https://registry.npmjs.org/-/npm/v1/user
```

Returns:

- 200 with a [**Profile**].
- 401 when not authenticated.

* * *

### User Update

```bash
curl -X POST \
     -u user:pass \
     -H 'content-type: application/json' \
     -d <Profile Update Request> \
     https://registry.npmjs.org/-/npm/v1/user
```

[**Authentication is required**][ref-authn]

Update a user's profile data. Takes a [**Profile Update Request**] and returns
a [**Profile**] on success. The following information can be updated:

- **The user's email address.** Changing the email address will mark their
  account as unverified and send a verification email to the new account.
  Subsequent package publish attempts will be blocked until they verify
  their account.
- **The user's Twitter, GitHub, Freenode, and personal homepage.**
- **The user's password.** Changing the password will send an email to the
  user as well. It will not invalidate any existing tokens.

#### Two Factor Authentication Flow

If the profile update request enables TFA for the user for the first time, the
`tfa` attribute of the [**Profile**] will contain an `otpauth://` URL. The user
agent is expected to present this URL as a QR code. The user should then be
prompted for one code from the TFA app of their choice. Those codes should be
sent as the `tfa` attribute of a subsequent [**Profile Update Request**]. If
successful, a final [**Profile**] object will be returned with its `tfa` set to
five one time recovery passes.

An example flow follows:

```
$ curl -X POST -u user:pass -H 'content-type: application/json' \
  -d '{"tfa": {"mode": "auth-and-writes", "password": "pass"}' \
  https://registry.npmjs.org/-/npm/v1/user | json tfa | npx qrcode -
# you should see a qrcode here. scan scan scan
#
# until you take the next step, tfa is not enabled for your account
#
# now record a one time pass from your authenticator like so:
$ curl -X POST -u user:pass -H 'content-type: application/json' \
  -d '{"tfa": ["CODE"]}' \
  https://registry.npmjs.org/-/npm/v1/user | json tfa
[
  "recovery",
  "codes",
  "are",
  "here",
  "yay"
]
```

Returns:

- 200 with a [**Profile**].
- 401 when not authenticated.

* * *

[Date]: #date
[Error]: #error
[Page]: #page
[Profile Update Request]: #profile-update-request
[Profile]: #profile
[**Error**]: #error
[**Page**]: #page
[**Profile Update Request**]: #profile-update-request
[**Profile**]: #profile
[User Detail]: #user-detail
[User Update]: #user-update
[Two Factor Authentication Flow]: #two-factor-authentication-flow
[ref-authn]: ./authentication.md#how-to-authenticate
[ref-tfa]: ./authentication.md#two-factor-authentication

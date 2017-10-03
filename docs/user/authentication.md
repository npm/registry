# User Authentication

## Table of Contents

- [How To Authenticate](#how-to-authenticate)
    - [Two Factor Authentication](#two-factor-authentication)
- [Routes](#routes)
- [Objects](#objects)
    - [Page]
    - [Error]
    - [Legacy Status]
    - [Date]
    - [Token]
    - [Token Creation Request]
    - [Login Request]
    - [Login Response]
- [API Methods](#api-methods)
    - [Login]
    - [User Update]
    - [Token List]
    - [Token Create]
    - [Token Delete]

* * *

## How To Authenticate

Authentication can be provided in `Basic` or `Bearer` form. If two factor auth
is enabled for the target user, a one time pass may be required. One time passes
may be provided using the `npm-otp` header.

All examples will use `-u username:password` for brevity. Bearer token auth is
generally preferred and will work in all cases. To create a Bearer token, use
the [**Login**] or [**Token Create**] endpoints.

Examples:

```
$ curl -u username:password https://registry.npmjs.org/          # basic

$ curl -H 'Authorization: Bearer 29bfaaaa-46c5-aaee-ba6f-92bedeadbeef' \
  https://registry.npmjs.org/                                    # bearer

$ curl -H 'npm-otp: 10101' \                                # basic + otp
  -u username:password https://registry.npmjs.org/

$ curl -H 'npm-otp: 10101' \                                # bearer + otp
  -H 'Authorization: Bearer 29bfaaaa-46c5-aaee-ba6f-92bedeadbeef' \
  https://registry.npmjs.org/
```

### Two Factor Authentication

Users may enable two factor authentication using the [User
Update][ref-userupdate] endpoint. npm's two factor auth implementation is
time-based, generating six-digit codes every 30 seconds. To authenticate a
request that is subject to two-factor auth, include a `npm-otp` with one of the
six-digit codes **or** an unused recovery code.

There are two possible modes for two factor auth: `auth-only` and
`auth-and-writes`.

In `auth-only` mode, only requests that require a password are subject to two
factor authentication. For example, changing one's password using the [User
Update][ref-userupdate] endpoint, creating a new token using the [Login] or
[Token Create] endpoints, or any `Basic` auth request.

In `auth-and-writes` mode, all `PUT`, `DELETE`, and `POST` requests are subject
to two factor authentication. The only exceptions to this rule are: starring or
unstarring a packaging, or adding or removing non-"latest" dist-tags on
packages.

* * *

## Routes

| Method      | Route                              | Name                      |
| ----------- | ---------------------------------- | ------------------------- |
| `PUT`       | `/-/user/org.couchdb.user:${user}` | [Login]                   |
| `POST`      | `/-/npm/v1/user`                   | [User Update]             |
| `GET`       | `/-/npm/v1/tokens`                 | [Token List]              |
| `POST`      | `/-/npm/v1/tokens`                 | [Token Create]            |
| `DELETE`    | `/-/npm/v1/tokens/token/:hash`     | [Token Delete]            |
| `DELETE`    | `/-/user/token/${token}`           | [Delete Token]            |

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

An error. The `message` property is contains a user-facing string. Clients should
check for `message`, then `error`.

```javascript
{
  message: String,  // * (check this first)
  error: String     // * (check this second)
  ok: false         // *
}
```

* * *

### Legacy Status

A legacy status. Reports whether or not an operation succeeded. The status is
largely superficial: users may rely on the HTTP status issued with the response
instead, which is authoritative.

```javascript
{
  ok: Boolean
}
```

* * *

### Date

All `Date` objects will be represented as ISO-8601 UTC datetime strings.

* * *

### Token

A token for a user. On creation, the `token` attribute will be a valid UUID. If
`readonly == true`, the token may only be used to authenticate for
non-destructive HTTP methods -- for example, `GET` and `HEAD` requests.

```javascript
{
  token: String,  // "[REDACTED]" on subsequent display. The "Bearer" token.
  key: String,    // sha512 hash of the token UUID
  cidr_whitelist: [String],
  created: Date,
  updated: Date,
  readonly: Boolean
}
```

* * *

### Token Creation Request

```javascript
// "*" indicates a field that may not always appear
{
  password: String,
  readonly: Boolean,  // *
  cidr_whitelist: [   // *
    String,
    ...
  ]
}
```

* * *

### Login Request

```javascript
// "*" indicates a field that may not always appear
{
  name: String,
  password: String,
  readonly: Boolean,  // *
  cidr_whitelist: [   // *
    String,           // e.g., "192.168.1.1/32"
    ...
  ]
}
```

* * *

### Login Response

A legacy response used by `npm login`. `id` and `rev` are vestigal fields from
couchdb and should be ignored by clients; they may go away in the future.

```javascript
{
  token: String,                      // the Bearer token
  ok: true,
  id: 'org.couchdb.user:undefined',
  rev: '_we_dont_use_revs_any_more'
}
```

* * *

## API Methods

### Login

```bash
curl -X PUT \
     -H 'content-type: application/json' \
     -d <Login Request> \
     http://registry.npmjs.org/-/user/org.couchdb.user:${user}
```

Authentication is not required.

Attempt to login as the username listed in [**Login Request**]. If the user in
question has [**Two Factor Authentication**] enabled, a valid `npm-otp` header
must be provided.

Returns:

- `201` with a [**Login Response**] on successful authentication.
- `401` with a [**Legacy Status**] if authentication was unsuccessful.

* * *

### User Update

See [**User Update**][ref-userupdate] in the profile documentation.

**NOTE**: This endpoint allows users to enable, disable, or change their TFA
settings.

* * *

### Token List

```bash
curl -u user:pass \
     https://registry.npmjs.org/-/npm/v1/tokens
```

[**Authentication is required**][ref-authn]

Fetch a user's active tokens. The token UUIDs will be redacted, but the objects
will have a `key` attribute populated, which is the hexadecimal sha512 hash of
the token.

Query parameters:

- `perPage`: Integer, 1-9999. Defaults to 10.
- `page`: Integer. Defaults to 0.

Returns:

- `200` with a [**Page**] of [**Token**] objects.
- `400` with an [**Error**] for an invalid page or out of range page.

* * *

### Token Create

```bash
curl -u user:pass \
     -X POST \
     -H 'content-type: application/json' \
     -d <Token Creation Request>
     https://registry.npmjs.org/-/npm/v1/tokens
```

[**Authentication is required**][ref-authn]

Create a new [**Token**] viable for authenticating as the currently
authenticated user. Takes a [**Token Creation Request**]. The following
attributes may be requested:

- `readonly`: The token will be valid only for authenticating non-destructive
  HTTP methods -- e.g., `GET` and `HEAD` requests.
- `cidr_whitelist`: The token will only be valid for requesting IPs specified
  by this list (or any IPs set on the user themselves as part of a profile
  update.)

Example:

```
$ curl -u test-chrisdickinson:some-password -X POST -d '{"password": "some-password"}' \
  -H 'content-type: application/json' https://registry.npmjs.org/-/npm/v1/tokens | json
{
  "token": "b4dc4fe3-30f1-43ea-bf32-87349e3b8cb5",
  "key": "48297953007cfb44e3451902d39c5bacf7a4f49369591e511de9f174afa90288f35b4a940a54dfe9f110ac30634443f24667c673fb65ef771954236c51e9b036",
  "cidr_whitelist": null,
  "readonly": false,
  "created": "2017-08-15T07:42:40.811Z",
  "updated": "2017-08-15T07:42:40.811Z"
}

$ curl -u test-chrisdickinson:some-password -X POST \
  -d '{"password": "some-password", "readonly": true, "cidr_whitelist": ["192.168.1.1/32"]}' \
  -H 'content-type: application/json' https://registry.npmjs.org/-/npm/v1/tokens | json
{
  "token": "86150c99-c17e-4ec8-99a4-d11b8c5fe3ad",
  "key": "fa06877427ed1fd75abd41c2fe12fa12b100e93b4dd00df46f653fa67de8b9ec38dd6982401adf360c3d380af432f619434b30199ec92332b96e42ce2802a0c9",
  "cidr_whitelist": [
    "192.168.1.1/32"
  ],
  "readonly": true,
  "created": "2017-08-15T07:44:32.151Z",
  "updated": "2017-08-15T07:44:32.151Z"
}
```

Returns:

- `200` with a [**Token**] object. The `token` attribute will be populated with
  a valid UUID. This UUID will never be displayed again by the registry.
- `401` with an [**Error**] object. In addition to the global authentication
  rules, if the `password` supplied in the body was invalid for the current user,
  this error will be returned.

* * *

### Token Delete

```bash
curl -X DELETE \
     https://registry.npmjs.org/-/npm/v1/tokens/token/<UUID>
```

Remove a token from the system. :warning: At present, **this does not evict the
token from cache.** It will continue to be valid for **one hour.**

- :crystal_ball: In the future, deleting the token will evict it from cache.

Returns:

- `204 No Content` on success.

[Date]: #date
[Error]: #error
[Page]: #page
[Token Creation Request]: #token-creation-request
[Token]: #token
[**Error**]: #error
[**Page**]: #page
[**Token Creation Request**]: #token-creation-request
[**Token**]: #token
[User Update]: #user-update
[Token List]: #token-list
[Token Create]: #token-create
[**Token Create**]: #token-create
[Token Delete]: #token-delete
[ref-userupdate]: ./profile.md#user-update
[ref-authn]: #how-to-authenticate
[Login Request]: #login-request
[Login Response]: #login-response
[**Two Factor Authentication**]: #two-factor-authentication
[**Login Request**]: #login-request
[**Login Response**]: #login-response
[Login]: #login
[**Login**]: #login
[Legacy Status]: #legacy-status
[**Legacy Status**]: #legacy-status

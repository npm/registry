# Org Memberships

## Table of Contents

- [Routes](#routes)
- [Objects](#objects)
    - [Error]
    - [Date]
    - [Roster]
    - [Membership Detail]
    - [Change Membership Request]
- [API Methods](#api-methods)
    - [Org Roster]
    - [Org Membership Replace]
    - [Org Membership Delete]

* * *

## Routes

| Method      | Route                             | Name                      |
| ----------- | --------------------------------- | ------------------------- |
| `GET`       | `/-/org/:scope/user`              | [Org Roster]              |
| `PUT`       | `/-/org/:scope/user`              | [Org Membership Replace]  |
| `DELETE`    | `/-/org/:scope/user`              | [Org Membership Delete]   |

* * *

## Objects

**Attributes marked with `// *` may not always appear in the object.**

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

### Roster

An object mapping usernames to org roles.

```javascript
{
  [username]: "developer" |
              "admin" |
              "owner"
}
```

* * *

### Membership Detail

```javascript
{
  org: {
    name: String,
    size: Number    // the current roster count of the org
  },
  user: String,
  role: "developer" |
        "admin" |
        "owner"
}
```

* * *

### Change Membership Request

```javascript
// "*" indicates a field that may not always appear
{
  "user": String,
  "role": "developer" |   // * -- defaults to "developer" if not given
          "admin" |
          "owner" |
          "team-admin" |  // archaic, synonymous with "admin"
          "super-admin"   // archaic, synonymous with "owner"
}
```

* * *

## API Methods

### Org Roster

```bash
curl -u user:pass \
     https://registry.npmjs.org/-/org/SCOPE/user
```

Get the set of members for the org identified by `SCOPE`. Returns
a [**Roster**] object on success.

* * *

### Org Membership Replace

```bash
curl -u user:pass \
     -X PUT \
     -H 'content-type: application/json' \
     -d <Change Membership Request> \
     https://registry.npmjs.org/-/org/SCOPE/user
```

Add or change a user's membership in the org identified by `SCOPE`. Takes
a [**Change Membership Request**] and returns a [**Membership Detail**]
object.

Returns:

- `201` with a [**Membership Detail**] object on success.
- `400` with an [**Error**] if `user` is omitted from the request body.

* * *

### Org Membership Delete

```bash
curl -u user:pass \
     -X DELETE \
     -H 'content-type: application/json' \
     -d {"user": USERNAME} \
     https://registry.npmjs.org/-/org/SCOPE/user
```

Remove a user from an org. The user will be removed from all teams managed by
the org.

Returns:

- `204` on success.

* * *

[Org Roster]: #org-roster
[Org Membership Replace]: #org-membership-replace
[Org Membership Delete]: #org-membership-delete
[Change Membership Request]: #change-membership-request
[Roster]: #roster
[Date]: #date
[Error]: #error
[**Change Membership Request**]: #change-membership-request
[**Membership Detail**]: #membership-detail
[Membership Detail]: #membership-detail
[**Roster**]: #roster
[**Error**]: #error

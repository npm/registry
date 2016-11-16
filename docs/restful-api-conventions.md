# REST proposal

Because API consistency is cheapest to apply at time of construction, and
because API inconsistency can slow future development, npm's
registry team has adopted the following guidelines when constructing REST APIs:

## URL Structures

### Versioning

Registry APIs will use in-URL versioning to represent **major changes** in API
versions. They will use header-based versioning to represent **minor** and
**patch**-level changes. Public versioning of the API in _any form_ should happen
_infrequently_.

In-URL versions will appear as follows:

`/vN`, where `N` is a positive integer — `/v1/`, `/v2/`, `/v1000/`.

Where many services share a namespace, as in the registry-frontdoor case, the
service may identify itself before the version:

`/service/vN`

### General Route Structure

The general structure describes all parts following the version prefix.

The structure should follow this pattern:

```
method  route                         name        attrs
------------------------------------------------------------
GET     /classes/                     list
POST    /classes/                     create      [optional]
GET     /classes/class/<public-id>    detail
POST    /classes/class/<public-id>    update      [optional]
PUT     /classes/class/<public-id>    replace     [optional]
DELETE  /classes/class/<public-id>    delete      [optional]
```

Resources are grouped by how they relate to the next highest resource
(`classes`). At the top level, they are grouped by how they relate to the
system the API describes. The second namespace is the name of the type of
resource (`class`). This leads to a `/scopes/scope/chrisdickinson` pattern, in
practice. The `class` namespace separates the namespaces of *public
identifiers* from the namespace of *relation-wide* resources. For example, a
general purpose "number of packages in the system" resource would be hosted at
`/packages/count`, and would not collide with the `detail` view of the `count`
package, at `/packages/package/count`.

When exposing a relation between resources — for instance, "a scope owns
multiple teams" this structure can be repeated:

```
GET /scopes/scope/<scope>/teams/
GET /scopes/scope/<scope>/teams/team/<name>
```

Desired responses for the given endpoints are listed below, in the
**Responses** section.

Notably, wherever it is safe to do so internally, we should make allowances for
incorrectly escaped scoped packages — that is, `@foo/bar` — treating those
requests in a DWIM fashion while alerting the improper use through logs and
metrics.

**HTTP Verb Use**:

* `GET` is nondestructive, and used _only_ for fetching resources.
* `POST` is destructive, and used to partially update resources or to create resources
  that are mounted at different routes: e.g., `POST /users/user → /scopes/scope/username`.
* `PUT` is nondestructive and idempotent, and is used to create or completely replace
  a resource. It should be preferred for resource creation where possible. It should be
  be used in instances where the user knows and _may control_ the public identifier of
  the new resource.
* `DELETE` is destructive and on success, subsequent `GET` requests at the
  given resource should return 404.

## Common Responses

<a href="#all-objects">**All objects**</a>: All objects that are returned from the API should include a
`"urls"` stanza, describing the locations of related resources, sans the host.
A client must be able to use these urls to construct subsequent requests
without modifying their contents.

Values in the `"urls"` stanza should NOT include the host, but MUST be absolute
paths.

```javascript
{
  "created": "2015-02-26T01:39:15.414Z",
  "updated": "2015-02-26T01:39:15.414Z",
  "type": "user",
  "parent": {
    "name": "chrisdickinson",
    "email": "chris@neversaw.us",
    "email_verified": true,
    "created": "2015-02-26T01:39:15.414Z",
    "updated": "2015-10-13T18:22:29.214Z",
    "deleted": null,
    "resource": {},
    "urls": {
      "detail": "/v2/scopes/scope/chrisdickinson"
    }
  },
  "urls": {
    "detail": "/v2/scopes/scope/chrisdickinson",
    "refresh": "/v2/scopes/scope/chrisdickinson/refresh",
    "packages": "/v2/scopes/scope/chrisdickinson/packages"
  }
}
```

<a href="#creation">**Creation**</a>: When using `PUT` for creation, differentiate between update and
creation: use `201` for creation, and `200` for update. If using `POST` to
create a resource, return a `Location` header pointing at the newly created
resource.

<a href="#list">**List endpoints**</a>: Use the common list response envelope:

```javascript
{
  "objects": [
    {...},
    {...}
  ],
  "total": 20000,
  "urls": {
    "next": "/url?page=10",
    "prev": "/url?page=8"
  }
}
```

`objects` should contain serialized constituent objects, with individual `urls`
stanzas. The overall `next` or `prev` urls may be omitted from the list
envelope urls stanza when there is no next or previous page to which to
navigate. If possible, `total` should be populated with an integer count of the
total number of objects the list represents.

<a href="#errors">**Errors**</a>: All errors should be in the form `{message: 'error message'}`.
Internally, error responses should include a `stack` property.

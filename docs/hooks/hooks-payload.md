

When a hook is triggered, data about the change is sent to the hook [endpoint](../creating-and-managing-hooks.md#creating-a-new-hook) via HTTP POST.
The body is JSON.  `application/json` content type.

it looks like this.

## payload

the payload consists of a few nested objects.

### envelope

the outer object has basic information about the event type and the package that changed.

| name      | type | meaning |
| ----      | ---- | ------- |
| `event`   | string | this is a string like [payload object type]:[event] event may be `change`,`publish`,`owner-rm` etc. see below for a full list |
| `name`    | string | the name of the object. for packages it's the package name. lodash, request etc. |
| `type`    | string | this is the object type. `"package"` is the only supported type for now. we may have more types in the future or hooks that can apply to more than one type of changed object |
| `version` | string | the version of this payload envelope. when we make changes to this payload object we'll change this version.|
| `sender`  | object | an object with a username property. this is the npm username of the person who owns this webhook|
| `payload` | object | for package type hooks this is the package document. The same data as curling the registry directly https://registry.npmjs.com/packagename`|
| `change`  | object | when available, this contains attributes that were modified and used to identify the change type. the keys in this object are variable to event type.|
| `time`    | int    | this is the unix timestamp in ms. its useful for logging but practically it provides a nonce that helps folks identify/prevent replay attacks|

```
{ 
    "event": "package:change",
    "name': "lodash",
    "type": "package",
    'version': "1.0.0",
    'hookOwner': { "username": "soldair"},
    'payload': {..package metadata..},
    'change': {..change data object..},
    "time": 1464791199635
}
```

### events

| event | meaning | npm command |
| ----- | ------- | ----------- |
| `package:star` | a package was starred | `npm star package` |
| `package:unstar` | a pckage was unstarred | `npm unstar package` |
| `package:publish` | a package was published | `npm publish` |
| `package:unpublish` | a package version was unpublished | `npm unpublish @foo/private` |
| `package:owner` | added an owner (maintainer) | `npm owner add username` |
| `package:owner-rm` | removed an owner | `npm owner rm username` |
| `package:deprecate` | a version was deprecated | `npm deprecate package@1.0.0 "dont use this"` |
| `package:undeprecate` | a version was undeprecared | `npm deprecate package@1.0.0 ""` |
| `package:change` | this is a catchall event. if for some reason im unable to identify the change type this type will be served | unknown | 

### change object

todo.

## security

hooks are delivered with an `x-npm-signature` header. this value is produced by taking your hook [secret](../creating-and-managing-hooks.md#creating-a-new-hook) and.... todo


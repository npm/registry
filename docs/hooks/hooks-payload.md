

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
| `hookOwner`  | object | an object with a username property. this is the npm username of the person who owns this webhook|
| `payload` | object | for package type hooks this is the package document. The same data as curling the registry directly https://registry.npmjs.com/packagename |
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
| `package:owner-rm` | removed an owner | `npm owner rm username`|
| `package:dist-tag` | dist tag added | `npm dist-tag add package@1.0.0 beta` |
| `package:dist-tag-rm` | dist tag removed | `npm dist-tag rm package beta` |
| `package:deprecated` | a version was deprecated | `npm deprecate package@1.0.0 "dont use this"` |
| `package:undeprecated` | a version was undeprecared | `npm deprecate package@1.0.0 ""` |
| `package:change` | this is a catchall event. if for some reason im unable to identify the change type this type will be served | unknown | 

### change object

Each change type (event) sets a key in this object. 

A publish to a latest version will be event type `package:publish`, but since it also sets the latest tag the change object would contain the dist-tag field `{"version":"1.0.0","dist-tag":"latest"}`

| key | type | description |  event | 
|-----|------|-------------|--------|
| version | string | the version that was changed | `package:publish`, `package:unpublish` |
| dist-tag | string | the tag that was changed | `package:dist-tag`, `package:dist-tag-rm` |
| user | string | the npm username of the starer | `package:star`, `package:unstar` |
| maintainer | string | the npm username of the changed maintainer/owner | `package:owner`, `package:owner-rm` |
| deprecated | string | the version of the package that was deprecated | `package:deprecated`, package:undeprecated` |


## security

hooks are delivered with an `x-npm-signature` header. this value is produced by taking your hook [secret](../creating-and-managing-hooks.md#creating-a-new-hook) 

validating this signature is done like this https://github.com/npm/npm-hook-receiver/blob/master/index.js#L24 

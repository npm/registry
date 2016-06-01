# Hooks API Endpoints

### `POST /v1/hooks/hook`

Create a webhook. All of the following fields are required in the post body:

| name | type | meaning |
| ---- | ---- | ------- |
| `type` | string | type of object being watched; `package`, `scope`, or `owner` |
| `name` | string | name of package or organization/user scope to watch, e.g, `package`, `@scope`, `@scope/package` etc. |
| `endpoint` | uri | full uri of the endpoint to post the notification to |
| `secret` | string | a secret shared between the registry & you; used to sign the payload |

The registry responds with the full hook object created, including the ID it generated for the hook.

Example:

```js
var opts = {
  uri: 'https://registry.npmjs.org/-/npm/v1/hooks/hook/',
  json: {
    type: 'scope',
    name: '@npmcorp',
    endpoint: 'https://example.com/webhook',
    secret: 'this is certainly very secret'
  }
};
Request.post(opts, function(err, res, body)
{
    console.log('just created hook with id=' + body.id);
});
```

### `PUT /v1/hooks/hook/:id`

Update the hook with the given id. The `endpoint` and `secret` fields are required in the body. Responds with the hook object if a matching hook was found & updated.

Example:

```js
var opts = {
  uri: 'https://registry.npmjs.org/-/npm/v1/hooks/hook/abcdefgh',
  json: {
    endpoint: 'https://example.com/webhook',
    secret: 'that secret turned out to be not very secret'
  }
};
Request.put(opts, function(err, res, body)
{
    console.log('just updated hook with id=' + body.id);
});
```

### `DELETE /v1/hooks/hook/:id`

Remove the hook with the given id. Responds with the hook object if a matching hook was found & removed.

### `GET /v1/hooks`

List all hooks currently set up for the authed user. Response body is a JSON object:

```js
{
    objects: []
}
```

`objects` is an array of hook objects.

You may pass filtering arguments by query string:

* `package`: filter by package name; regexp patterns are *not* parsed
* `limit`: return at most N hooks
* `offset`: start at the Nth hook (use with limit for pagination)

### `GET /v1/hooks/hook/:id`

Fetch the hook object for the single named hook.

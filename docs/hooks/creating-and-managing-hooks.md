# Creating and Managing Hooks

You can create hooks using [wombat](https://www.npmjs.com/package/wombat), or you can write
an application that can create and manage them programmatically.
These docs will focus on the latter.

## Creating a New Hook

To create a new hook, you'll need to have your application `POST` to
the `https://registry.npmjs.org/-/npm/v1/hooks/hook` endpoint.

The following fields are required in the post body:

| name | type | meaning |
| ---- | ---- | ------- |
| `type` | string | type of object being watched; `package`, `scope`, or `owner` |
| `name` | string | name of package or organization/user scope to watch, e.g, `@scope` or `package` |
| `endpoint` | uri | full uri of the endpoint to post the notification to |
| `secret` | string | a secret shared between the registry & you; used to sign the payload |

Note: make sure to save the `secret` as it'll be important for receiving `POST`s
from the registry on hook event changes.

If a hook is successfully created, The registry will respond with the full hook object created,
including the ID it generated for the hook.

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

If the hook is not created, the registry will return an error.

## Updating a Hook

To update a hook, you'll need to send a `PUT` request to 
`https://registry.npmjs.org/-/npm /v1/hooks/hook/:id`

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

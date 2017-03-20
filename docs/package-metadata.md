# Package Metadata

You can request _package metadata_ from this endpoint:

`GET https://registry.npmjs.org/:package`

The registry responds with a JSON-formatted string containing metadata for the package named, either in full or abbreviated form depending on what you request in the `Accept` header. If you provide no Accept header, the full document is returned. To request an _abbreviated_ document with only the fields required to support installation, set the `Accept` header in your request to the following string:

`application/vnd.npm.install-v1+json`

A more typical accept header might request json as a fallback, like this:

`application/vnd.npm.install-v1+json; q=1.0, application/json; q=0.8, */*`

The formats are described in detail below, but you can compare them by making requests using any tool you like. To request package metadata documents with [httpie](https://httpie.org):

```shell
http GET https://registry.npmjs.org/npm
http GET https://registry.npmjs.org/npm Accept:application/vnd.npm.install-v1+json
```

With the less user-friendly but ubiquitous curl:

```shell
curl -H "Accept: application/vnd.npm.install-v1+json" https://registry.npmjs.org/npm
```

## Abbreviated metadata format

This form of the package metadata exists to provide a smaller payload designed to support installation. It contains a whitelisted subset of fields selected for their usefulness.

The abbreviated metadata document contains the following fields:

• `name`: the package name
• `modified`: ISO string of the last time this package was modified
• `dist-tags`: a mapping of dist tags to the versions they point to
• `versions`: a mapping of version numbers to objects containing the information needed to install that version

Each version object contains the following fields:

• `name`: the package name
• `version`: the version string for this version
• `dependencies`: a mapping of other packages this version depends on to the required semver ranges
• `optionalDependencies`:  an object mapping package names to the required semver ranges of _optional_ dependencies
• `devDependencies`: a mapping of package names to the required semver ranges of _development_ dependencies
• `bundleDependencies`: an array of dependencies bundled with this version
• `bundledDependencies`: a synonym for `bundleDependencies`
• `peerDependencies`: a mapping of package names to the required semver ranges of _peer_ dependencies
• `bin`: a mapping of bin commands to set up for this version
• `directories`: an array of directories included by this version
• `dist`: an object with at least two fields:
    - `tarball`: the url of the tarball containing the payload for this package
    - `shasum`: the SHA-1 sum of the tarball
    - (in the future) a SHA-2 512 sum of the tarball
• `engines`: node versions required for this package version
• `_hasShrinkwrap`: `true` if this version has a shrinkwrap that must be used to install it; `false` if not.

## Full metadata format

Top-level fields:

• `_id`: the package name, used as an ID in CouchDB
• `_rev`: the revision number of this version of the document in CouchDB
• `author`: object with email & name fields
• `bugs`: as given in package.json, for the latest version
• `contributors`:
• `description`: the short text description of the package, from the latest version
• `dist-tags`: a mapping of dist tags to versions. Every package will have a `latest` tag defined.
• `homepage`: as given in package.json, for the latest version
• `keywords`:  as given in package.json, for the latest version
• `license`: the []
• `maintainers`: An array of author objects. Not authoritative, but a cache.
• `name`: the package name
• `readme`: The first 64K of the README data for the most-recently published version of the package.
• `readmeFilename`: The name of the file from which the readme data was taken.
• `repository`: as given in package.json, for the latest version
• `time`:
• `users`: an object whose keys are the users who have starred this package
• `versions`: a mapping of semver-compliant version numbers to version data

Each package version data object contains all of the fields in the abbreviated document, plus at least the following:

• `_id`
• `_nodeVersion`
• `_npmUser`
• `_npmVersion`
• `_shasum`
• `contributors`
• `description`
• `keywords`
• `main`
• `maintainers`

The full version object will also contain any other fields the package publisher chose to include in their package.json file for that version.

## Example: tiny-tarball

Here is a very tiny package, with only one version and no dependencies. Abbreviated package metadata:

```json
{
    "dist-tags": {
        "latest": "1.0.0"
    },
    "modified": "2015-05-16T22:27:54.741Z",
    "name": "tiny-tarball",
    "versions": {
        "1.0.0": {
            "_hasShrinkwrap": false,
            "directories": {},
            "dist": {
                "shasum": "bbf102d5ae73afe2c553295e0fb02230216f65b1",
                "tarball": "https://registry.npmjs.org/tiny-tarball/-/tiny-tarball-1.0.0.tgz"
            },
            "name": "tiny-tarball",
            "version": "1.0.0"
        }
    }
}
```

Full package metadata:

```json
{
    "_attachments": {},
    "_id": "tiny-tarball",
    "_rev": "3-085759e977d42299e64a35aedc17d250",
    "author": {
        "email": "ben@npmjs.com",
        "name": "Ben Coe"
    },
    "description": "tiny tarball used for health checks",
    "dist-tags": {
        "latest": "1.0.0"
    },
    "license": "ISC",
    "maintainers": [
        {
            "email": "ben@npmjs.com",
            "name": "bcoe"
        }
    ],
    "name": "tiny-tarball",
    "readme": "# TinyTarball\n\ntiny-tarball used for health checks\n\n**don't unpublish me!**\n",
    "readmeFilename": "README.md",
    "time": {
        "1.0.0": "2015-03-24T00:12:24.039Z",
        "created": "2015-03-24T00:12:24.039Z",
        "modified": "2015-05-16T22:27:54.741Z"
    },
    "versions": {
        "1.0.0": {
            "_from": ".",
            "_id": "tiny-tarball@1.0.0",
            "_nodeVersion": "1.5.0",
            "_npmUser": {
                "email": "bencoe@gmail.com",
                "name": "bcoe"
            },
            "_npmVersion": "2.7.0",
            "_shasum": "bbf102d5ae73afe2c553295e0fb02230216f65b1",
            "author": {
                "email": "ben@npmjs.com",
                "name": "Ben Coe"
            },
            "description": "tiny tarball used for health checks",
            "directories": {},
            "dist": {
                "shasum": "bbf102d5ae73afe2c553295e0fb02230216f65b1",
                "tarball": "https://registry.npmjs.org/tiny-tarball/-/tiny-tarball-1.0.0.tgz"
            },
            "license": "ISC",
            "main": "index.js",
            "maintainers": [
                {
                    "email": "bencoe@gmail.com",
                    "name": "bcoe"
                }
            ],
            "name": "tiny-tarball",
            "scripts": {
                "test": "echo \"Error: no test specified\" && exit 1"
            },
            "version": "1.0.0"
        }
    }
}
```

The size difference for

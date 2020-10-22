# Public Registry API

## Table of Contents

- [Objects](#objects)
  - [Registry](#registry)
  - [Package](#package)
  - [Version](#version)
- [Filtering](#filtering)
- [Errors](#errors)
- [Endpoints](#endpoints)
  - [Meta Endpoints](#meta-endpoints)
    - [`GET·/`](#get)
    - [`GET·/-/all`](#get-all)
    - [`GET·/-/`]
  - [Package Endpoints](#package-endpoints)
    - [`GET·/{package}`](#getpackage)
    - [`GET·/{package}/{version}`](#getpackageversion)
    - [`GET·/-/v1/search`](#get-v1search)

## Objects

### Registry

- `db_name`: "registry"
- `doc_count`: 376841,
- `doc_del_count`: 354,
- `update_seq`: 2889325,
- `purge_seq`: 0,
- `compact_running`: false,
- `disk_size`: 2098360443,
- `data_size`: 1485346312,
- `instance_start_time`: "1471680653634734",
- `disk_format_version`: 6,
- `committed_update_seq`: 2889325

### Package

- `_id`: the package name
- `_rev`: latest revision id
- `name`: the package name
- `description`: description from the `package.json`
- `dist-tags`: an object with at least one key, `latest`, representing dist-tags
- `versions`: a List of all [Version](#version) objects for the Package
- `time`: an object containing a `created` and `modified` time stamp
- `author`: object with `name`, `email`, and or `url` of author as listed in `package.json`
- `repository`: object with `type` and `url` of package repository as listed in `package.json`
- `_attachments`: http://docs.couchdb.org/en/2.0.0/intro/api.html#attachments
- `readme`: full text of the `latest` version's `README`

### Version

- `name`: package name,
- `version`: version number
- `homepage`: homepage listed in the `package.json`
- `repository`: object with `type` and `url` of package repository as listed in `package.json`
- `dependencies`: object with dependencies and versions as listed in `package.json`
- `devDependencies`: object with devDependencies and versions as listed in `package.json`
- `scripts`: object with scripts as listed in `package.json`
- `author`: object with `name`, `email`, and or `url` of author as listed in `package.json`
- `license`: as listed in `package.json`
- `readme`: full text of `README` file as pointed to in `package.json`
- `readmeFilename`: name of `README` file
- `_id`: `<name>@<version>`
- `description`: description as listed in `package.json`
- `dist`: and object containing a `shasum` and `tarball` url, usually in the form of `https://registry.npmjs.org/<name>/-/<name>-<version>.tgz`
- `_npmVersion`: version of npm the package@version was published with
- `_npmUser`: an object containing the `name` and `email` of the npm user who published the package@version
- `maintainers`: and array of objects containing `author` objects as listed in `package.json`
- `directories`:???

## Filtering

## Errors

## Endpoints

### Meta Endpoints

#### `GET·/`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |

```
{
  "db_name": "registry",
  "doc_count": 399172,
  "doc_del_count": 354,
  "update_seq": 3351374,
  "purge_seq": 0,
  "compact_running": false,
  "disk_size": 2118398075,
  "data_size": 1600835750,
  "instance_start_time": "1475135224217333",
  "disk_format_version": 6,
  "committed_update_seq": 3351374
}
```

#### `GET·/-/all`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |

### Package Endpoints

#### `GET·/{package}`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |
| package | String | **Path**  | ✅         | the name of the package |

This endpoint responds with the package metadata document, sometimes informally called a "packument" or "doc.json". The format of the response is described in detail in the [package metadata documentation](responses/package-metadata.md).

#### `GET·/{package}/{version}`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |
| package | String | **Path** | ✅         | the name of the package |
| version | String | **Path** | ✅         | a version number or `latest`        |

#### `GET·/-/v1/search`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |
| text | String | **Query** | ❌         | full-text search to apply |
| size | integer | **Query** | ❌         | how many results should be returned (default 20, max 250) |
| from | integer | **Query** | ❌         | offset to return results from |
| quality | float | **Query** | ❌         | how much of an effect should quality have on search results |
| popularity | float | **Query** | ❌         | how much of an effect should popularity have on search results |
| maintenance | float | **Query** | ❌         | how much of an effect should maintenance have on search results |

_Note: the values of `quality`, `popularity`, and `maintenance` are normalized into a unit-vector provide values between `0 - 1` for each to modify weightings, e.g., to return
 results based solely on `quality`, set `quality=1.0`, `maintenance=0.0`, `popularity=0.0`._

**response format:**

```json
{
  "objects": [
    {
      "package": {
        "name": "yargs",
        "version": "6.6.0",
        "description": "yargs the modern, pirate-themed, successor to optimist.",
        "keywords": [
          "argument",
          "args",
          "option",
          "parser",
          "parsing",
          "cli",
          "command"
        ],
        "date": "2016-12-30T16:53:16.023Z",
        "links": {
          "npm": "https://www.npmjs.com/package/yargs",
          "homepage": "http://yargs.js.org/",
          "repository": "https://github.com/yargs/yargs",
          "bugs": "https://github.com/yargs/yargs/issues"
        },
        "publisher": {
          "username": "bcoe",
          "email": "ben@npmjs.com"
        },
        "maintainers": [
          {
            "username": "bcoe",
            "email": "ben@npmjs.com"
          },
          {
            "username": "chevex",
            "email": "alex.ford@codetunnel.com"
          },
          {
            "username": "nexdrew",
            "email": "andrew@npmjs.com"
          },
          {
            "username": "nylen",
            "email": "jnylen@gmail.com"
          }
        ]
      },
      "score": {
        "final": 0.9237841281241451,
        "detail": {
          "quality": 0.9270640902288084,
          "popularity": 0.8484861649808381,
          "maintenance": 0.9962706951777409
        }
      },
      "searchScore": 100000.914
    }
  ],
  "total": 1,
  "time": "Wed Jan 25 2017 19:23:35 GMT+0000 (UTC)"
}
```

**special search qualifiers:**

Special search qualifiers can be provided in the full-text query:

* `author:bcoe`: Show/filter results in which `bcoe` is the author
* `maintainer:bcoe`: Show/filter results in which `bcoe` is qualifier as a maintainer
* `keywords:batman`: Show/filter results that have `batman` in the keywords
  * separating multiple keywords with
    * `,` acts like a logical `OR`
    * `+` acts like a logical `AND`
    * `,-` can be used to exclude keywords
* `not:unstable`: Exclude packages whose version is `< 1.0.0`
* `not:insecure`: Exclude packages that are insecure or have vulnerable dependencies (based on the [nsp](https://nodesecurity.io/) registry)
* `is:unstable`: Show/filter packages whose version is `< 1.0.0`
* `is:insecure`: Show/filter packages that are insecure or have vulnerable dependencies (based on the [nsp](https://nodesecurity.io/) registry)
* `boost-exact:false`: Do not boost exact matches, defaults to `true`

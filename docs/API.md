# Public Registry API

## Table of Contents

- [Objects](#objects)
  - [Registry](#registry)
  - [Change](#change)
  - [Package](#package)
  - [Version](#version)
- [Filtering](#filtering)
- [Errors](#errors)
- [Endpoints](#endpoints)
  - [Meta Endpoints](#meta-endpoints)
    - [`GET·/`](#get)
    - [`GET·/_changes`](#getchanges)
    - [`GET·/-/all`](#getall)
  - [Package Endpoints](#package-endpoints)
    - [`GET·/{package}`](#getpackage)
    - [`GET·/{package}/{version}`](#getpackageversion)

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

### Change

- `seq`
- `id`
- `changes`
  - `rev`

### Package

- `_id`: the package name
- `_rev`: latest revision id
- `name`: the package name
- `description`: description from the `package.json`
- `dist-tags`: an object with at least one key, `latest`, representing dist-tags 
- `versions`: a List of all [Version](#version) objects for the Package
- `time`: an object containing a `created` and `modifited` time stamp
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

#### `GET·/_changes`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |

```
{"results":[
{"seq":1,"id":"_design/scratch","changes":[{"rev":"1-bbc2dd850a61789f96c1bbd219084d93"}]},
{"seq":2,"id":"_design/app","changes":[{"rev":"1-4392b71a1168beefb0ba5173b63b24c3"}]},
{"seq":3,"id":"Reston","changes":[{"rev":"1-de734a2e9145f8c74a2de62a385f49b5"}]},
{"seq":4,"id":"asyncevents","changes":[{"rev":"1-15c03acf9851eb582f77c192a4df9fe3"}]},
...
```

#### `GET·/-/all`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |

### Package Endpoints

#### `GET·/{package}`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |
| package | String | **Path**  | ✅         | the name of the package |
| version | String | **Query** | ❌         | a version number        |

#### `GET·/{package}/{version}`

| Name     | Value     | Kind     | Required?     | Notes     |
|------    |-------    |------    |-----------    |-------    |
| package | String | **Path** | ✅         | the name of the package |
| version | String | **Path** | ✅         | a version number        |

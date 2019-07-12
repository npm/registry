# Replication API

## Table of Contents

- [Overview](#overview)
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

## Overview

The npm registry is a collection of services that allow clients to
install and publish packages to the npm official registry.

At it's core, the registry is a [CouchDB app] which serves metadata
for all the packages it contains. npm makes it easy to replicate this
service by providing 2 services:

- ~[`http://skimdb.npmjs.com`] (unscoped packages ONLY)~ DEPRECATED
- [`https://replicate.npmjs.com`] (scoped packages)

This documentation will explain the API exposed by both of these 
services. While they are very similar, at some points they diverge.
We'll be sure to point out those differences where they happen :)

### Configuring the CLI

The npm CLI client is configurable to be used with a different registry
than the official npm registry (which is the default). For more information
on this, check out the [npm CLI registry doc page]!

[npm CLI registry doc page]: https://docs.npmjs.com/misc/registry

### Public Registry API

The replication services we offer are slightly different than the
public registry API that the website and CLI client use. If you'd like
to read up about those instead, head over to the [Registry API Docs].

[Registry API Docs]: REGISTRY-API.md

### The Follower Pattern

The primary pattern of using these services is to build a follower.
If you'd rather just jump into building something with the registry
data, head on over to [this tutorial] to get started!

[this tutorial]: https://github.com/npm/registry-follower-tutorial

## Objects

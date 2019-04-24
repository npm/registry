> npm registry API docs
# introduction

In this article, you'll learn:

- a brief overview of the npm registry's historical architecture
- an introduction to CouchDB databases

## background

[CouchDB] is a database that uses JSON for documents, JavaScript for MapReduce indexes,
and regular HTTP for its API. CouchDB comes with an out-of-the-box functionality, called
[CouchApp], which is a web application served directly from CouchDB. npm's registry was 
originally exclusively a CouchApp, but as it has grown, it has slowly been separated out
into many different services, some of which are no longer backed by the original CouchDB,
and are backed by [Postgres].

Because npm cares deeply about backwards compatibility, as the npm regsitry
grows out of its CouchApp and CouchDB origins, all of the original endpoints and
functionality of the original API will continue to be supported.

This documentation serves as the source of truth for what has been, is, and will continue to
be support by the npm registry team.

[CouchDB]: http://couchdb.org
[CouchApp]: http://docs.couchdb.org/en/1.6.1/couchapp/index.html
[Postgres]: https://www.postgresql.org/

## introduction to couchdb



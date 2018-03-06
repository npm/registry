# so you want to write a follower
> ch-ch-ch-ch-changes

This tutorial will teach you how to write a generic boilerplate
NodeJS application that can manipulate, respond to, broadcast, analyze,
and otherwise play with package metadata as it changes in the npm registry.

Wait...what? Why?

Here's the deal: do you want to have some fun with the `package.json` data
from every version of every package in the npm registry? Some neat ideas:

- Find all the package `README`s that mention dogs
- Discover how many package authors are named "Kate"
- Calculate how many dependency changes occur on average in a major version
   bump

And more! So stop waiting and write a follower!

Checkout the final code here: https://github.com/npm/registry-follower-tutorial

## prerequisites

In order to follow along with this tutorial you'll need [NodeJS] and
[npm]. I recommend installing these using a version manager; I use [nvm].

[NodeJS]: https://nodejs.org
[npm]: https://www.npmjs.com/
[nvm]: https://github.com/creationix/nvm

## application setup

Let's set up our application:

1. Create a directory called `follower-tutorial`. 
  (`mkdir follower-tutorial`)
2. Move into that directory (`cd follower-tutorial`).
3. Initialize an npm project by typing `npm init --yes`. This will create a
  `package.json` with default values.
4. Create a file called `.gitignore` and add the line `node_modules` to it.

Our application currently looks like this:

```
+ follower-tutorial
  |- .gitignore
  |- package.json
```

## dependencies

Our application is going to depend on a couple super helpful npm packages:

- [`changes-stream`]: This package gives us access to a [stream] of changes
  from the npm registry's CouchDB. We'll listen for and respond to events
  from this stream in our app. These events represent changes in the npm
  registry.
- [`request`]: This package allows us to make HTTP requests. We'll use this
  to retrieve the current total number of changes currently in the 
  database so that we can optionally end our progarm when it has received
  all the current changes.

[`changes-stream`]: https://www.npmjs.com/package/changes-stream
[stream]: https://nodejs.org/api/stream.html
[`request`]: https://www.npmjs.com/package/request

To install these packages we'll type:

```
npm install changes-stream request --save
```
... which will add both of our dependencies to our `package.json`.

## set up a changes stream

Let's start writing our application now! We'll be writing our application
in an `index.js` file at the root of our `follower-tutorial` application
directory:

```
+ follower-tutorial
  |- .gitignore
  |- index.js       // <-- here's where our app goes
  |- package.json
```

The first thing we'll do inside our `index.js` is use the `changes-stream`
package to create a new `ChangesStream` to listen to listen to the npm
registry. To do so we'll write:

```
1  const ChangesStream = require('changes-stream');
2 
3  const db = 'https://replicate.npmjs.com';
4 
5  var changes = new ChangesStream({
6    db: db
7  });
``` 

Let's talk about what's happening here:

- On line 1, we require the `changes-stream` package
- On line 3, we save the URL of the npm registry db
- On lines 5-7, we create a new ChangesStream instance, passing it an
  options object that points to our db

Now that we've created a changes stream, let's listen to it! To do this, we
write:

```
9  changes.on('data', function(change) {
10   console.log(change);
11 });
```

Let's test it out: Run this application by typing:

```
node index.js
```

If everything is working correctly, you'll see something like this start
**streaming** through your console:

```
{ seq: 445,
  id: 'CompoundSignal',
  changes: [ { rev: '5-a0695c30fdaa3471246ef0cd6c8a476d' } ] }
{ seq: 446,
  id: 'amphibian',
  changes: [ { rev: '5-1a864e76d844e90bf6c63cb94303b593' } ] }
{ seq: 447,
  id: 'aop',
  changes: [ { rev: '9-9acc0139df57a1db2604f13f12b500f2' } ] }
{ seq: 448,
  id: 'dynamo-schema',
  changes: [ { rev: '5-bf8052c0d4b6e80e6664625137efd610' } ] }
{ seq: 451,
  id: 'password-reset',
  changes: [ { rev: '21-948e6633799ffd56a993c3fb144d1728' } ] }
```

If you don't see that, and/or got an error, reread the sample code in this
section and be sure you don't have any typos! If you continue having
trouble, [file an issue on this repo].

[file an issue on this repo]: /npm/registry/issues/new

Otherwise... Congrats! You have a successful registry follower. Hurry up
and hit `ctrl-c` - this stream won't ever exit the way we've written it
now!

## moar data please

So our follower works! But it's not that great right now because we don't 
really have all that much interesting data. Let's look at what we have
right now:

```
{ seq: 446,
  id: 'amphibian',
  changes: [ { rev: '5-1a864e76d844e90bf6c63cb94303b593' } ] }
```

- `seq`: the packages order in the sequence of change events
- `id`: the name of the `package` (sometimes this is something else! we'll
  get to that in a bit tho, it doesn't matter too much right now.)
- `changes`: an array containing a single object, with a single key `rev`
  that point to a change id

Let's be real: this data is not *that* interesting. Where's the good stuff?
It turns out that the fun data is in a key called `doc` that we need to
tell our `ChangesStream` instance, `changes`, to specifically grab. To do
this, we'll add `include_docs: true` to the options object we pass to the
`ChangesStream` constructor.

Once we've told our ChangesStream to `include_docs`, we get some new
awesome data.  This new data lives off of a key called `doc` on the `change`
object we received from the stream.

The two changes we make to our code look like this:

```
5  var changes = new ChangesStream({
6    db: db,
7    include_docs: true            // <- this is the thing we're adding
8  })
9
10 changes.on('data', function(change) {
11   console.log(change.doc)      // <- so that we can add `.doc` here
12 });
```

Let's test it out by running `node index.js`. Assuming you've made all the
changes we described above you should be seeing a LOT more data. Here's a
summary of what you get:

- `_id`: the name of the package
- `_rev`: the revision id
- `name`: the name of the package
- `description`: the package description
- `'dist-tags'`: an object with all dist-tag names and versions
- `versions`: a nested object where every version is a key, and an object of
  all of the package metadata for that key is the value (this includes: 
  `main`, `directories`, `dependencies`, `scripts`, `engines`, `bin`, 
    `devDependencies`, and more)
- `maintainers`: an array of objects, each representing info about a 
  maintainer (name, email, website)
- `time`: timestamps for every version of the package published, plus
  `created` and `modified`
- `author`: an object representing the author of the package (name, email
  website)
- `repository`: an object representing the location of the package code,
  e.g. `{ type: 'git', url: 'git://github.com/my/gitrepo.git' }`

Take a moment and play around with your application by exploring the
different pieces of data you can get from this stream. You may notice that
some nested structures appear like `[Object]` in your console. You can
print those out by adding `JSON.stringify` to your log, like this:

```
console.log(JSON.stringify (change.doc,null,' '));
```

...which will ensure that the nested objects aren't flattened (e.g. appear
like `[Object]`).

Note: you'll have to `ctrl-C` out of your application every time you run
it. It's still a neverending stream! In the next section we'll explain how
to make it stop. 

## a neverending stream

As we mentioned in the previous section, our follower currently won't ever
stop! Let's dive a little deeper into why that's the case:

Our application functions by listening for an event called `data` from
a stream coming out of npm's registry, each event hands us an object that
represents a `change` in the registry... and the registry is changing all
the time!

Luckily, our db endpoint gives us some useful info to help us consume just
the current changes as they exist at the time of access, and then stop the 
process.

Navigate your browser to our db url:

```
https://replicate.npmjs.com
```

...you should see something that looks like this:

```
{
  "db_name": "registry",
  "doc_count": 345391,
  "doc_del_count": 355,
  "update_seq": 2496579,
  "purge_seq": 0,
  "compact_running": false,
  "disk_size": 1713074299,
  "data_size": 1320944467,
  "instance_start_time": "1466084344558224",
  "disk_format_version": 6,
  "committed_update_seq": 2496579
}
```

Let's talk about what some of these mean:

- `doc_count`: is the number of documents that the db contains
- `update_seq`: is the number of changes that are stored in the db

`update_seq` is the important bit of information here. As stated, it 
represents the number of `change` resources contained in the db. Remember
the `seq` attribute on the `change` object we received from the stream?
That number counts up until `update_seq`! This means that we can use this
number to tell our follower to stop when `change.seq` meets or exceeds the
`update_seq` value, signifying that it has processed all the changes in the
db up until the time we accessed the `update_seq` value.

That was a lot of words, let's take a look at what this would look like in
code.

```
2  const Request = require('request');
...
11 Request.get(db, function(err, req, body) {        // <- make a request to the db
12   var end_sequence = JSON.parse(body).update_seq; // <- grab the update_seq value
13   changes.on('data', function(change) {
14     if (change.seq >= end_sequence) {             // <- if we're at the last change
15       process.exit(0);                            // <- end the program successfully ("0")
16     }
17     console.log(change.doc);
18   }) 
19 });
```

Let's walk through what this code is doing:

- On line 2, we are require the `request` package
- On line 11, we are making a request to our db using the `request` package
- On line 12, we parse the response from our request and grab the `update_seq`
  value.
- On line 13, on every `data` event, we check to see if the `change.seq`
  value we get is greater than or equal to `update_seq`. Why `>=` and
  not just equal? Remember that the registry is *always* changing, and there's
  a good chance it will change while we are following it! Using `>=` means that
  we can account for the change that happens while our application is running. 
- On line 15, we end our program. We send the value `0` to `process.exit` to
  indicate thaat we are ending the program successfully, i.e. not with an error.

Ok! Given this code, our application will now run for all the current changes in the
registry and then exit. Take a moment and give it a go! Note: There are a lot of
changes, so this can take up to an hour.

## clean up

So our follower is pretty much done! However, there's a few things that aren't quite
right about our data. Let's do that now so we can finish up.

Firstly, remember the `id`/`_id` key we recieve from our changes stream? We had
identified that as being the name of the package, but that was a generalization.
It turns out that there are actually 2 types of things in the changes db: changes
and "design docs".

"Design docs"? What? Right. To understand this requires understanding how CouchDB works
a bit. One way to program with CouchDB is to write an application **within** the db.
At this point, npm is moving away from this structure, but at one point (and still!)
the registry was/is written as a CouchDB application. These applications exist as 
"design docs" inside the db, so when we receive data from the db, *sometimes* we receive
these design docs. If you watched your follower closely, you'd notice that 
*sometimes* it's logging `undefined`. Those are the "design docs".

To ignore these files, we can just check to see if a `change` is an actual package
by checking if it has a `name`. We can accomplish this by checking if 
`change.doc.name` has a value before we do anything with the `change` data. In our
code, this looks like this:

```
...
17 if (change.doc.name) {             // <-- make sure the change is a change
18   console.log(change.doc);
19 }
``` 

Ok, so we're **almost** done. Actually, we are totally done. But there's one last
thing we can do to make our data even better: we can normalize our data so that
it is nearly exactly the same as the CLI uses and is returned by http://registry.npmjs.com.

To do this, we'll add *one more* dependency to our application: [`normalize-registry-metadata`].

[`normalize-registry-metadata`]: https://github.com/npm/normalize-registry-metadata

First things first: let's install this package and save it to our `package.json`:

```
npm install normalize-registry-metadata --save
```

Next, we require in our `index.js`:

```
3  const Normalize = require(`normalize-registry-metadata`);
```

Lastly, let's call `Normalize()` on the `change` data before we log it to the console:

```
...
18   console.log(Normalize(change.doc));        // <-- we only have to change this line!
...
```

And we're done! You can double check that your code is correct by looking at the 
complete code [here].

[here]: https://github.com/ashleygwilliams/registry-follower-tutorial/blob/master/index.js

## forever follower

Don't want to stop? Want to write a persistent follower? You can move
foward with our app as it is currently written, however you'll likely have
a better experience replacing `changes-stream` with 
[`concurrent-couch-follower`] which is safer for operations that may 
require async (like a file write!). [`concurrent-couch-follower`] remembers
the last change you processed and can start back from there if at some point
you need to restart the program.

[`concurrent-couch-follower`]: https://github.com/npm/concurrent-couch-follower

## go forth and make something awesome!

We're seriously excited about what you'll build. Please share with us on
twitter ([@npmjs])! And please don't hesitate to ask for help in the issues
on this repo :)

[@npmjs]: https://twitter.com/npmjs

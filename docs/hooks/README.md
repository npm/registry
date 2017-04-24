## Introduction to Hooks

## What is Hooks?

Hooks is a new service from npm that sends information to the web whenever
a package or a package-scope's information changes on the registry. This could
be a new version of a package being publish, a new package added to an
organization, a new owner added to a package, and much more.

We're super excited to offer this service as it affords our community the
ability to build and set up integrations.

## How's it Work?

1. End users (that's you!) build or set up an integration application.
2. That application subscribes to a set of change events on the npm registry,
   i.e. builds a set of hooks.
3.  When a package is changed, we'll send an HTTP POST payload to the URI
   you've configured in your application for your hook.

## What Events are Currently Supported?

Hooks can be added to:
  - specific packages, i.e. `lodash`
  - all packages in a user scope, i.e. `@username`
  - all packages in an organization scope, i.e. `@npmcorp`
  - all packages maintained by a user, i.e. `substack`

#### Any plans on expanding the offerings?

Definitely! But we want to make sure we make these work well before expanding. :wink:

## Who can use Hooks?

Any npm user can use Hooks!

## How many Hooks can I have?

**You can have 100 hooks total**.

So what's that mean? You can put all 100 on a single package or scatter them across
100 different packages.

You can make a hooks and use it in several places. For example, you can use hooks
to trigger integration testing, trigger a deploy, make an announcement in a chat
channel, or trigger an update of your own packages.

<hr/>

Excited about hooks? Check out the [API Documentation]!

[API Documentation]: ./endpoints.md

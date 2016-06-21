# Fastly Configuration
> https://github.com/npm/varnish-configs

[Fastly] is a self-described Real-Time CDN. All requests
to npm's registry first go through Fastly, with the goal of:

- controlling access via IP
- assigning a Backend per request
- caching responses

The configurations are [Varnish configuration files].
They are written in [Varnish Configuration Language] (VCL), a
small domain-specific language (DSL) designed used to define
request handling and document caching policies for the Varnish
HTTP accelerator. A Varnish configuration file customizes the
handling of requests made to a specific [Service]. A Service
represents the configuration for a website, app, api, or
anything else to be served through Fastly.

Within npm, there are 5 configurations, 3 of which pertain to
the registry Services:

- [`registry.npm.red`]: testing/staging server
- [`registry.npmjs.org`]: production server
- [`registrytwo.npmjs.com`]: canary server for new versions

In this documentation, we'll focus on [`registry.npmjs.org`].

## Controlling Access via IP

When a request is initially made to `registry.npmjs.org`, the
first check that is made is to a defined `ACL` (Access Control
List) which determines whether the requesting IP will be allowed
to access the service.

Currently, `registry.npmjs.org` blocks 2 IPs: one as a result of
spam, another as a result of security violations.

## Assigning a Backend per Request

After we've determined whether the requesting IP is allowed to
access the service, we define [Directors].

We define directors for 3 sub-Services of the registry Service:

- package metadata (front-door)
- tarballs (package data)
- registry search*

*Note: registry search is a separate service from www search.

...for 4 locations (see the [Fastly Network Map] for more details):

- west
- east
- eu (packages and tarballs only)
- syd (tarballs only)

A Director is responsible for balancing requests among a group of
[Backends]. A Backend is an address (IP or domain) from which Fastly
pulls content. For the various services and locations, npm defines a
different number of Backends:

|       | package meta data       | tarballs    | search      |
|------ |------------------------ |------------ |-----------  |
| east  | 2 Backends              | 2 Backends  | 1 Backend   |
| west  | 3 Backends, (1 Canary)  | 2 Backends  | 1 Backend   |
| eu    | 1 Backend               | 1 Backend   | N/A         |
| syd   | N/A                     | 1 Backend   | N/A         |

N.B. The registry search Service is not based on location and randomly
chooses east or west, regardless of requesting IP.

## Caching Responses

In order to improve performance, npm caches the response to many requests.

When a request is received, Fastly will check if the Backend response is
something that should be cached:

- Responses to requests for package tarballs, metadata, search data, etc., 
are cached for specific durations depending on the type of data they 
contain. 
- Requests that fail and scoped packages are not cached. 

More specificially:

- If the Backend response (`beresp`) is Not Found (`404`), Internal Server
  Error (`500`), or Service Unavailable (`503`) and a restart hasn't been
  tried, try a restart

  - If that restart doesn't work, tell Varnish to not use any cache and not
    to cache. More specifically:

    - set the header's `Cache-Control` property to `max-age=0`
      (NOTE: this affects both Varnish and the browser)
    - set the time-to-live (`ttl`) to 0
    - not perform a lookup to see if an object is in cache
    - not cache the response from the origin

- If the request URL contains an `@`, indicating it is a scoped package,
  do not cache, exactly as above. Any caching would potentially leak 
  information. (Scoped packages are potentially private.)

- Otherwise:
  
  - If the request is for a tarball, cache for 6 hours, or
  - If the request is a search request, cache for 1 hour, or
  - Cache for 5 minutes

  then deliver the cached response.

## Putting It All Together

Now that we have defined an ACL, Directors, and a caching strategy, we can
synthesize a coherent request handling process. This is defined in `vcl_recv`.

  1. Receive all requests to `registry.npmjs.org`
  2. Reject all IPs listed in the the ACL
  3. Determine if the request is for a tarball (package data) or something else

From here, we handle the request in one of two ways:

### Tarball Requests

Tarball requests are requests for package data. These requests hit the 
package servers. These are only `GET` requests.

If a tarball request is received:
    
1. On the first attempt, try to download the tarball from EC2/nginx. To
  do so we'll need to update the request URL:

  - **Scoped Packages**

    ```
    /@wombat/coolpkg/-/coolpkgname-2.16.2.tgz --->
    /npm/public/registry/@/@wombat/coolpkg/_attachments/coolpkg-2.16.2.tgz
    ```

  - **All Other Packages**

    ```
    /foo/-/foo-0.2.2.tgz --->
    /npm/public/registry/f/foo/_attachments/foo-0.2.2.tgz
    ```

2. After updating the URL, send the request to the appropriate `frontdoor` server
   location. To do this, npm:

      1. Looks at the global region of the [Fastly Point of Presence (POP)] handling the request. 
      2. Attempts to match it to the closest datacenter in npm's infrastructure, by passing
        the request to one of several tarball Directors which will distribute the incoming requests
        among several local Backends.
      3. If this request fails, perhaps because a specific server has gone offline,
        send the request to `west`, where we there is the most redundancy.

### Non-Tarball Requests

Non-Tarball Requests can be two types of requests:

- **New, Update, or Delete Actions on packages:** `PUT`, `POST`, or `DELETE`
   requests.
- **Requests for package meta-, user, search, or view data:** `GET` requests.

If a non-tarball request is received we first determine whether the request is of
one of the above two types, and then proceed accordingly.

#### New, Update, or Delete Actions on Packages

Similar to Tarball Requests, as explained above, npm will send the request to
the appropriate server by:

  A. Look at the global region of the [Fastly Point of Presence (POP)] handling the request.
  B. Attempt to match it to the closest datacenter in npm's infrastructure, by passing
     the request to one of two Directors (`east` and `west`) which will distribute the
     incoming requests  among several local Backends.

  NOTE: These requests need to be directed to a packages Backend that has a local
  [validate-and-store (V&S)] (private npm service), so only `east` and `west` locations
  are supported right now.

  C. If this request fails, perhaps because a specific server has gone offline,
     send the request to `west`, where we there is the most redundancy.

- Send `US-East`, `US-Central`, and `EU` to the `packages_east` Director
- Send all other request locations to the `packages_west` Director 

#### Requests for Package Meta-, User, Search, or View Data

- If the request URL includes `/-/all`, the request should be handled by the
  `search_` Director, which randomly chooses an east or west Backend.
- Else, tell Varnish to not perform a lookup to see if an object is in cache and
  do not cache the response from the origin. (`return(pass)`)

[Fastly]: https://www.fastly.com/
[`registry.npm.red`]: https://github.com/npm/varnish-configs/tree/master/fastly/registry.npm.red
[`registry.npmjs.org`]: https://github.com/npm/varnish-configs/tree/master/fastly/registry.npmjs.org
[`registrytwo.npmjs.com`]: https://github.com/npm/varnish-configs/tree/master/fastly/registrytwo.npmjs.com
[Varnish Configuration Language]: http://varnish-cache.org/trac/wiki/VCL
[Directors]: https://docs.fastly.com/api/config#director
[Backends]: https://docs.fastly.com/api/config#backend
[Varnish configuration files]: https://docs.fastly.com/api/config#vcl
[Service]: https://docs.fastly.com/api/config#service
[Fastly Network Map]: https://www.fastly.com/network-map
[Fastly Point of Presence (POP)]: https://docs.fastly.com/guides/about-fastly-services/fastly-pop-locations
[validate-and-store (V&S)]: https://github.com/npm/validate-and-store

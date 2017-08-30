# package download counts

There is a public api that gives you download counts by package and time range.

Our blog has an explanation of [how npm download counts work](http://blog.npmjs.org/post/92574016600/numeric-precision-matters-how-npm-download-counts), including "what counts as a download?"

## Data source

npm's raw log data is continuously written to a series of buckets on AWS S3. Once per day, soon
after UTC midnight, a map-reduce cluster is spun up that crunches the previous day's logs and
pushes them into the database. Because this is UTC this creates some slightly unintuitive results,
e.g. if you are on the west coast on the 19th of September, the data for the 19th of September will
become available at 5pm (because UTC already moved to the 20th) during the winter, but not until 6pm
during the summer, because the US observes daylight savings but UTC is fixed.

## Point values

Gets the total downloads for a given period, for all packages or a specific package.

<code>GET https://api.npmjs.org/downloads/point/{period}[/{package}]</code>

### Examples

<dl>
	<dt>All packages, last day:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/last-day">/downloads/point/last-day</a></dd>
	<dt>All packages, specific date:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/2014-02-01">/downloads/point/2014-02-01</a></dd>
	<dt>Package "express", last week:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/last-week/express">/downloads/point/last-week/express</a></dd>
	<dt>Package "express", given 7-day period:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/2014-02-01:2014-02-08/express">/downloads/point/2014-02-01:2014-02-08/express</a></dd>
	<dt>Package "@slack/client", last 30 days:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/last-month/@slack/client">/downloads/point/last-month/@slack/client</a></dd>
	<dt>Package "jquery", specific month:</dt>
	<dd><a href="https://api.npmjs.org/downloads/point/2014-01-01:2014-01-31/jquery">/downloads/point/2014-01-01:2014-01-31/jquery</a></dd>
</dl>

### Parameters

Acceptable values are:

<dl>
	<!--
	<dt>all-time</dt>
	<dd>Gets total downloads.</dd>
	-->
	<dt>last-day</dt>
	<dd>Gets downloads for the last available day. In practice, this will usually be "yesterday" (in GMT) but if stats for that day have not yet landed, it will be the day before.</dd>
	<dt>last-week</dt>
	<dd>Gets downloads for the last 7 available days.</dd>
  	<dt>last-month</dt>
	<dd>Gets downloads for the last 30 available days.</dd>
</dl>

### Output

The following incredibly simple JSON is the output:

```javascript
{
  downloads: 31623,
  start: "2014-01-01",
  end: "2014-01-31",
  package: "jquery"
}
```

If you have not specified a package, that key will not be present. The start and end dates are inclusive.

## Ranges

Gets the downloads per day for a given period, for all packages or a specific package.

<code>GET https://api.npmjs.org/downloads/range/{period}[/{package}]</code>

### Examples

<dl>
	<dt>Downloads per day, last 7 days</dt>
	<dd><a href="https://api.npmjs.org/downloads/range/last-week">/downloads/range/last-week</a></dd>
	<dt>Downloads per day, specific 7 days</dt>
	<dd><a href="https://api.npmjs.org/downloads/range/2014-02-07:2014-02-14">/downloads/range/2014-02-07:2014-02-14</a></dd>
	<dt>Downloads per day, last 30 days</dt>
	<dd><a href="https://api.npmjs.org/downloads/range/last-month/jquery">/downloads/range/last-month/jquery</a></dd>
	<dt>Downloads per day, specific 30 day period</dt>
	<dd><a href="https://api.npmjs.org/downloads/range/2014-01-03:2014-02-03/jquery">/downloads/range/2014-01-03:2014-02-03/jquery</a></dd>
</dl>

### Parameters

Same as for `/downloads/point`.

### Output

Responses are very similar to the point API, except that downloads is now an array of days with downloads on each day:

```javascript
{
	downloads: [
		{
			day: "2014-02-27",
			downloads: 1904088
		},
		..
		{
			day: "2014-03-04",
			downloads: 7904294
		}
	],
	start: "2014-02-25",
	end: "2014-03-04",
	package: "somepackage"
}
```

As before, the package key will not be present if you have not specified a package.

## Bulk Queries

To perform a bulk query, you can hit the range or point endpoints with a comma
separated list of packages rather than a single package, e.g.,

`/downloads/point/last-day/npm,express`

__Important:__ As of this writing, 19 April 2017, *scoped* packages are not yet supported in bulk queries. So you cannot request `/downloads/point/last-day/@slack/client,@iterables/map` yet. We *do* plan to support scoped modules in bulk queries as soon as we can make bulk auth checks efficiently. The latter work is in progress.

## Limits

Bulk queries are limited to at most *128* packages at a time and at most *365 days* of data.

All other queries are limited to at most *18 months* of data. The earliest date for which data will be returned is January 10, 2015.

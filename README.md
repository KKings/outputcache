# Outputcache

[![Version](https://img.shields.io/npm/v/outputcache.svg)](https://www.npmjs.com/package/outputcache)
[![License](https://img.shields.io/npm/l/outputcache.svg)](https://www.npmjs.com/package/outputcache)
[![Build Status](https://travis-ci.org/mpfdavis/outputcache.svg?branch=master)](https://travis-ci.org/mpfdavis/outputcache)
[![Test Coverage](https://coveralls.io/repos/mpfdavis/outputcache/badge.svg?branch=master&service=github)](https://coveralls.io/github/mpfdavis/outputcache?branch=master)

Simple to use, load-tested, outputcache for node - supports caching the response of **res.send, res.render, res.json**, **res.redirect** and all headers

## Install

```bash
  npm install outputcache
```

## Why?

Under heavy load, Node applications can suffer poor performance even if they make use of cached data. 
Outputcache stores the final response and returns it immediately for future requests matching the original request signature.

- Fast - returns raw response and uses optimised version of LRU cache by default (Maps)
- Simple - honours original headers and requires few code changes

## Initialize

```js
const OutputCache = require('outputcache');
//example
const cache = new OutputCache({ varyByQuery: true, logger: winston, varyByCookies: ['geoId', 'country'] });
```

### Options

- `ttl`: *(default: `600`)* the standard ttl as number in seconds for each cache item  
- `maxItems`: *(default: `1000`)* the number of items allowed in the cache before older, unused items are pushed out
- `useCacheHeader`: *(default: `true`)* use the max-age cache header from the original response as ttl by default. If you set this to false the options.ttl or default ttl will be used and the cache-control response will not be modifed to match. This enables you to respond with a different cache control header to the actual in-memory ttl if desired
- `varyByQuery`: *(default: `false`)* cache key will use the request path by default, setting this to true will include the querystring for more complex cache keys
- `varyByCookies`: *(default: `[]`)* accepts an array of cookie names - the cache key will include the value of the named cookie if found in the request
- `logger`: *(default: null)* pass in an instance of your chosen logger for logging info - expects a debug method to be available unless a specific logLevel is supplied
- `logLevel`: *(default: debug)* the log level outputcache should log hits with if a logger is provided
- `skip3xx`: *(default: false)* never cache 3xx responses
- `skip4xx`: *(default: false)* never cache 4xx responses
- `skip5xx`: *(default: false)* never cache 5xx responses
- `noHeaders`: *(default: false)* do not add X-Output-Cache headers to the response - useful for security if you wish to hide server technologies
- `staleWhileRevalidate`: set staleWhileRevalidate header for client as ttl in seconds

**Note:** varyByCookies requires you to register a cookie parser such as the popular 'cookie-parser' module in your application before Outputcache.

## Usage

- Can be used as a route or global middleware. 
- Should be placed as early as possible in the response lifecycle to maximise performance.

### Methods

```js
.middleware => // (req, res, next)

```

### Example

The following example places Outputcache before "data.middleware" - this ensures all cached responses return as soon as possible and avoid any subsequent data gathering or processing.

```js

const OutputCache = require('outputcache');
const cache = new OutputCache({logger: winston});

app.get('/', cache.middleware, data.middleware, (req, res) => {
  
  //hit this once per ttl
  const someData = res.locals.data.hello;      
  res.render('helloworld', someData);
  
});

app.get('/api/:channel', cache.middleware, data.middleware, (req, res) => {

  //hit this once per ttl    
  const someData = res.locals.data.hello;      
  res.json(someData);
  
});

```

## Headers

- Will add 'X-Output-Cache' to the response headers with a ms (miss) or ht +ttl (hit) value, this can be useful for troubleshooting
- Will honour headers assigned to the original response, including for redirects

## Cache skip

A cache skip (miss) will occur for all requests when:

- The querystring collection contains 'cache=false' value pair.
- The request has an 'X-Output-Cache' header set with the value 'ms'
- A cookie X-Output-Cache with value 'ms' was set on the original response
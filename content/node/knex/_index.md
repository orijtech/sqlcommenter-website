---
title: "Knex.js"
date: 2019-05-29T08:40:11-06:00
draft: false
weight: 1
tags: ["knex", "knex.js", "query-builder", "node", "node.js", "express", "express.js"]
---

![](/images/knex-logo.png)

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
    - [Manually](#manually)
    - [Package manager](#package-manager)
- [Usage](#usage)
    - [Plain knex wrapper](#plain-knex-wrapper)
    - [Express middleware](#express-middleware)
- [Fields](#fields)
    - [Options](#options)
        - [Options examples](#options-examples)
- [End to end examples](#end-to-examples)
    - [Source code](#source-code)
    - [Results](#results)
- [References](#references)

#### Introduction

This package is in the form of `Knex.Client.prototype.query` wrapper whose purpose is to augment a SQL statement right before execution, with
information about the controller and user code to help correlate them with SQL statements emitted by Knex.js.

Besides plain knex.js wrapping, we also provide a wrapper for the following frameworks:

{{<card-vendor href="#express-middleware" src="/images/express_js-logo.png">}}

### Requirements

Name|Resource
---|---
Knex.js|https://knexjs.org/
Node.js|https://nodejs.org/

### Installation

We can add integration into our applications in the following ways:

#### Manually

Please read [installing sqlcommenter-nodejs from source](/node/#install-from-source)

#### Package manager
Add to your package.json the dependency
{{<highlight json>}}
{
    "@google-cloud/sqlcommenter-knex": "*"
}
{{</highlight>}}
and then run `npm install` to get the latest version or 
{{<highlight shell>}}
npm install @google-cloud/sqlcommenter-knex --save
{{</highlight>}}

### Usage
#### Plain knex wrapper
{{<highlight javascript>}}
const {wrapMainKnex} = require('@google-cloud/sqlcommenter-knex');
const Knex = require('knex');
wrapMainKnex(Knex);

// Now you can create the knex client.
const knex = Knex(options);
{{</highlight>}}

#### Express middleware
This wrapper/middleware can be used as is or better with [express.js](https://expressjs.com/)
{{<highlight javascript>}}
const {wrapMainKnexAsMiddleware} = require('@google-cloud/sqlcommenter-knex');
const Knex = require('knex');
const app = require('express')();

// This is the important step where we set the middleware.
app.use(wrapMainKnexAsMiddleware(Knex));

// Now you can create the knex client.
const knex = Knex(options);
{{</highlight>}}

### Fields

In the database server logs, the comment's fields are:

* comma separated key-value pairs e.g. `route='%5Epolls/%24'`
* values are SQL escaped i.e. `key='value'`
* URL-quoted except for the equals(`=`) sign e.g `route='%5Epolls/%24'`. so should be URL-unquoted

Field|Format|Description|Example
---|---|---|---
`db_driver`|`<database_driver>:<version>`|URL quoted name and version of the database driver|`db_driver='knex%3A0.16.5'`
`route`|`<the route used>`|The URL-quoted route used to match the express.js controller|`route='%5Epolls/%24'`
`traceparent`|`<XX-TRACE_ID-SPAN_ID-TRACE_OPTIONS>`|The serialized [W3C TraceContext.Traceparent](https://www.w3.org/TR/trace-context/#traceparent-field)|`traceparent='00-f5e3fa7fb15a461dbf3b03690e4bd5e1-e6de66630cd19b9a-01'`
`tracestate`|`<KEY1=VALUE1,KEY2=VALUE2,...>`|The serialized [W3C TraceContext.Tracestate](https://www.w3.org/TR/trace-context/#tracestate-field)|`tracestate='congo%3Dt61rcWkgMzE%2Crojo%3D00f067aa0ba902b7'`

#### Options

When creating the middleware, one can optionally toggle attributes to be set in the comments by passing in the option `include` which is a map

```javascript
wrapMainKnexAsMiddleware(Knex, include={...});
```

Field|On by default
---|---
route|<div style="text-align: center">&#10004;</div>
tracestate|<div style="text-align: center">&#10060;</div>
traceparent|<div style="text-align: center">&#10060;</div>
db_driver|<div style="text-align: center">&#10060;</div>

Additionally, if the tracestate or traceparent fields are included, one can specify which tracing library the wrapper should collect data from. This can be done by providing a third argument, which is an object. Within that options object should be a field named TraceProvider, containing the string name of the tracing library (case-insensitive).

```javascript
wrapMainSequelizeAsMiddleware(Sequelize, include={...}, {TraceProvider: 'opencensus'});
```
Option Name|Associated Library
---|---
opencensus|https://opencensus.io/
opentelemetry|https://opentelemetry.io/

##### Options examples

{{<tabs "trace attributes" route db_driver "all set">}}

{{<highlight javascript>}}
wrapMainKnexAsMiddleware(Knex, include={
    traceparent: true,
    tracestate: true
});
{{</highlight>}}

{{<highlight javascript>}}
wrapMainKnexAsMiddleware(Knex, include={route: true});
{{</highlight>}}

{{<highlight javascript>}}
wrapMainKnexAsMiddleware(Knex, include={db_driver: true});
{{</highlight>}}

{{<highlight javascript>}}
// Manually set all the variables.
wrapMainKnexAsMiddleware(Knex, include={
    db_driver: true,
    route: true,
    traceparent: true,
    tracestate: true,
});
{{</highlight>}}

{{</tabs>}}

### End to end examples

#### Source code 

{{<tabs "With OpenCensus" "With Route" "With DB Driver" "With All Options Set">}}

{{<highlight javascript>}}
// In file app.js.
const tracing = require('@opencensus/nodejs');
const {B3Format} = require('@opencensus/propagation-b3');
const {ZipkinTraceExporter} = require('@opencensus/exporter-zipkin');
const Knex = require('knex'); // Knex to be wrapped say v0.0.1
const {wrapMainKnexAsMiddleware} = require('@google-cloud/sqlcommenter-knex');
const express = require('express');

const exporter = new ZipkinTraceExporter({
    url: process.env.ZIPKIN_TRACE_URL || 'localhost://9411/api/v2/spans',
    serviceName: 'trace-542'
});

const b3 = new B3Format();
const traceOptions = {
    samplingRate: 1, // Always sample
    propagation: b3,
    exporter: exporter
};

// start tracing
tracing.start(traceOptions);

const knexOptions = {
    client: 'postgresql',
    connection: {
        host: '127.0.0.1',
        password: '$postgres$',
        database: 'quickstart_nodejs'
    }
};
const knex = Knex(knexOptions); // knex instance

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the knex+express middleware with trace attributes 
app.use(wrapMainKnexAsMiddleware(Knex, {
    traceparent: true, 
    tracestate: true,
    route: false
}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    knex.raw('SELECT * from polls_question').then(function(polls) {
        const blob = JSON.stringify(polls);
        res.send(blob);
    }).catch(function(err) {
        console.log(err);
        res.send(500);
    });
});
app.listen(port, () => console.log(`Application listening on ${port}`));
{{</highlight>}}

{{<highlight javascript>}}
// In file app.js.
const Knex = require('knex'); // Knex to be wrapped say v0.0.1
const {wrapMainKnexAsMiddleware} = require('@google-cloud/sqlcommenter-knex');
const express = require('express');

const options = {
    client: 'postgresql',
    connection: {
        host: '127.0.0.1',
        password: '$postgres$',
        database: 'quickstart_nodejs'
    }
};
const knex = Knex(options); // knex instance

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the knex+express middleware with route
app.use(wrapMainKnexAsMiddleware(Knex, {route: true}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    knex.raw('SELECT * from polls_question').then(function(polls) {
        const blob = JSON.stringify(polls);
        res.send(blob);
    }).catch(function(err) {
        console.log(err);
        res.send(500);
    });
});
app.listen(port, () => console.log(`Application listening on ${port}`));
{{</highlight>}}


{{<highlight javascript>}}
// In file app.js
const Knex = require('knex'); // Knex to be wrapped say v0.0.1
const {wrapMainKnexAsMiddleware} = require('@google-cloud/sqlcommenter-knex');
const express = require('express');

const options = {
    client: 'postgresql',
    connection: {
        host: '127.0.0.1',
        password: '$postgres$',
        database: 'quickstart_nodejs'
    }
};
const knex = Knex(options); // knex instance

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the knex+express middleware with db driver
app.use(wrapMainKnexAsMiddleware(Knex, {db_driver: true}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    knex.raw('SELECT * from polls_question').then(function(polls) {
        const blob = JSON.stringify(polls);
        res.send(blob);
    }).catch(function(err) {
        console.log(err);
        res.send(500);
    });
});
app.listen(port, () => console.log(`Application listening on ${port}`));
{{</highlight>}}

{{<highlight javascript>}}
// In file app.js.
const tracing = require('@opencensus/nodejs');
const {B3Format} = require('@opencensus/propagation-b3');
const {ZipkinTraceExporter} = require('@opencensus/exporter-zipkin');
const Knex = require('knex'); // Knex to be wrapped say v0.0.1
const {wrapMainKnexAsMiddleware} = require('@google-cloud/sqlcommenter-knex');
const express = require('express');

const exporter = new ZipkinTraceExporter({
    url: process.env.ZIPKIN_TRACE_URL || 'localhost:9411/api/v2/spans',
    serviceName: 'trace-542'
});

const b3 = new B3Format();
const traceOptions = {
    samplingRate: 1, // Always sample
    propagation: b3,
    exporter: exporter
};

// start tracing
tracing.start(traceOptions);

const knexOptions = {
    client: 'postgresql',
    connection: {
        host: '127.0.0.1',
        password: '$postgres$',
        database: 'quickstart_nodejs'
    }
};
const knex = Knex(knexOptions); // knex instance

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the knex+express middleware with all attributes set
app.use(wrapMainKnexAsMiddleware(Knex, {
    traceparent: true,
    tracestate: true,
    route: true,
    db_driver: true
}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    knex.raw('SELECT * from polls_question').then(function(polls) {
        const blob = JSON.stringify(polls);
        res.send(blob);
    }).catch(function(err) {
        console.log(err);
        res.send(500);
    });
});
app.listen(port, () => console.log(`Application listening on ${port}`));
{{</highlight>}}

{{</tabs>}}

which after running by
```shell
$ node app.js 
Application listening on 3000
```

#### Results

On making a request to that server at `http://localhost:3000/polls/1000`, the PostgreSQL logs show:

{{<tabs "With OpenCensus" "With Route" "With DB Driver" "With All Options Set">}}

{{<highlight shell>}}
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question 
/*traceparent='00-11000000000000ff-020000ee-01',tracestate='brazzaville=t61rcWkgMzE,rondo=00f067aa0ba902b7'*/
{{</highlight>}}

{{<highlight shell>}}
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question 
/*route='%5E%2Fpolls%2F%1000'*/
{{</highlight>}}

{{<highlight shell>}}
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question 
/*db_driver='knex%3A0.0.1'*/
{{</highlight>}}

{{<highlight shell>}}
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question 
/*db_driver='knex%3A0.0.1',route='%5E%2Fpolls%2F%1000',traceparent='00-11000000000000ff-020000ee-01',tracestate='brazzaville=t61rcWkgMzE,rondo=00f067aa0ba902b7'*/
{{</highlight>}}

{{</tabs>}}

#### References

Resource|URL
---|---
Knex.js project|https://knexjs.org/
sqlcommenter on Yarn|`<FILL_ME_IN>`
sqlcommenter on npm|`<FILL_ME_IN>`
express.js|https://expressjs.com/

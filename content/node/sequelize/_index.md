---
title: "Sequelize.js"
date: 2019-05-29T08:40:11-06:00
draft: false
weight: 1
tags: ["sequelize", "sequelize.js", "query-builder", "node", "node.js", "express", "express.js"]
---

![](/images/sequelize-logo.png)

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
    - [Manually](#manually)
    - [Package manager](#package-manager)
- [Usage](#usage)
    - [Plain sequelize wrapper](#plain-sequelize-wrapper)
    - [Express middleware](#express-middleware)
- [Fields](#fields)
    - [Options](#options)
        - [Options](#options-example)
- [End to end example](#end-to-example)
    - [Source code](#source-code)
    - [Results](#results)
- [References](#references)

#### Introduction

This package is in the form of `Sequelize.Client.prototype.query` wrapper whose purpose is to augment a SQL statement right before execution, with
information about the controller and user code to help correlate them with SQL statements emitted by Sequelize.js.

Besides plain sequelize.js wrapping, we also provide a wrapper for the following frameworks:

{{<card-vendor href="#express-middleware" src="/images/express_js-logo.png">}}

### Requirements

* [Sequelize.js](http://docs.sequelizejs.com/)
* [Node.js](https://nodejs.org/)

### Installation

We can add integration into our applications in the following ways:

#### Manually

Please read [installing sqlcommenter-nodejs from source](/node/#install-from-source)

#### Package manager
Add to your package.json the dependency
{{<highlight json>}}
{
    "@sqlcommenter/sequelize": "*"
}
{{</highlight>}}

and then run `npm install` to get the latest version or

{{<highlight shell>}}
npm install @sqlcommenter/sequelize --save
{{</highlight>}}

### Usage
#### Plain sequelize wrapper
{{<highlight javascript>}}
const {wrapSequelize} = require('@sqlcommenter/sequelize');
const Sequelize = require('sequelize');

// Create the sequelize client.
const sequelize = new Sequelize(options);

// Finally wrap the sequelize client.
wrapSequelize(sequelize);
{{</highlight>}}

#### Express middleware
This wrapper/middleware can be used as is or better with [express.js](https://expressjs.com/)
{{<highlight javascript>}}
const {wrapSequelizeAsMiddleware} = require('@sqlcommenter/sequelize');
const Sequelize = require('sequelize');
const sequelize = new Sequelize(options);
const app = require('express')();

// Use the sequelize+express middleware.
app.use(wrapSequelizeAsMiddleware(sequelize));
{{</highlight>}}

### Fields

In the database server logs, the comment's fields are:

* comma separated key-value pairs e.g. `route='%5E%2Fpolls%2F'`
* values are SQL escaped i.e. `key='value'`
* URL-quoted except for the equals(`=`) sign e.g `route='%5Epolls/%24'`. so should be URL-unquoted

Field|Format|Description|Example
---|---|---|---
`client_timezone`|`<string>`|URL quoted name of the timezone used when converting a date from the database into a JavaScript date|`'+00:00'`
`db_driver`|`<sequelize>`|URL quoted name and version of the database driver|`db_driver='sequelize'`
`route`|`<the route used>`|The URL-quoted route used to match the express.js controller|`route='%5E%2Fpolls%2F`

#### Options
When creating the middleware, one can optionally toggle attributes to be set in the comments by passing in the option `include` which is a map

```javascript
wrapMainSequelizeAsMiddleware(Sequelize, include={...});
```

Field|On by default
---|---
client_timezone|No
db_driver|No
route|Yes
tracestate|Yes
traceparent|Yes

##### Options examples

{{<tabs "trace attributes" "client_timezone" route db_driver "all set">}}

{{<highlight javascript>}}
wrapMainSequelizeAsMiddleware(Sequelize, include={
    traceparent: true,
    tracestate: true
});
{{</highlight>}}

{{<highlight javascript>}}
wrapMainSequelizeAsMiddleware(Sequelize, include={route: true});
{{</highlight>}}

{{<highlight javascript>}}
wrapMainSequelizeAsMiddleware(Sequelize, include={client_timezone: true});
{{</highlight>}}

{{<highlight javascript>}}
wrapMainSequelizeAsMiddleware(Sequelize, include={db_driver: true});
{{</highlight>}}

{{<highlight javascript>}}
// Manually set all the variables.
wrapMainSequelizeAsMiddleware(Sequelize, include={
    client_timezone: true,
    db_driver: true,
    route: true,
    traceparent: true,
    tracestate: true,
});
{{</highlight>}}

{{</tabs>}}

### End to end examples

#### Source code 

{{<tabs "With OpenCensus" "With Route" "With DB Driver and CLIENT TIMEZONE" "With All Options Set">}}

{{<highlight javascript>}}
// In file app.js.
const tracing = require('@opencensus/nodejs');
const {B3Format} = require('@opencensus/propagation-b3');
const {ZipkinTraceExporter} = require('@opencensus/exporter-zipkin');
const {wrapSequelizeAsMiddleware} = require('@sqlcommenter/sequelize');
const Sequelize = require('sequelize');
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

// Using a connection URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the sequelize+express middleware with trace attributes 
app.use(wrapSequelizeAsMiddleware(sequelize, {
    traceparent: true, 
    tracestate: true,
    route: false
}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    sequelize.query('SELECT * from polls_question').then(function(polls) {
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
const Sequelize = require('sequelize');
const {wrapSequelizeAsMiddleware} = require('@sqlcommenter/sequelize');
const express = require('express');

// Using a connection URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the sequelize+express middleware with route
app.use(wrapSequelizeAsMiddleware(sequelize, {route: true}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    sequelize.query('SELECT * from polls_question').then(function(polls) {
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
const Sequelize = require('sequelize');
const {wrapSequelizeAsMiddleware} = require('@sqlcommenter/sequelize');
const express = require('express');

// Using a connection URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the sequelize+express middleware with db driver and timezone
app.use(wrapSequelizeAsMiddleware(sequelize, {
    db_driver: true, 
    client_timezone: true
}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    sequelize.query('SELECT * from polls_question').then(function(polls) {
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
const Sequelize = require('sequelize');
const {wrapSequelizeAsMiddleware} = require('@sqlcommenter/sequelize');
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

// Using a connection URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

const app = express();
const port = process.env.APP_PORT || 3000;

// Use the sequelize+express middleware with all attributes set
app.use(wrapSequelizeAsMiddleware(sequelize, {
    traceparent: true,
    tracestate: true,
    route: true,
    db_driver: true,
    client_timezone: true
}));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    sequelize.query('SELECT * from polls_question').then(function(polls) {
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

{{<tabs "With OpenCensus" "With Route" "With DB Driver and CLIENT TIMEZONE" "With All Set">}}

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
/*client_timezone:'%2B00%3A00',db_driver='sequelize%3A0.0.1'*/
{{</highlight>}}

{{<highlight shell>}}
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question 
/*client_timezone:'%2B00%3A00',db_driver='sequelize%3A0.0.1',route='%5E%2Fpolls%2F%1000',traceparent='00-11000000000000ff-020000ee-01',tracestate='brazzaville=t61rcWkgMzE,rondo=00f067aa0ba902b7'*/
{{</highlight>}}

{{</tabs>}}


#### References

Resource|URL
---|---
@sqlcommenter/sequelize on Yarn|`<FILL_ME_IN>`
@sqlcommenter/sequelize on npm|`<FILL_ME_IN>`
express.js|https://expressjs.com/

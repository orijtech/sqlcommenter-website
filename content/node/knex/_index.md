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
- [Usage](#usage)
    - [Plain knex wrapper](#plain-knex-wrapper)
    - [Express middleware](#express-middleware)
- [Fields](#fields)
- [End to end example](#end-to-example)
    - [Source code](#source-code)
    - [Results](#results)
- [References](#references)

#### Introduction

This package is in the form of `Knex.Client.prototype.query` wrapper whose purpose is to augment a SQL statement right before execution, with
information about the controller and user code to help correlate them with SQL statements emitted by Knex.js.

Besides plain knex.js wrapping, we also provide a wrapper for the following frameworks:

{{<card-vendor href="#express-middleware" src="/images/express_js-logo.png">}}

### Requirements

Since the middleware uses Knex.js's [`connection.execute_wrapper`](https://docs.knexproject.com/en/stable/topics/db/instrumentation/#connection-execute-wrapper) and because of that we require:

* [Knex.js 2.X](https://docs.knexproject.com/en/stable/faq/install)
* [Node.js](https://nodejs.org/)

#### Installation
Add to your package.json the dependency
{{<highlight json>}}
{
    "@sqlcommenter/knex": "*"
}
{{</highlight>}}

or by `npm install` as per:

{{<highlight shell>}}
npm install @sqlcommenter/knex
{{</highlight>}}

### Usage
#### Plain knex wrapper
{{<highlight javascript>}}
const {wrapMainKnex} = require('@sqlcommenter/knex');
const Knex = require('knex');
wrapMainKnex(Knex);

// Now you can create the knex client.
const knex = Knex(options);
{{</highlight>}}

#### Express middleware
This wrapper/middleware can be used as is or better with [express.js](https://expressjs.com/)
{{<highlight javascript>}}
const {wrapMainKnexAsMiddleware} = require('@sqlcommenter/knex');
const Knex = require('knex');
const app = require('express')();

// This is the important step where we set the middleware.
app.use(wrapMainKnexAsMiddleware(Knex));

// Now you can create the knex client.
const knex = Knex(options);
{{</highlight>}}

### Fields

In the database server logs, the comment's fields are:

* comma separated key-value pairs e.g. `db_type='postgresql'`
* values are SQL escaped i.e. `key='value'`
* URL-quoted except for the equals(`=`) sign e.g `route='%5Epolls/%24'`. so should be URL-unquoted

Field|Format|Description|Example
---|---|---|---
`db_driver`|`<database_driver>:<version>`|URL quoted name and version of the database driver|`db_driver='knex%3A0.16.5'`
`db_name`|`<name of database>`|URL quoted name and version of the database driver|`db_name='quickstart_nodejs'`
`db_type`|`<the type of database>`|URL quoted name of the type of database|`db_type='postgresql'`
`route`|`<the route used>`|The URL-quoted route used to match the express.js controller|`route='%5Epolls/%24'`

### End to end example

#### Source code
{{<tabs app package_json>}}
{{<highlight javascript>}}
// In file app.js.
const express = require('express');
const app = express();
const port = process.env.APP_PORT || 3000;
const {wrapMainKnexAsMiddleware} = require('@sqlcommenter/knex');

const options = {
    client: 'postgresql',
    connection: {
        host: '127.0.0.1',
        password: '$postgres$',
        database: 'quickstart_nodejs'
    }
};

const Knex = require('knex');
const knex = Knex(options);

// Use the knex+express middleware.
app.use(wrapMainKnexAsMiddleware(Knex));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    console.log(req.route);
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
{{<highlight json>}}
{
    "name": "knex_app",
    "dependencies": {
        "express": "*",
        "knex": "*",
        "pg": "*"
    }
}
{{</highlight>}}
{{</tabs>}}

which after running by
```shell
$ node app.js 
Application listening on 3000
```

#### Results

On making a request to that server at `http://localhost:3000/polls/1000`, the PostgreSQL logs show:
```shell
2019-06-03 14:32:10.842 PDT [32004] LOG:  statement: SELECT * from polls_question
/* db_driver='knex%3A0.16.5',db_name='quickstart_nodejs',db_type='postgresql',
db_version='11.3',route='%5E%2Fpolls%2F%3Aparam' */
```


#### References

Resource|URL
---|---
sqlcommenter on Yarn|`<FILL_ME_IN>`
sqlcommenter on npm|`<FILL_ME_IN>`
express.js|https://expressjs.com/

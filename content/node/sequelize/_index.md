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
- [Usage](#usage)
    - [Plain sequelize wrapper](#plain-sequelize-wrapper)
    - [Express middleware](#express-middleware)
- [Fields](#fields)
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

#### Installation
Add to your package.json the dependency
{{<highlight json>}}
{
    "@sqlcommenter/sequelize": "*"
}
{{</highlight>}}

or by `npm install` as per:

{{<highlight shell>}}
npm install @sqlcommenter/sequelize
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

### End to end example

#### Source code
{{<tabs app package_json>}}
{{<highlight javascript>}}
// In file app.js.
const express = require('express');
const app = express();
const port = process.env.APP_PORT || 3000;
const {wrapSequelizeAsMiddleware} = require('/Users/emmanuelodeke/Desktop/nodejs-sqlcommenter/packages/sequelize');

const Sequelize = require('sequelize');
const sequelize = new Sequelize({
    dialect: 'postgresql',
    password: '$postgres$',
    database: 'quickstart_nodejs'
});

// Use the sequelize+express middleware.
app.use(wrapSequelizeAsMiddleware(sequelize));

app.get('/', (req, res) => res.send('Hello, sqlcommenter-nodejs!!'));
app.get('^/polls/:param', function(req, res) {
    console.log(req.route);
    sequelize.query('SELECT * from polls_question').then(function(polls, metadata) {
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
    "name": "sequelize_app",
    "dependencies": {
        "express": "*",
        "sequelize": "*",
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
2019-06-03 15:09:35.575 PDT [32665] LOG:  statement: SELECT * from polls_question
/*client_timezone='%2B00%3A00',db_driver='sequelize',db_version='11.3.0',route='%5E%2Fpolls%2F%3Aparam'*/
```


#### References

Resource|URL
---|---
@sqlcommenter/sequelize on Yarn|`<FILL_ME_IN>`
@sqlcommenter/sequelize on npm|`<FILL_ME_IN>`
express.js|https://expressjs.com/

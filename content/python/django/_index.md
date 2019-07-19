---
title: "Django"
date: 2019-05-29T08:40:11-06:00
draft: false
weight: 1
tags: ["python", "django"]
---

![](/images/django-logo.png)

{{<card-vendor href="/python/django/aws" src="/images/aws-logo.png">}}
{{<card-vendor href="/python/django/gcp" src="/images/gcp-logo.png">}}
{{<card-vendor href="/python/django/local" src="/images/locally-logo.png">}}

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
    - [pip install](#pip-install)
    - [Source](#source)
- [Fields](#fields)
    - [Sample log entry](#sample-log-entry)
    - [Expected fields](#expected-fields)
- [References](#references)

#### Introduction

This package is in the form of [Django middleware]() whose purpose is to augment a SQL statement right before execution, with
information about the [controller and user code]() to help with later making database optimization decisions, after those statements
are examined from the database server's logs.

The middleware uses Django's `connection.execute_wrapper`.

### Requirements

The middleware uses Django's [`connection.execute_wrapper`](https://docs.djangoproject.com/en/stable/topics/db/instrumentation/#connection-execute-wrapper) and therefore requires [Django 2.0](https://docs.djangoproject.com/en/stable/faq/install) or later (which [support various versions](https://docs.djangoproject.com/en/stable/faq/install/#what-python-version-can-i-use-with-django) of [Python 3](https://www.python.org/downloads/)).

#### Installation
This middleware can be installed by any of the following:
{{<tabs pip source>}}
{{<highlight shell>}}
pip3 install sqlcommenter-django
{{</highlight>}}

{{<highlight shell>}}
git clone https://github.com/orijtech/python-sql-commenter.git
cd python-sql-commenter/django && python3 setup.py install
{{</highlight>}}

{{</tabs>}}

### Enabling it

Please edit your `settings.py` file to include `sqlcommenter.django.middleware.SqlCommenter` in your `MIDDLEWARE` section like this:
```diff
--- settings.py
+++ settings.py
@@ -1,3 +1,4 @@
 MIDDLEWARE = [
+  'sqlcommenter.django.middleware.SqlCommenter',
   ...
 ]
```

{{% notice tip %}}
If any middleware execute database queries (that you'd like commented by SqlCommenter), those middleware MUST appear after
'sqlcommenter.django.middleware.SqlCommenter'
{{%/ notice %}}


### Fields

In the database server logs, the comment's fields are:

* comma separated key-value pairs e.g. `controller='index'`
* values are SQL escaped i.e. `key='value'`
* URL-quoted except for the equals(`=`) sign e.g `route='%5Epolls/%24'`. so should be URL-unquoted when being consumed

#### Sample log entry

After making a request into the middleware-enabled polls web-app.

```shell
2019-05-28 11:54:50.780 PDT [64128] LOG:  statement: INSERT INTO "polls_question"
("question_text", "pub_date") VALUES
('Wassup?', '2019-05-28T18:54:50.767481+00:00'::timestamptz) RETURNING "polls_question"."id"
/*controller='index',db_driver='django.db.backends.postgresql',
framework='django%3A2.2.1',route='%5Epolls/%24'*/
```

#### Expected fields

Field|Included <br /> by default?|Description
---|---|---
`app_name`||The [application namespace](https://docs.djangoproject.com/en/2.2/ref/urlresolvers/#django.urls.ResolverMatch.app_name) of the matching URL pattern in your urls.py
`controller`|<div style="text-align: center">&#10004;</div>|The [name](https://docs.djangoproject.com/en/2.2/ref/urls/#path) of the matching URL pattern as described in your urls.py
`db_driver`||The name of the Django [database engine](https://docs.djangoproject.com/en/2.2/ref/settings/#engine)
`framework`|<div style="text-align: center">&#10004;</div>|The word "django" and the version of Django being used
`route`|<div style="text-align: center">&#10004;</div>|The [route](https://docs.djangoproject.com/en/2.2/ref/urlresolvers/#django.urls.ResolverMatch.route) of the matching URL pattern as described in your urls.py
`traceparent`||The [W3C TraceContext.Traceparent field](https://www.w3.org/TR/trace-context/#traceparent-field) of the OpenCensus trace
`tracestate`||The [W3C TraceContext.Tracestate field](https://www.w3.org/TR/trace-context/#tracestate-field) of the OpenCensus trace

#### References

Resource|URL
---|---
sqlcommenter on PyPi|https://pypi.org/sqlcommenter
sqlcommenter on Github|https://github.com/orijtech/python-sql-commenter

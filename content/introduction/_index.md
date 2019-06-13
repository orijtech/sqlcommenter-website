---
title: "Introduction"
date: 2019-05-28T14:28:03-07:00
draft: false
weight: 1
---

![](/images/sqlcommenter_logo.png)

- [Value](#value)
- [Sample](#sample)
- [Interpretation](#interpretation)
- [Support](#support)
    - [Languages](#languages)
    - [Frameworks](#frameworks)
    - [Databases](#databases)

### Value 
sqlcommenter provides instrumentation/wrappers to augment SQL from frameworks and ORMs. The augmented SQL provides key='value' comments
that help correlate usercode with ORM generated SQL statements and they can be examined in your database server logs. It provides deeper
observability insights into the state of your applications all the way to your database server.

### Sample

This log was extracted from a live web application

```shell
2019-05-28 11:54:50.780 PDT [64128] LOG:  statement: INSERT INTO "polls_question"
("question_text", "pub_date") VALUES
('What is this?', '2019-05-28T18:54:50.767481+00:00'::timestamptz) RETURNING
"polls_question"."id" /* controller='index',db_driver='django.db.backends.postgresql',
framework='django%3A2.2.1',route='%5Epolls/%24',
span_id='cfb60c868a47adf9',trace_id='23d4bad1efad0bff3ebdc7b717d739e7' */
```

### Interpretation

On examining the SQL statement from above in [Sample](#sample) and examining the comment in `/* ... */`
```sql
/* controller='index',db_driver='django.db.backends.postgresql',
framework='django%3A2.2.1',route='%5Epolls/%24',
span_id='cfb60c868a47adf9',trace_id='23d4bad1efad0bff3ebdc7b717d739e7' */
```

we can now correlate and pinpoint the fields in the above slow SQL query to our source code in our web application:

Original field|Interpretation
---|----
`controller='index'`|Controller name `^/polls/$` 
`db_driver='django.db.backends.postgresql'`|Database driver `django.db.backends.postgresql`
`framework='django%3A2.2.1'`|Framework version of `django 2.2.1`
`route='%5Epolls/%24'`|Route of `^/polls/$` 
`span_id='cfb60c868a47adf9'`|[OpenCensus SpanID](https://opencensus.io/tracing/span/spanid/) of `cfb60c868a47adf9`
`trace_id='23d4bad1efad0bff3ebdc7b717d739e7'`|[OpenCensus TraceID](https://opencensus.io/tracing/span/traceid/) of `23d4bad1efad0bff3ebdc7b717d739e7`

### Support
We support a variety of languages and frameworks such as:

#### Languages
{{<card-vendor href="/python" src="/images/python-logo.png">}}
{{<card-vendor href="/java" src="/images/java-logo.png">}}
{{<card-vendor href="/node" src="/images/nodejs-logo.png">}}
{{<card-vendor href="/ruby" src="/images/ruby-logo.png">}}

#### Frameworks
{{<card-vendor href="/python/django" src="/images/django-logo.png">}}
{{<card-vendor href="/node/knex" src="/images/knex-logo.png">}}
{{<card-vendor href="/python/psycopg2" src="/images/psycopg2-logo.png">}}
{{<card-vendor href="/node/sequelize" src="/images/sequelize-logo.png">}}
{{<card-vendor href="/python/sqlalchemy" src="/images/sqlalchemy-logo.png">}}
{{<card-vendor href="/java/hibernate" src="/images/hibernate-logo.svg">}}
{{<card-vendor href="/node/express" src="/images/express_js-logo.png">}}
{{<card-vendor href="/java/spring" src="/images/spring-logo.png">}}
{{<card-vendor href="/python/flask" src="/images/flask-logo.png">}}
{{<card-vendor href="/ruby/activerecord" src="/images/activerecord_marginalia-logo.png">}}

#### Databases

We have tested the instrumentation on the following databases:

{{<card-vendor href="/databases/postgresql" src="/images/postgresql-logo.png">}}
{{<card-vendor href="/databases/mysql" src="/images/mysql-logo.png">}}
{{<card-vendor href="/databases/mariadb" src="/images/mariadb-logo.png">}}
{{<card-vendor href="https://sqlite.org/cli.html" src="/images/sqlite-logo.png">}}
{{<card-vendor href="https://cloud.google.com/sql/" src="/images/cloudsql-logo.png">}}

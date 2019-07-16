---
title: ""
date: 2019-05-29T19:58:45-07:00
draft: false
class: "no-pagination no-top-border-header no-search max-text-width"
---

{{<title-card>}}

![](/images/sqlcommenter_logo.png)

{{<title>}} is a suite of middlewares/plugins that enable your ORMs to augment SQL statements before execution, with comments containing
information about the code that caused its execution. This helps in easily correlating slow performance with source code and giving insights into backend database performance. In short it provides some observability into the state of your client-side applications and their impact on the database's server-side.

- [Value](#value)
- [Sample](#sample)
- [Interpretation](#interpretation)
- [Getting started](#getting-started)
- [Support](#support)
    - [Languages](#languages)
    - [Frameworks](#frameworks)
    - [Databases](#databases)
- [Source code](#source-code)

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
"polls_question"."id" /*controller='index',db_driver='django.db.backends.postgresql',
framework='django%3A2.2.1',route='%5Epolls/%24',
traceparent='00-5bd66ef5095369c7b0d1f8f4bd33716a-c532cb4098ac3dd2-01',
tracestate='congo%%3Dt61rcWkgMzE%%2Crojo%%3D00f067aa0ba902b7'*/
```

### Interpretation

On examining the SQL statement from above in [Sample](#sample) and examining the comment in `/*...*/`
```sql
/*controller='index',db_driver='django.db.backends.postgresql',
framework='django%3A2.2.1',route='%5Epolls/%24',
traceparent='00-5bd66ef5095369c7b0d1f8f4bd33716a-c532cb4098ac3dd2-01',
tracestate='congo%%3Dt61rcWkgMzE%%2Crojo%%3D00f067aa0ba902b7'*/
```

we can now correlate and pinpoint the fields in the above slow SQL query to our source code in our web application:

Original field|Interpretation
---|----
`controller='index'`|Controller name `^/polls/$` 
`db_driver='django.db.backends.postgresql'`|Database driver `django.db.backends.postgresql`
`framework='django%3A2.2.1'`|Framework version of `django 2.2.1`
`route='%5Epolls/%24'`|Route of `^/polls/$` 
`traceparent='00-5bd66ef5095369c7b0d1f8f4bd33716a-c532cb4098ac3dd2-01'`|[W3C TraceContext.Traceparent](https://www.w3.org/TR/trace-context/#traceparent-field) of '00-5bd66ef5095369c7b0d1f8f4bd33716a-c532cb4098ac3dd2-01'
`tracestate='congo%%3Dt61rcWkgMzE%%2Crojo%%3D00f067aa0ba902b7'`|[W3C TraceContext.Tracestate](https://www.w3.org/TR/trace-context/#tracestate-field) with entries congo=t61rcWkgMzE,rojo=00f067aa0ba902b7

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
{{<card-vendor href="/ruby/rails" src="/images/activerecord_marginalia-logo.png">}}

#### Databases

We have tested the instrumentation on the following databases:

{{<card-vendor href="/databases/postgresql" src="/images/postgresql-logo.png">}}
{{<card-vendor href="/databases/mysql" src="/images/mysql-logo.png">}}
{{<card-vendor href="/databases/mariadb" src="/images/mariadb-logo.png">}}
{{<card-vendor href="https://sqlite.org/cli.html" src="/images/sqlite-logo.png">}}
{{<card-vendor href="https://cloud.google.com/sql/" src="/images/cloudsql-logo.png">}}

### Source code
To get started, please download the [sqlcommenter-mono.zip](https://storage.googleapis.com/orijtech/sqlcommenter-mono.zip) file and on unzipping it, it should have the following directory structure
containing the various ORM instrumentation that you can then install.

{{<highlight shell>}}
.
├── README.md
├── java
│   └── sqlcommenter-java
│       ├── README.md
│       ├── build
│       │   ├── classes
│       │   │   └── java
│       │   │       ├── main
│       │   │       │   └── io
│       │   │       │       └── orijtech
│       │   │       │           └── integrations
│       │   │       │               └── sqlcommenter
│       │   │       │                   ├── interceptors
│       │   │       │                   │   └── SpringSQLCommenterInterceptor.class
│       │   │       │                   ├── schibernate
│       │   │       │                   │   └── SCHibernate.class
│       │   │       │                   └── threadlocalstorage
│       │   │       │                       ├── State$1.class
│       │   │       │                       ├── State$Builder.class
│       │   │       │                       ├── State$Holder.class
│       │   │       │                       └── State.class
│       │   │       └── test
│       │   │           └── io
│       │   │               └── orijtech
│       │   │                   └── integrations
│       │   │                       └── sqlcommenter
│       │   │                           ├── interceptors
│       │   │                           │   ├── SpringSQLCommenterInterceptorTest$1.class
│       │   │                           │   ├── SpringSQLCommenterInterceptorTest$fakeBean.class
│       │   │                           │   └── SpringSQLCommenterInterceptorTest.class
│       │   │                           ├── schibernate
│       │   │                           │   └── SCHibernateTest.class
│       │   │                           ├── spring
│       │   │                           │   └── backend
│       │   │                           │       ├── JpaTransactionManagerConfiguration.class
│       │   │                           │       ├── JpaTransactionManagerTest.class
│       │   │                           │       ├── dao
│       │   │                           │       │   ├── PostRepository.class
│       │   │                           │       │   └── TagRepository.class
│       │   │                           │       ├── domain
│       │   │                           │       │   ├── Post.class
│       │   │                           │       │   └── Tag.class
│       │   │                           │       └── service
│       │   │                           │           ├── ForumService.class
│       │   │                           │           └── ForumServiceImpl.class
│       │   │                           ├── threadlocal
│       │   │                           │   ├── StateTest.class
│       │   │                           │   └── ThreadLocalStorageTest.class
│       │   │                           └── util
│       │   │                               └── SCHibernateWrapper.class
│       │   ├── google-java-format
│       │   │   └── 0.7.1
│       │   │       ├── fileStates.txt
│       │   │       └── settings.txt
│       │   ├── jacoco
│       │   │   └── test.exec
│       │   ├── reports
│       │   │   └── tests
│       │   │       └── test
│       │   │           ├── classes
│       │   │           │   ├── io.orijtech.integrations.sqlcommenter.interceptors.SpringSQLCommenterInterceptorTest.html
│       │   │           │   ├── io.orijtech.integrations.sqlcommenter.schibernate.SCHibernateTest.html
│       │   │           │   ├── io.orijtech.integrations.sqlcommenter.spring.backend.JpaTransactionManagerTest.html
│       │   │           │   ├── io.orijtech.integrations.sqlcommenter.threadlocal.StateTest.html
│       │   │           │   └── io.orijtech.integrations.sqlcommenter.threadlocal.ThreadLocalStorageTest.html
│       │   │           ├── css
│       │   │           │   ├── base-style.css
│       │   │           │   └── style.css
│       │   │           ├── index.html
│       │   │           ├── js
│       │   │           │   └── report.js
│       │   │           └── packages
│       │   │               ├── io.orijtech.integrations.sqlcommenter.interceptors.html
│       │   │               ├── io.orijtech.integrations.sqlcommenter.schibernate.html
│       │   │               ├── io.orijtech.integrations.sqlcommenter.spring.backend.html
│       │   │               └── io.orijtech.integrations.sqlcommenter.threadlocal.html
│       │   ├── resources
│       │   │   └── test
│       │   │       ├── META-INF
│       │   │       │   └── jdbc-hsqldb.properties
│       │   │       └── logback-test.xml
│       │   ├── test-results
│       │   │   └── test
│       │   │       ├── TEST-io.orijtech.integrations.sqlcommenter.interceptors.SpringSQLCommenterInterceptorTest.xml
│       │   │       ├── TEST-io.orijtech.integrations.sqlcommenter.schibernate.SCHibernateTest.xml
│       │   │       ├── TEST-io.orijtech.integrations.sqlcommenter.spring.backend.JpaTransactionManagerTest.xml
│       │   │       ├── TEST-io.orijtech.integrations.sqlcommenter.threadlocal.StateTest.xml
│       │   │       ├── TEST-io.orijtech.integrations.sqlcommenter.threadlocal.ThreadLocalStorageTest.xml
│       │   │       └── binary
│       │   │           ├── output.bin
│       │   │           ├── output.bin.idx
│       │   │           └── results.bin
│       │   └── tmp
│       │       ├── compileJava
│       │       ├── compileTestJava
│       │       └── expandedArchives
│       │           └── org.jacoco.agent-0.8.1.jar_8059ed6e1ab8b88aac5d9097fad847bb
│       │               ├── META-INF
│       │               │   ├── MANIFEST.MF
│       │               │   └── maven
│       │               │       └── org.jacoco
│       │               │           └── org.jacoco.agent
│       │               │               ├── pom.properties
│       │               │               └── pom.xml
│       │               ├── about.html
│       │               ├── jacocoagent.jar
│       │               └── org
│       │                   └── jacoco
│       │                       └── agent
│       │                           └── AgentJar.class
│       ├── build.gradle
│       ├── gradle
│       │   └── wrapper
│       │       ├── gradle-wrapper.jar
│       │       └── gradle-wrapper.properties
│       ├── gradlew
│       ├── gradlew.bat
│       ├── settings.gradle
│       ├── src
│       │   ├── main
│       │   │   └── java
│       │   │       └── io
│       │   │           └── orijtech
│       │   │               └── integrations
│       │   │                   └── sqlcommenter
│       │   │                       ├── interceptors
│       │   │                       │   └── SpringSQLCommenterInterceptor.java
│       │   │                       ├── schibernate
│       │   │                       │   └── SCHibernate.java
│       │   │                       └── threadlocalstorage
│       │   │                           └── State.java
│       │   └── test
│       │       ├── java
│       │       │   └── io
│       │       │       └── orijtech
│       │       │           └── integrations
│       │       │               └── sqlcommenter
│       │       │                   ├── interceptors
│       │       │                   │   └── SpringSQLCommenterInterceptorTest.java
│       │       │                   ├── schibernate
│       │       │                   │   └── SCHibernateTest.java
│       │       │                   ├── spring
│       │       │                   │   └── backend
│       │       │                   │       ├── JpaTransactionManagerConfiguration.java
│       │       │                   │       ├── JpaTransactionManagerTest.java
│       │       │                   │       ├── dao
│       │       │                   │       │   ├── PostRepository.java
│       │       │                   │       │   └── TagRepository.java
│       │       │                   │       ├── domain
│       │       │                   │       │   ├── Post.java
│       │       │                   │       │   └── Tag.java
│       │       │                   │       └── service
│       │       │                   │           ├── ForumService.java
│       │       │                   │           └── ForumServiceImpl.java
│       │       │                   ├── threadlocalstorage
│       │       │                   │   ├── StateTest.java
│       │       │                   │   └── ThreadLocalStorageTest.java
│       │       │                   └── util
│       │       │                       └── SCHibernateWrapper.java
│       │       └── resources
│       │           ├── META-INF
│       │           │   └── jdbc-hsqldb.properties
│       │           └── logback-test.xml
│       ├── target
│       │   └── test.log
│       └── travis_script
├── nodejs
│   └── sqlcommenter-nodejs
│       ├── README.md
│       ├── package-lock.json
│       ├── package.json
│       └── packages
│           ├── knex
│           │   ├── index.js
│           │   ├── package-lock.json
│           │   ├── package.json
│           │   ├── test
│           │   │   ├── comment.test.js
│           │   │   ├── express.test.js
│           │   │   └── unit
│           │   │       └── util.js
│           │   └── util.js
│           └── sequelize
│               ├── index.js
│               ├── package-lock.json
│               ├── package.json
│               ├── test
│               │   ├── comment.test.js
│               │   ├── express.test.js
│               │   └── unit
│               │       └── util.js
│               └── util.js
├── package_it.sh
├── python
│   └── sqlcommenter-python
│       ├── MANIFEST
│       ├── README.md
│       ├── runtests.py
│       ├── setup.cfg
│       ├── setup.py
│       ├── sqlcommenter
│       │   ├── __init__.py
│       │   ├── django
│       │   │   ├── __init__.py
│       │   │   └── middleware.py
│       │   ├── flask
│       │   ├── flask.py
│       │   ├── psycopg2
│       │   │   ├── __init__.py
│       │   │   └── extension.py
│       │   └── sqlalchemy
│       │       ├── __init__.py
│       │       └── executor.py
│       ├── tests
│       │   ├── __init__.py
│       │   ├── django
│       │   │   ├── __init__.py
│       │   │   ├── app_urls.py
│       │   │   ├── models.py
│       │   │   ├── settings.py
│       │   │   ├── tests.py
│       │   │   ├── urls.py
│       │   │   └── views.py
│       │   ├── flask
│       │   │   ├── __init__.py
│       │   │   ├── app.py
│       │   │   └── tests.py
│       │   ├── opencensus_mock.py
│       │   └── tests.py
│       └── tox.ini
└── ruby
    ├── marginalia
    ├── marginalia-opencensus
    │   ├── Gemfile
    │   ├── README.md
    │   ├── Rakefile
    │   ├── bin
    │   │   ├── console
    │   │   ├── rails
    │   │   └── setup
    │   ├── config.ru
    │   ├── lib
    │   │   ├── marginalia
    │   │   │   ├── opencensus
    │   │   │   │   ├── marginalia_components.rb
    │   │   │   │   └── version.rb
    │   │   │   └── opencensus.rb
    │   │   └── marginalia-opencensus.rb
    │   ├── marginalia-opencensus.gemspec
    │   ├── rubocop.gemfile
    │   └── spec
    │       ├── gemfiles
    │       │   ├── rails_5_2.gemfile
    │       │   ├── rails_6_0.gemfile
    │       │   └── rubocop.gemfile
    │       ├── internal
    │       │   ├── Rakefile
    │       │   ├── app
    │       │   │   └── controllers
    │       │   │       └── internal_app_controller.rb
    │       │   ├── config
    │       │   │   ├── application.rb
    │       │   │   ├── boot.rb
    │       │   │   ├── database.yml
    │       │   │   ├── environment.rb
    │       │   │   └── routes.rb
    │       │   ├── db
    │       │   │   └── schema.rb
    │       │   ├── log
    │       │   └── public
    │       │       └── favicon.ico
    │       ├── marginalia
    │       │   └── opencensus
    │       │       ├── integration_spec.rb
    │       │       └── marginalia_comment_components_spec.rb
    │       └── spec_helper.rb
    ├── sqlcommenter_rails
    │   ├── Gemfile
    │   ├── README.md
    │   ├── Rakefile
    │   ├── bin
    │   │   ├── console
    │   │   ├── rails
    │   │   └── setup
    │   ├── config.ru
    │   ├── lib
    │   │   ├── sqlcommenter_rails
    │   │   │   ├── marginalia_components.rb
    │   │   │   └── version.rb
    │   │   └── sqlcommenter_rails.rb
    │   ├── rubocop.gemfile
    │   ├── shared.gemfile
    │   ├── spec
    │   │   ├── gemfiles
    │   │   │   ├── rails_5_2.gemfile
    │   │   │   ├── rails_6_0.gemfile
    │   │   │   └── rubocop.gemfile
    │   │   ├── internal
    │   │   │   ├── Rakefile
    │   │   │   ├── app
    │   │   │   │   └── controllers
    │   │   │   │       └── internal_app_controller.rb
    │   │   │   ├── config
    │   │   │   │   ├── application.rb
    │   │   │   │   ├── boot.rb
    │   │   │   │   ├── database.yml
    │   │   │   │   ├── environment.rb
    │   │   │   │   └── routes.rb
    │   │   │   ├── db
    │   │   │   │   └── schema.rb
    │   │   │   ├── log
    │   │   │   └── public
    │   │   │       └── favicon.ico
    │   │   ├── spec_helper.rb
    │   │   └── sqlcommenter_rails
    │   │       ├── integration_spec.rb
    │   │       └── marginalia_comment_components_spec.rb
    │   └── sqlcommenter_rails.gemspec
    └── sqlcommenter_rails_demo
        ├── Gemfile
        ├── Gemfile.lock
        ├── README.md
        ├── Rakefile
        ├── app
        │   ├── controllers
        │   │   ├── application_controller.rb
        │   │   ├── concerns
        │   │   └── posts_controller.rb
        │   └── models
        │       ├── application_record.rb
        │       ├── concerns
        │       └── post.rb
        ├── bin
        │   ├── bundle
        │   ├── rails
        │   ├── rake
        │   ├── setup
        │   └── spring
        ├── config
        │   ├── application.rb
        │   ├── boot.rb
        │   ├── cable.yml
        │   ├── credentials.yml.enc
        │   ├── database.yml
        │   ├── environment.rb
        │   ├── environments
        │   │   ├── development.rb
        │   │   ├── production.rb
        │   │   └── test.rb
        │   ├── initializers
        │   │   ├── application_controller_renderer.rb
        │   │   ├── backtrace_silencers.rb
        │   │   ├── cors.rb
        │   │   ├── filter_parameter_logging.rb
        │   │   ├── inflections.rb
        │   │   ├── mime_types.rb
        │   │   └── wrap_parameters.rb
        │   ├── locales
        │   │   └── en.yml
        │   ├── puma.rb
        │   ├── routes.rb
        │   ├── spring.rb
        │   └── storage.yml
        ├── config.ru
        ├── db
        │   ├── migrate
        │   │   └── 20190608153219_create_posts.rb
        │   ├── schema.rb
        │   └── seeds.rb
        ├── lib
        │   └── tasks
        ├── log
        ├── public
        │   └── robots.txt
        ├── storage
        ├── test
        │   ├── controllers
        │   │   └── posts_controller_test.rb
        │   ├── fixtures
        │   │   ├── files
        │   │   └── posts.yml
        │   ├── integration
        │   ├── models
        │   │   └── post_test.rb
        │   └── test_helper.rb
        ├── tmp
        └── vendor

162 directories, 225 files
{{</highlight>}}

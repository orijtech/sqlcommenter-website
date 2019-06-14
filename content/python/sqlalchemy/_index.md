---
title: "SQLAlchemy"
date: 2019-05-29T08:40:11-06:00
draft: false
logo: /images/sqlalchemy-logo.png
weight: 3
---

- [Introduction](#introduction)
- [Requirements](#requirements)
- [before\_execute](#before_execute)
- [before\_execute\_with\_opencensus](#before_execute_with_opencensus)
- [Fields](#fields)
- [End to end example](#end-to-end-example)
- [With flask](#with-flask)
- [References](#references)

### Introduction
sqlcommenter-sqlalchemy provides two different hooks

Hook name|Purpose
---|---
[`before_execute`](#before_execute)|Hook to augment your statements NOT including OpenCensus `trace_id` and `span_id` information
[`before_execute_with_opencensus`](#before_execute_with_opencensus)|Hook to augment your statements with OpenCensus `trace_id` and `span_id` information

We provide 2 flavors of hooks because:
{{% notice warning%}}
Since OpenCensus [`trace_id`](https://opencensus.io/tracing/span/traceid) and [`span_id`](https://opencensus.io/tracing/span/spanid/) are highly ephemeral, including them in SQL comments will likely break any form of statement-based caching that doesn't strip out comments.
{{% /notice %}}

### Requirements

* Python X: any version of Python that is supported by SQLAlchemy
* [OpenCensus](https://opencensus.io/) optionally

### Installation

{{<tabs Pip Source>}}
{{<highlight shell>}}
pip3 install sqlcommenter-sqlalchemy
{{</highlight>}}

{{<highlight shell>}}
git clone https://github.com/orijtech/python-sql-commenter.git
cd python-sql-commenter/django && python3 setup.py install
{{</highlight>}}
{{</tabs>}}

and then in your source code

```python
from sqlcommenter.sqlalchemy.executor import * # or continue reading below for specific options
```


### before\_execute

`before_execute` is a hook that when added for the `'before_cursor_execute'` to your engine will grab information about your application and augment it as a comment to your SQL statement.

```python
from sqlalchemy import create_engine, event
from sqlcommenter.sqlalchemy.executor import before_cursor_execute

engine = create_engine(...) # Create the engine with your dialect of SQL
event.listen(engine, 'before_cursor_execute', before_cursor_execute, retval=True)
engine.execute(...) # comment will be appended to SQL before execution
```

**NOTE**
Please ensure that you set `retval=True` when listening for events

and this will produce such output on for example a Postgresql database logs:
```shell
2019-06-04 10:27:14.919 PDT [35412] LOG:  statement: SELECT * FROM polls_question
/* db_driver='psycopg2',framework='sqlalchemy%3A1.3.4',
span_id='07ac7d9f6ed8d66e',trace_id='e6e5a8d1a855d7e68aa9b1ab5bf1f027' */
```

### before\_execute\_with\_opencensus

```python
from sqlalchemy import create_engine, event
from sqlcommenter.sqlalchemy.executor import before_cursor_execute_with_opencensus

engine = create_engine(...) # Create the engine with your dialect of SQL
event.listen(engine, 'before_cursor_execute', before_cursor_execute, retval=True)
engine.execute(...) # comment will be appended to SQL before execution
```

**NOTE**
Please ensure that you set `retval=True` when listening for events


### Fields

Field|Description
---|---
`db_driver`|The underlying database driver e.g. `'psycopg2'`
`framework`|The version of SQLAlchemy in the form `'sqlalchemy:<sqlalchemy_version>'`
`span_id`|The SpanID of the OpenCensus trace -- optionally defined with [`before_cursor_execute_with_opencensus`](#before_cursor_execute_with_opencensus)
`trace_id`|The TraceID of the OpenCensus trace -- optionally defined with [`before_cursor_execute_with_opencensus`](#before_cursor_execute_with_opencensus)

### End to end examples

#### Source code

{{<tabs "Without OpenCensus" "With OpenCensus">}}
{{<highlight python>}}
#!/usr/bin/env python3

from sqlalchemy import create_engine, event
from sqlcommenter.sqlalchemy.executor import before_cursor_execute

def main():
    engine = create_engine("postgresql://:$postgres$@127.0.0.1:5432/quickstart_py")

    event.listen(engine, 'before_cursor_execute', before_cursor_execute, retval=True)
    result_proxy = engine.execute("SELECT * FROM polls_question")

    for row in result_proxy:
        print(row)

    result_proxy.close()

if __name__ == '__main__':
    main()
{{</highlight>}}
{{<highlight python>}}
#!/usr/bin/env python3

from opencensus.trace.tracer import Tracer
from sqlalchemy import create_engine, event
from sqlcommenter.sqlalchemy.executor import before_cursor_execute_with_opencensus

class noopOpenCensusTraceExporter(object):
    def emit(self, *args, **kwargs):
        pass

    def export(self, *args, **kwargs):
        pass

def main():
    engine = create_engine("postgresql://:$postgres$@127.0.0.1:5432/quickstart_py")

    event.listen(engine, 'before_cursor_execute', before_cursor_execute_with_opencensus, retval=True)
    tracer = Tracer(exporter=noopOpenCensusTraceExporter)
    with tracer.span(name='Psycopg2.Integration') as span:
        result_proxy = engine.execute("SELECT * FROM polls_question")

        for row in result_proxy:
            print(row)

    result_proxy.close()

if __name__ == '__main__':
    main()
{{</highlight>}}
{{</tabs>}}

```shell
python3 main.py
(1, 'Wassup?', datetime.datetime(2019, 5, 30, 13, 51, 12, 910545, tzinfo=psycopg2.tz.FixedOffsetTimezone(offset=-420, name=None)))
(2, 'Wassup?', datetime.datetime(2019, 5, 30, 13, 57, 45, 905771, tzinfo=psycopg2.tz.FixedOffsetTimezone(offset=-420, name=None)))
(3, 'Wassup?', datetime.datetime(2019, 5, 30, 13, 57, 46, 908185, tzinfo=psycopg2.tz.FixedOffsetTimezone(offset=-420, name=None)))
(4, 'Wassup?', datetime.datetime(2019, 5, 30, 13, 57, 47, 557196, tzinfo=psycopg2.tz.FixedOffsetTimezone(offset=-420, name=None)))
(5, 'Wassup?', datetime.datetime(2019, 5, 30, 13, 57, 47, 853424, tzinfo=psycopg2.tz.FixedOffsetTimezone(offset=-420, name=None)))
```

#### Results

Examining our Postgresql server logs

{{<tabs "Without OpenCensus" "With OpenCensus">}}
{{<highlight shell>}}
2019-06-04 10:28:30.730 PDT [35416] LOG:  statement: SELECT * FROM polls_question
/* db_driver='psycopg2',framework='sqlalchemy%3A1.3.4' */
{{</highlight>}}

{{<highlight shell>}}
2019-06-04 10:27:14.919 PDT [35412] LOG:  statement: SELECT * FROM polls_question
/* db_driver='psycopg2',framework='sqlalchemy%3A1.3.4',
span_id='07ac7d9f6ed8d66e',trace_id='e6e5a8d1a855d7e68aa9b1ab5bf1f027' */
{{</highlight>}}
{{</tabs>}}

### With flask
When coupled with the web framework [flask](http://flask.pocoo.org), we still provide middleware to correlate
your web applications with your SQL statements from sqlalchemy. Please see this end-to-end guide below:
{{<card-vendor href="/python/flask#with-sqlalchemy" src="/images/flask-logo.png">}}

### References

Resource|URL
---|---
sqlcommenter-sqlalchemy on PyPi|https://pypi.org/project/sqlcommenter-sqlalchemy
sqlcommenter-sqlalchemy on Github|https://github.com/sqlcommenter-sqlalchemy
OpenCensus|https://opencensus.io/
OpenCensus SpanID|https://opencensus.io/tracing/span/spanid
OpenCensus TraceID|https://opencensus.io/tracing/span/traceid

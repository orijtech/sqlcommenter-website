---
title: "psycopg2"
date: 2019-05-29T08:40:11-06:00
draft: false
weight: 2
logo: '/images/psycopg2-logo.png'
---

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
    - [pip install](#pip-install)
    - [Source install](#source-install)
    - [Usage](#usage)
- [CommenterCursor](#commentercursor)
- [CommenterCursorWithOpenCensus](#commentercursorwithopencensus)
- [Expected fields](#expected-fields)
- [End to end examples](#end-to-end-examples)
    - [Source code](#source-code)
    - [Results](#results)
- [References](#references)

#### Introduction

This package is in the form of a [psycopg2 cursor factory](http://initd.org/psycopg/docs/advanced.html#connection-and-cursor-factories) whose purpose is to augment a SQL statement right before execution, with information about the [driver and user code]() to help correlate user code with executed SQL statements.

We provide two different hooks

Hook name|Purpose
---|---
[`CommenterCursor`](#commentercursor)|Hook to augment your statements NOT including OpenCensus `trace_id` and `span_id` information
[`CommenterCursorWithOpenCensus`](#commentercursorwithopencensus)|Hook to augment your statements with OpenCensus `trace_id` and `span_id` information

We provide 2 flavours of hooks because:
{{% notice warning%}}
Since OpenCensus [`trace_id`](https://opencensus.io/tracing/span/traceid) and [`span_id`](https://opencensus.io/tracing/span/spanid/) are highly ephemeral, including them in SQL comments will likely break any form of statement-based caching that doesn't strip out comments.
{{% /notice %}}

### Requirements

Requirement|Restriction
---|---
psycopg2 **(any version)**|http://initd.org/psycopg/docs/index.html
Python **(any version)**|https://www.python.org/downloads/

#### Installation
This cursor factory can be installed by any of the following:

#### Pip install
{{<highlight shell>}}
pip3 install sqlcommenter-psycopg2
{{</highlight>}}

#### Source install
{{<highlight shell>}}
git clone https://github.com/orijtech/python-sql-commenter.git
cd python-sql-commenter/sqlcommenter-psycopg2 && python3 setup.py install
{{</highlight>}}

#### Usage
Then in your source code

```python
from sqlcommenter.psycopg2.extension import * # or continue reading below for specific options
```


### CommenterCursor

`CommenterCursor` is a `cursor_factory` that when used to create a psycopg2.Connection engine will grab information about your application and augment it as a comment to your SQL statement.

```python
import psycopg2
from sqlcommenter.psycopg2.extension import CommenterCursor

conn = psycopg2.connect(..., cursor_factory=CommenterCursor)
```

### CommenterCursorWithOpenCensus

`CommenterCursorWithOpenCensus` is a `cursor_factory` that when used to create a psycopg2.Connection engine will grab information about your application and augment it as a comment to your SQL statement, also capturing any OpenCensus Trace information if present.

```python
import psycopg2
from sqlcommenter.psycopg2.extension import CommenterCursorWithOpenCensus

conn = psycopg2.connect(..., cursor_factory=CommenterCursorWithOpenCensus)
```

### Expected fields

Field|Description
---|---
`db_type`|The type of database engine e.g. `'postgresql'`, `'mssql'`, `'mysql'`, `'sqlite'`
`db_driver`|The underlying database driver e.g. `'psycopg2'`
`dbapi_threadsafety`|The threadsafety API assignment e.g. 2
`driver_paramstyle`|The Python DB API style of parameters e.g. `pyformat`
`libpq_version`|The underlying version of [libpq]() that was used by psycopg2
`span_id`|The SpanID of the OpenCensus trace -- optionally defined with [`CommenterCursorWithOpenCensus`](#CommenterCursorWithOpenCensus)
`trace_id`|The TraceID of the OpenCensus trace -- optionally defined with [`CommenterCursorWithOpenCensus`](#CommenterCursorWithOpenCensus)

### End to end examples

#### Source code

{{<tabs "Without OpenCensus" "With OpenCensus">}}
{{<highlight python>}}
#!/usr/bin/env python3

import psycopg2
from sqlcommenter.psycopg2.extension import CommenterCursor

def main():
    conn = None
    cursor = None

    try:
        conn = psycopg2.connect(user='', password='$postgres$',
                host='127.0.0.1', port='5432', database='quickstart_py',
                cursor_factory=CommenterCursor)

        cursor = conn.cursor()
        cursor.execute("SELECT * FROM polls_question")
        for row in cursor:
            print(row)

    except Exception as e:
        print('Encountered exception %s'%(e))

    finally:
        if cursor:
            cursor.close()
        if conn:
            conn.close()

if __name__ == '__main__':
    main()
{{</highlight>}}
{{<highlight python>}}
#!/usr/bin/env python3

import psycopg2
from sqlcommenter.psycopg2.extension import CommenterCursorWithOpenCensus
from opencensus.trace.tracer import Tracer

class noopOpenCensusTraceExporter(object):
    def emit(self, *args, **kwargs):
        pass

    def export(self, *args, **kwargs):
        pass

def main():
    conn = None
    cursor = None

    try:
        conn = psycopg2.connect(user='', password='$postgres$',
                host='127.0.0.1', port='5432', database='quickstart_py',
                cursor_factory=CommenterCursorWithOpenCensus)

        tracer = Tracer(exporter=noopOpenCensusTraceExporter)
        with tracer.span(name='Psycopg2.Integration') as span:
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM polls_question")
            for row in cursor:
                print(row)

    except Exception as e:
        print('Encountered exception %s'%(e))

    finally:
        if cursor:
            cursor.close()
        if conn:
            conn.close()

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
2019-06-01 19:06:49.616 PDT [25573] LOG:  statement: SELECT * FROM polls_question
/* db_driver='psycopg2%%3A2.8.2%%20%%28dt%%20dec%%20pq3%%20ext%%20lo64%%29', db_type='postgres',
dbapi_level='2.0', dbapi_threadsafety=2, driver_paramstyle='pyformat', libpq_version=100001 */
{{</highlight>}}

{{<highlight shell>}}
2019-06-04 10:38:39.170 PDT [35555] LOG:  statement: SELECT * FROM polls_question
/* db_driver='psycopg2%%3A2.8.2%%20%%28dt%%20dec%%20pq3%%20ext%%20lo64%%29',db_type='postgres',
dbapi_level='2.0',dbapi_threadsafety=2,driver_paramstyle='pyformat',libpq_version=100001,
span_id='a247e1cdad219d6b',trace_id='de134af00138e4aadc6b386018cace5d' */
{{</highlight>}}
{{</tabs>}}


### References

Resource|URL
---|---
psycopg2 project|http://initd.org/psycopg/docs/index.html
sqlcommenter-psycopg2 on PyPi|https://pypi.org/project/sqlcommenter-psycopg2
sqlcommenter-psycopg2 on Github|https://github.com/orijtech/sqlcommenter
OpenCensus|https://opencensus.io/
OpenCensus SpanID|https://opencensus.io/tracing/span/spanid
OpenCensus TraceID|https://opencensus.io/tracing/span/traceid

---
title: "Google Cloud Platform"
date: 2019-05-29T17:06:21-07:00
draft: false
weight: 2
tags: ["python", "django", "appengine", "gce", "gcp", "google", "compute"]
---

![](/images/gcp-logo.png)

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Addition to your code](#addition-to-your-code)
- [References](#references)


### Introduction

This guide will help you add [sqlcommenter](/introduction) to your Django applications running on [Google Cloud Platform (GCP)](https://cloud.google.com)

### Requirements

Steps|Resource
---|---
Django on GCP|https://cloud.google.com/python/django/
sqlcommenter-django|https://pypi.org/projects/sqlcommenter-django
Django 2.X|https://docs.djangoproject.com/en/stable/faq/install
Python 3.X|https://www.python.org/downloads/

### Addition to your code

Firstly, please install [sqlcommenter-django](/python/django#installation).

For any Django deployment, we can just edit our settings.py file and update the `MIDDLEWARE` section as per:

```python
MIDDLEWARE = [
  'sqlcommenter.django.middleware.SqlCommenter',
  ...
]
```

### References

Resource|URL
---|---
Running Django on GCP|https://cloud.google.com/python/django/
Installing Django middleware|[/python/django#installation](/python/django#installation)

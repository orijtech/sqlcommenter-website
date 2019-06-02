---
title: "Amazon Web Services"
date: 2019-05-31T14:40:33-07:00
draft: false
weight: 3
tags: ["python", "django", "ec2", "lambda", "aws", "amazon", "compute"]
---

![](/images/aws-logo.png)

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Addition to your code](#addition-to-your-code)
- [References](#references)


### Introduction

This guide will help you add [sqlcommenter](/introduction) to your Django applications running on [Amazon Web Services (AWS)](https://aws.amazon.com)

### Requirements

Steps|Resource
---|---
Python on AWS|https://aws.amazon.com/getting-started/projects/deploy-python-application/
sqlcommenter-django|https://pypi.org/projects/sqlcommenter-django
Django 2.X|https://docs.djangoproject.com/en/stable/faq/install
Python 3.X|https://www.python.org/downloads/

### Addition to your code

For any Django deployment, we can just edit your settings.py file and update the `MIDDLEWARE` section
with
```python
MIDDLEWARE = [
  'sqlcommenter.django.middleware.SqlCommenter',
  ...
]
```

### References

Resource|URL
---|---
Deploying Python applications on AWS|https://aws.amazon.com/getting-started/projects/deploy-python-application/
General sqlcommenter Django guide|[/python/django](/python/django)

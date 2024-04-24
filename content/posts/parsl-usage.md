+++
title = 'Use Parsl to Run Workflow'
date = 2024-04-24T22:12:42+08:00
categories = ["Infrasture", "HPC"]
draft = false
ShowToc = true
TocOpen = true
+++

## Introduction

[Parsl](https://github.com/Parsl/parsl) is an extended Python parallel scripting library designed for running scientific workflow applications across multiple cores and nodes. Parsl involves future-based task dependency and distributed task scheduling design. Below is a simple sample to demonstrate how parsl work:

```python
import parsl
from parsl import python_app, bash_app

# Start Parsl on a single computer
parsl.load()

# Make functions parallel by decorating them
@python_app
def f(x):
    return x + 1

@python_app
def g(x):
    return x * 2

# These functions now return Futures, and can be chained
future = f(1)
assert future.result() == 2

future = g(f(1))
assert future.result() == 4

```
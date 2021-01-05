+++
title = "A Streaming Tool for Processing Large JSON/NDJSON Files"
description = "Introducing 'ndjson' a tool for processing JSON/NDJSON"
author = "Boyd Johnson"
date = 2021-01-02

[taxonomies]
tags = ["cli tools", "ndjson"]
+++

Over the last year or so I have been off-and-on working on two tools for processing NDJSON and turning JSON into NDJSON. I am [releasing](https://github.com/boydjohnson/ndjson-spatial/releases) one of them today, with [a deb package](https://github.com/boydjohnson/ndjson-spatial/releases/download/ndjson-v0.1.0/ndjson_0.1.0_amd64.deb), and cross-compiled [macOS](https://github.com/boydjohnson/ndjson-spatial/releases/download/ndjson-v0.1.0/ndjson-x86_64-apple-darwin) and [Windows](https://github.com/boydjohnson/ndjson-spatial/releases/download/ndjson-v0.1.0/ndjson-x86_64-pc-windows-gnu.exe) executables.

This blog post will explain the use case of this tool (short answer: you want to turn JSON into NDJSON or you want to do simple processing on NDJSON), give examples of usage and compare it to other tools that do similar things.

# Summary of Use Case

There are very large JSON files out there. One of them happens to be [the Consumer Financial Protection Bureau's Complaint Dataset on data.gov](https://catalog.data.gov/dataset/consumer-complaint-database). It is 1.7 GB unzipped. It is possible to load this into memory on most computers, but why should you have to.

Maybe you are wondering how many total complaints there are. Maybe you are wondering the number of complaints per state.

You may think to reach for [jq](https://stedolan.github.io/jq/). The default for `jq` is to load everything into memory, but it has a streaming mode.

> **WARNING**: these commands will be really slow

Default: takes 1m18s on my workstation and uses a bunch of memory

```sh
cat complaints.json | jq -c '.[]' | wc -l

1833581
```

Streaming: takes 7m6s on my workstation and uses less memory:
```sh
cat complaints.json | jq -cn --stream 'fromstream(1|truncate_stream(inputs))' | wc -l

1833581
```

# Introducing ndjson tool

Number of complaints in the dataset

takes 26s on my workstation

```sh
cat complaints.json | ndjson from-json d | wc -l

1833581
```

Number of complaints per state, 2 ways

**1st way**

takes 44s on my workstation

```sh
cat complaints.json | ndjson from-json d | ndjson pick-field d.state | sort | uniq -c | sort -n

...
...
243520 "CA"
```

**2nd way**

takes 2m56s on my workstation

```sh
cat complaints.json | ndjson from-json d | ndjson agg --group-by d.state --agg count d

...
{"_count":243520,"state":"CA"}
...
{"_count":1553,"state":"WY"}
```

## In-depth on the JSON selector patterns

Every command takes a selector to target a part of the JSON/NDJON. These selectors work very much like [d3](https://d3js.org/) JSON selectors. Each selector starts with `d`, to refer to the whole document (or for NDJSON this refers to 1 NDJSON element). Next is each JSON key or an index in a JSON array (d.key[5] for the 5th item in the array given by the key `key`).

`d.key[5]` refers to `5`.

```js
{"key":[1,2,3,4,5]}

```

Suppose we have the following JSON document, and we want to process it into NDJSON:
```js
{ "foo": [{ "bar": 1}, {"bar": 2}]}
```

Selector

```js
d.foo
```

Keys can be combined

```js
{"company_name": "Dunder Mifflin", "employees": [{"name": "Dwight", "age": 37}]}
```

Selector targeting `37`

```js
d.employees[0].age
```

### ndjson tool commands

```sh
SUBCOMMANDS:
    agg           Aggregatation commands on a grouped-by key
    filter        returns only json that matches filter expression
    from-json     Converts json to ndjson
    join          joins json file to ndjson stream
    pick-field    picks a field from all of the ndjson objects
```

## Caveats

This tool can't do everything `jq` can do and it never will. It is supposed to be simple and fast. Also the `agg` subcommand takes a lot of memory if the `group-by` field is not already sorted.

## Github repo

[ndjson-spatial](https://github.com/boydjohnson/ndjson-spatial) is the project that this tool is a part of.

[ndjson tool README.md](https://github.com/boydjohnson/ndjson-spatial/tree/master/ndjson)

# Conclusion

For gigantic JSON files there is a new way to process them into NDJSON. This NDJSON can be filtered and processed in relatively simple ways to find basic answers about the data.
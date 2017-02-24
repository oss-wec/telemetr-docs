+++
weight = 5
title = "Why telemetR"
date = "2017-02-22T12:09:09-08:00"
toc = true
next = "/introduction/dependencies"
prev = "/introduction/what-is-telemetr"

+++

Why should you choose telemetR?

In my experience data management at wildlife management agencies is an after thought. Animal telemetry data can be very complex and difficult to deal with. And the vendors don't make it easy. Prior to telemetR NDOW's management of telemetry data was non-existant.

1. the data was distributed across the organization
2. differing collection and storage methods
3. organization wide collaborative efforts were difficult
4. each time a dataset was needed, it needed to be created from scratch

## Background

Since 2010 NDOW has deployed about 1300+ gps devices. Most of these devices are deployed on large mammals (deer, elk, bighorn sheep, bears, mountain lions). This isn't all our data, many of the devices deployed on raptors and small mammals haven't been processed yet. Most of the devices on the large animals are from 3 vendors, ATS, Lotek, and Vectronic. Each company has a different method for downloading data. Each company provides their data in different formats. All these pieces of data were stored as sets of Excel files. As we approached 1.5 million relocations things got out of hand. We quickly realized we needed a database to manage all the collar data. So we got to work.

## A Database Solution

At the core of telemetR is a spatial database. The database stores animal/capture, device, and GPS data. The database will check the GPS data against a deployments inservice and outservice dates. There are a few QA/QC checks that can be run. The GPS data is available with a simple query.

```
SELECT *
FROM relocs_analysis
WHERE perm_id = 'Animal1';
```

The benefit of using a database is that all the data from all the disparate sources are stored in a standardized and centralized format.

## An API

An API makes data access effortless. The database can be accessed from any programing language to build applications, run analyses, or build reports.

## Applications

There are a few applications that are built using the telemter-api. These applications consume the API and make visualization and exploratory data analysis easy for general use.

## Open Source

The entire telemetR stack is open source. All the code, from the database to the applications, are available on GitHub. Anyone can contribute.

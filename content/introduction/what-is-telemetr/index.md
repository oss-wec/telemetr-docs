+++
date = "2017-02-17T08:57:01-08:00"
toc = true
next = "/next/path"
prev = "/prev/path"
weight = 5
title = "Installation"
+++

## Core Dependencies

1. PostgreSQL
2. PostGIS
3. Node.js
4. R

## telemetR Modules

telemetR is a modular data management system. Each module is developed independently and has it's own requirements. Below is a list of the modules.

1. *telemetr-db*: this is where all the telemetry data is stored. This is the minimum requirement.
2. *telemetr-generator*: this is an set of tools that will bootstrap the telemetr database. This isn't required but provides an easy method to create the database.
3. *telemetr-api*: an API that is used by many telemetr applications. This is require only if downstream applications are going to be used.
4. *applications*: applications developed to work with the telemeter database and API.

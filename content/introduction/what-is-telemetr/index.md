+++
date = "2017-02-17T08:57:01-08:00"
toc = true
next = "/introduction/why-telemetr"
prev = "/introduction"
weight = 1
title = "What is telemetR"
+++

telemetR is a modular data management system for animal movement data. At the foundation is a spatial database that stores all the data. Every other piece of telemetR software builds on the database, like legos, creating a system for storage, access, analysis, and visualization.

## Modules

telemetR is a modular system. Each piece is developed and maintained independently. Below is a list of all the different modules.

* *[telemetr-db](https://github.com/wiesr/telemetr-db)*: this is where all the telemetry data is stored. This is the minimum requirement.
* *[telemetr-generator](https://github.com/wiesr/telemetr-generator)*: this is an set of tools that will bootstrap the telemetr database. This isn't required but provides an easy method to create the database.
* *[telemetr-api](https://github.com/wiesr/telemetr-api)*: an API that is used by many telemetr applications. This is require only if downstream applications are going to be used.
4. *applications*: applications developed to work with the telemeter database and API.

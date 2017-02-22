+++
date = "2017-02-17T08:57:01-08:00"
toc = true
next = "/next/path"
prev = "/prev/path"
weight = 5
title = "Installing Dependencies"
+++

## Core Dependencies

1. PostgreSQL
2. PostGIS
3. Node.js
4. R

## Installing PostgreSQL

PostgreSQL is the database that is used by telemetR.

### macOS

There are prefered methods to install postgres on a mac.

#### [1) postgres.app](https://postgresapp.com/)

*from postgres.app website*

Postgres.app is a full-featured PostgreSQL installation packaged as a standard Mac app. It includes everything you need to get started: we’ve even included popular extensions like PostGIS for geo data and plv8 for Javascript.

Postgres.app has a beautiful user interface and a convenient menu bar item. You never need to touch the command line to use it – but of course we do include all the necessary command line tools and header files for advanced users.

Postgres.app updates automatically, so you get bugfixes as soon as possible.

The current version requires macOS 10.10 or later and comes with PostgreSQL versions 9.5 and 9.6, but we also maintain other versions of Postgres.app.



#### [2) homebrew](https://brew.sh/)

Homebrew is the missing package manager for macOS. If you use homebrew, you know the drill.

``` bash
$ brew update
$ brew install postgresql
```

If you don't already use brew I recommend looking into it and installing postgres.app (unless you really want to fall down the homebrew rabbit hole).

## PostGIS

*from [PostGIS website](http://postgis.net/)*

PostGIS is a spatial database extender for PostgreSQL object-relational database. It adds support for geographic objects allowing location queries to be run in SQL.

``` sql
SELECT superhero.name
FROM city, superhero
WHERE ST_Contains(city.geom, superhero.geom)
AND city.name = 'Gotham';
```

In addition to basic location awareness, PostGIS offers many features rarely found in other competing spatial databases such as Oracle Locator/Spatial and SQL Server. Refer to PostGIS Feature List for more details.

If you are using postgres.app, good news, you don't have anything else to do. If you are using a homebrew installation then you will need to `brew install postgis`.

Once PostGIS is installed it needs to be added to the database that will use it with:

```
psql <db_name>
CREATE EXTENSION postgis;
```

## Node.js

[Node.js](nodejs.org) is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world. Node is great because JavaScript can be written on the client as well as the server. This is a huge time saver (IMHO).

The telemetr-api uses Node.js as a web server to send resources from the database to the user. Learning JavaScript is easy if you already know another language (some syntax similarities with R). There is a bit of a learning curve, but in the long run it'll be worth while for many ecologists/biologists. A great resource for learning JavaScript is [freecodecamp.com](freecodecamp.com).

### macOS

Node is easily installed with homebrew, `brew install node`. Alternatively use the [download page](https://nodejs.org/en/download/) and choose the proper installation for your system.

## R

I'm going to bet you already have R installed on your system! If you do, make sure it is a recent release (3.3.0+). R will not be covered heavily, however I have some development planned for an R package.

### macOS

Install R and RStudio from the respective websites.

### Windows

Install R and RStudio from the respective websites.

## telemetR Modules

telemetR is a modular data management system. Each module is developed independently and has it's own requirements. Below is a list of the modules.

1. *telemetr-db*: this is where all the telemetry data is stored. This is the minimum requirement.
2. *telemetr-generator*: this is an set of tools that will bootstrap the telemetr database. This isn't required but provides an easy method to create the database.
3. *telemetr-api*: an API that is used by many telemetr applications. This is require only if downstream applications are going to be used.
4. *applications*: applications developed to work with the telemeter database and API.

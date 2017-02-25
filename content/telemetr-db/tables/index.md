+++
date = "2017-02-24T12:05:16-08:00"
toc = true
next = "/next/path"
prev = "/prev/path"
weight = 5
title = "Tables"

+++

First thing first, we need to create all the tables for the database. The code for the tables can be found (in this folder on GitHub)[https://github.com/wiesr/telemetr-db/tree/master/schema]. I like to create the tables in the order that the data flows in the database.

## Conventions

All tables will have an `id` field. This field is the primary key for that number. I've decided to use a sequential integer (serial) for all the primary keys for simplicity.

All tables have `created_at` and `updated_at` fields. These will keep track of the times new data is entered and when data is changed. `created_at` defaults to the current timestamp (`now()`). `updated_at` also defaults to `now()`, we will use a trigger to change this when the row is updated.

## Devices

This table holds all the telemetry devices. There cannot be any duplicates devices. There is a one to many relationship between this table and the `deployments` table.

``` sql
CREATE TABLE devices (
  id serial PRIMARY KEY,
  serial_num varchar(50) UNIQUE,
  frequency real,
  vendor varchar(50),
  model varchar(75),
  device_type varchar(50),
  mfg_date date,
  vhf_lot varchar(50),
  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now()
);
```

### Field Descriptions

| Field | Type | Description     |
| :------------- | :------------- | :------------- |
| id | |`serial` | sequentially generated primary key |
| serial_num | `varchar` | The serial number for the device. This can be called the device ID as well |
| frequency | `real` | The frequency of the VHF on the device |
| vendor | `varchar` | The manufacturer of the device |
| model | `varchar` | The model number or model description of the device |
| device_type | `varchar` | Whether or not the device is a VHF or GPS or other |
| mfg_date | `date` | The data the device was made |
| vhf_lot | `varchar` | Some VHF devices have a lot number |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |

## Study Lookup

This table holds information about the study area or project ID. We use these to easily query for specific animals based on what project/study they are associated with. This table should be seeded with some data when created. I'll include some examples from our study table.

``` sql
CREATE TABLE lkp_study (
  id serial PRIMARY KEY,
  study_name varchar(100),
  attributes jsonb,
  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now()
);
```

### Field Descriptions

| Field | Type | Description |
| :------------- | :------------- | :------------- |
| id | `serial` | sequentially generated primary key |
| study_name | `varchar` | Name that the project/study is commonly referred to |
| attributes | `jsonb` | Any other attributes related to the study. This is a json datatype. For proper insertion make sure data conforms to json standards |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |


### Data

An example of inserting data into this table.

``` sql
INSERT INTO lkp_study (study_name)
VALUES
  ('Ruby Mtn'),
  ('Simpson Park'),
  ('Carson');
```

### Animals

The `animals` table holds all the animal related data for collar deployments. An animal can be entered multiple times, each entry is for a different deployment. Generally only animals that have collars deployments should be entered.

``` sql
CREATE TABLE animals (
  id serial PRIMARY KEY,
  perm_id varchar(20),
  species varchar(4),
  serial_num varchar(50) REFERENCES devices(serial_num),
  cap_date date,
  sex varchar(8),
  age varchar(10),
  study_id integer REFERENCES lkp_study(id),
  mgmt_area integer,
  fate varchar,
  fate_date date,
  notes varchar,
  attributes jsonb,
  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now()
);

-- an index on perm_id to make searches faster
CREATE INDEX perm_id_idx
ON animals (perm_id);

COMMENT ON TABLE animals IS
'All the animals that are captured and have a telemetry device deployed on them. This list should be updated when an animal dies or the collar is removed. This is the fate and fate_date';
```

## Field Descriptions

| Field | Type | Description |
| :------------- | :------------- | :------------- |
| id | `serial` | sequentially generated primary key |
| perm_id | `varchar` | An ID for the animal. This can be a name or number. Preferably an ID that corresponds to a mark on the animal (not the collar id/serial number) |
| species | `varchar` | An alpha abbreviation of the species. For instance, MULD = mule deer, GOEA = golden eagle |
| cap_date | `date` | The date the animal is captured and the collar is deployed |
| sex | `varchar` | Sex of the animal |
| age | `varchar` | Categorical age class of the animal |
| study_id | `integer` | A reference to the study id the animal is associated with |
| mgmt_area | `integer` | A hunt unit or management area associated with the animal |
| fate | `varchar` | The ultimate fate of the deployment. This can be due to the death of the animal or the collar dropping of or quit working |
| fate_date | `date` | The date the fate occurred |
| notes | `varchar` | Any relevant notes |
| attributes | `jsonb` | Any additional attributes to add to the animal, for instance: test results, numeric age, body condition score, etc. |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |

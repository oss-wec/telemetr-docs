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

## Animals

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

### Field Descriptions

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

### Data

coming soon ...

## Deployments

An animal can be given many devices over it's lifetime and a collar can be deployed on many animals. This animal-device association is stored in the `deployments` table.

Each row in this table is created after an animal is entered in the `animals` table. When an animal dies and the `animals.fate_date` and `animals.fate` are recorded in the database the `deployments.outservice` is updated for the deployment.

### Field Descriptions

| Field | Type | Description |
| :------------- | :------------- | :------------- |
| id | `integer` | sequentially generated primary key |
| animal_id | `integer` | foreign key to the animal ID in the `animal` table |
| device_id | `integer` | foreign key to the device ID in the `devices` table |
| inservice | `date` | The date the device is deployed on the animals. Should be from `animals.cap_date`. |
| outservice | `date` | The date the device is removed from the animal. Should be from `animals.fate_date`. |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |

### Data

coming soon ...

## GPS

The `gps` table is where new telemetry locations are initially uploaded. This data is nearly raw data from the vendorsand should be slightly processed, which means that the data only contains the fields id the field description section. This data does not have an `animals.perm_id` assigned to it (the database takes care of this).

Once data is inserted a function is triggered (function name & link) to check the `deployments.device_id`, `deployments.inservice`, and `deployments.outservice` to determine the `deployments.animal_id` for the locations, then insert the row into the `relocations` table.

### Field Descriptions

| Field | Type | Description |
| :------------- | :------------- | :------------- |
| id | `integer` | sequentially generated primary key |
| serial_num | `varchar` | Serial number of the deployed device |
| acq_time_utc | `timestamp` | Acquisition time of the location with the timezone set to UTC |
| acq_time_lcl | `timestamp` |Acquisition time of the location with the timezone set to the local time where the device is deployed |
| latitude | `numeric` | latitude of the location|
| longitude | `numeric` | longitude of the location |
| altitude | `numeric` | Altitude of the location |
| easting | `numeric` | *OPTIONAL* Easting of the location |
| northing | `numeric` | *OPTIONAL* Northing of the location |
| activity | `numeric` | A measure of the activity of the animal between GPS fixes. The definition of this changes depending on the vendor of the device. |
| temperature | `numeric` | Ambient temperature at the acquisition time. |
| hdop | `numeric` | A measure of error |
| pdop | `numeric` | A measure of error |
| n_sats | `integer` | number of satellites used to record the location |
| fixtype | varchar | Whether the GPS is 2 dimensional or 3 dimensional. |
| gps_volts | real | A measure of the remaining battery life |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |

### Data

Coming soon ...

## Relocations

The `relocations` table holds parsed relocation data for each deployment. This data has been filtered to only include relocations that occur between an `deployments.inservice` and `deployments.outservice` date. The data in this table is also associated with an `animals.animal_id`. This data is "nearly analysis ready". This table doesn't include all of the fields from the `gps` table.

### Field Descriptions

| Field | Type | Description |
| :------------- | :------------- | :------------- |
| id | `integer` | sequentially generated primary key |
| gps_id | `integer` | Foreign Key references `gps.id`|
| animal_id | `integer` | Foreign key reference to `animals.id` |
| device_id | `integer` | Foreign key reference to `devices.id` |
| acq_time_utc | `timestamp` | Acquisition time of the location with the timezone set to UTC |
| acq_time_lcl | `timestamp` |Acquisition time of the location with the timezone set to the local time where the device is deployed |
| latitude | `numeric` | latitude of the location|
| longitude | `numeric` | longitude of the location |
| geom | `geometry(Point, 4326)` | Spatially referenced location |
| altitude | `numeric` | Altitude of the location |
| validity_code | `integer` | Foreign key reference to `validity_code.code`. Describes the validity of a location. |
| easting | `numeric` | Easting of the location |
| northing | `numeric` | Northing of the location |
| utm_srid | `integer` | Calculated UTM Zone ID |
| activity | `numeric` | A measure of the activity of the animal between GPS fixes. The definition of this changes depending on the vendor of the device. |
| temperature | `numeric` | Ambient temperature at the acquisition time. |
| created_at | `timestamp` | timestamp the row is created |
| updated_at | `timestamp` | timestamp the row is updated |

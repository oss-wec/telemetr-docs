+++
date = "2017-02-28T10:01:42-08:00"
toc = true
next = "/next/path"
prev = "/prev/path"
weight = 10
title = "Views"

+++

Views are stored queries that can be used as read-only tables. They can be used to store complex queries into simple SQL calls. Views do not store the actual data, instead the only store the view definition.

## Active Device Deployments

``` sql
CREATE VIEW active_deployments AS
SELECT
  animals.perm_id,
  animals.species,
  animals.sex,
  animals.age,
  devices.serial_num,
  deployments.inservice,
  deployments.outservice,
  deployments.animal_id,
  deployments.device_id,
  deployments.id AS deployment_id
FROM (deployments
  INNER JOIN animals ON deployments.animal_id = animals.id)
  INNER JOIN devices ON deployments.device_id = devices.id
WHERE deployments.outservice IS NULL
ORDER BY deployments.inservice DESC;
```

`active_deployments` will show all of the devices that are actively deployed on animals (i.e. `deployments.outservice` is null). 


### 1- Introduction

This section explain the process to load our datasets into Postgres Database. In order to facilitate data processing, features extraction and modelisation we will make date into appropriate structure.


### 2- Database preparation

> Log to Database

```sh
# log to database
sudo su - datatech

# log to database
psql dev_db datatech 
```

> Create table

We will create a new table named bearing with the following columns:  
- __id__: The record id
- __record_time__: Recod timestamp 
- __set_id__: Set identifier. Each set describes a test-to-failure experiment (3 sets for 3 independant tests)
- __bearing_id__: Bearing ID. There is four bearings installed on a shaft.
- __x_axis__: X-Axis accelerometer measure (two accelerometers for each bearing [x- and y-axes] for data set 1, one accelerometer for each bearing for data sets 2 and 3)
- __y_axis__: Y-Axis accelerometer measure (two accelerometers for each bearing [x- and y-axes] for data set 1, one accelerometer for each bearing for data sets 2 and 3)


```sql
/* create table */

DROP TABLE IF EXISTS bearing;

CREATE TABLE bearing (
  id serial PRIMARY KEY,  
  record_time VARCHAR (255) NOT NULL, 
  set_id VARCHAR (255) NOT NULL,  
  bearing_id VARCHAR (255), 
  x_axis FLOAT,
  y_axis FLOAT
;
 
```

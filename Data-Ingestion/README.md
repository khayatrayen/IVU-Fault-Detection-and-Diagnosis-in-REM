
### 1- Introduction

This section explain the process to load our datasets into Postgres Database. In order to facilitate data processing, features extraction and modelisation we will make date into appropriate structure.


### 1

```sh
# install some useful packages
sudo su -
yum -y install p7zip
yum -y install wget

cd /tmp
wget https://www.rarlab.com/rar/rarlinux-x64-5.6.0.tar.gz
tar -zxvf rarlinux-x64-5.6.0.tar.gz
cd rar
cp -v rar unrar /usr/local/bin/

yum -y groupinstall "Development Tools"
yum -y install python3-devel
yum -y install postgresql-libs
yum -y install postgresql-devel
pip install psycopg2

```

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
);
 
```

### 3- Download bearing datatset

```sh
# create data folder
mkdir dataset
cd dataset

# download sataset
wget -O IMS.7z https://ti.arc.nasa.gov/c/3/

# unzip file
7za x IMS.7z

# unrar dataset
unrar e 1st_test.rar 1st_test &
unrar e 2nd_test.rar 2nd_test &
unrar e 3rd_test.rar 3rd_test &
```

### 4- Process data and insert it into database

```sh
# install database driver
pip3 install psycopg2
```

```py
import psycopg2


DatabaseManager(object):
  
  def __init__(self):
    try:
      self.conn = psycopg2.connect("dbname='dev_db' user='datatech' host='localhost' password='datatech'")
    except:
      print "Unable to connect to the database"

  def query(self, sql_query):
    cur = self.conn.cursor()
    cur.execute(sql_query)
    rows = cur.fetchall()
    self.conn.commit()
    cur.close()    
    return row
  
  def insert(self, insert_query, data):
    cur = self.conn.cursor()    
    psycopg2.extras.execute_values (
      curr, insert_query, data, template=None, page_size=100
    )
    self.conn.commit()
    cur.close() 

  def close(self):
    self.conn.close()


create_table_query = 'create table t (a integer, b varchar(250))'
list_table_query = 'select * from t limit 10'
data = [(1,'x'), (2,'y')]
insert_query = 'insert into t (a, b) values %s'


db = DatabaseManager()
val_1 = db.query(create_table_query)
val_2 = db.query(list_table_query)
db.insert(insert_query, data)
val_3 = db.query(list_table_query)

for val in val_1:
  print ('val 1')
  print val
 
print ('\n')

for val in val_1:
  print ('val 1')
  print val

print ('\n')

for val in val_1:
  print ('val 1')
  print val

print ('\n')

```



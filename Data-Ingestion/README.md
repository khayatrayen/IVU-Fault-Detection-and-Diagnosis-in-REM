
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
pips install psycopg2
pip3 install pandas

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
mkdir 1st_test
cp 1st_test.rar 1st_test
cd 1st_test
unrar e 1st_test.rar
rm 1st_test.rar
cd ..

mkdir 2nd_test
cp 2nd_test.rar 2nd_test
cd 2nd_test
unrar e 2nd_test.rar
rm 2nd_test.rar
cd ..

mkdir 3rd_test
cp 3rd_test.rar 3rd_test
cd 3rd_test
unrar e 3rd_test.rar
rm 3rd_test.rar
cd ..
```

### 4- Process data and insert it into database

```sh
# install database driver
pip3 install psycopg2
```

```py
import psycopg2
import psycopg2.extras as extras


class DatabaseManager:
    
    def __init__(self):
        try:
            self.conn = psycopg2.connect("dbname='dev_db' user='datatech' host='localhost' password='datatech'")
        except:
            print ("Unable to connect to the database")
    
    def query(self, sql_query):
        cur = self.conn.cursor()
        cur.execute(sql_query)
        rows = cur.fetchall()
        self.conn.commit()
        cur.close()    
        return rows
    
    def insert(self, insert_query, data):
        cur = self.conn.cursor()    
        extras.execute_values (
            cur, insert_query, data, template=None, page_size=100
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
db.close()


for val in val_1:
  print ('val 1')
  print (val)
 
print ('\n')

for val in val_2:
  print ('val 2')
  print (val)

print ('\n')

for val in val_3:
  print (val)

print ('\n')

```

```py
import os

from os import listdir
from os.path import isfile, join
import pandas as pd

BASE_DIR = '/home/datatech/dataset'
sets = ['1st_test', '2nd_test', '3rd_test']

for set in sets:
    print ('Processing set {0}'.format(set))
    files = [f for f in listdir(os.path.join(BASE_DIR, set)) if isfile(join(os.path.join(BASE_DIR, set), f))]    
    count = 1
    for file in files:        
        print ('Processing file {0}/{1}...'.format(str(count), str(len(files))))
        df = pd.DataFrame(columns=['record_time', 'set_id', 'bearing_id', 'x_axis', 'y_axis'])
        data = pd.read_csv(os.path.join(BASE_DIR, set, file), sep='\t', header=None)        
        data.head()
        if set == '1st_test':
            bearing_1 = data[[0, 1]]
            bearing_1.columns = ['x_axis', 'y_axis']
            bearing_1['bearing_id'] = 1
            
            bearing_2 = data[[2, 3]]
            bearing_2.columns = ['x_axis', 'y_axis']
            bearing_2['bearing_id'] = 2
            
            bearing_3 = data[[4, 5]]
            bearing_3.columns = ['x_axis', 'y_axis']
            bearing_3['bearing_id'] = 3
            
            bearing_4 = data[[6, 7]]
            bearing_4.columns = ['x_axis', 'y_axis']
            bearing_4['bearing_id'] = 4
            
            bearing_df = pd.concat([bearing_1, bearing_2, bearing_3, bearing_4], ignore_index=True)
            bearing_df['set_id'] = 1
            bearing_df['record_time'] = file
            df = bearing_df[['record_time', 'set_id', 'bearing_id', 'x_axis', 'y_axis']]
        else:
            bearing_1 = data[[0]]
            bearing_1.columns = ['x_axis']
            bearing_1['y_axis'] = 0
            bearing_1['bearing_id'] = 4
            
            bearing_2 = data[[1]]
            bearing_2.columns = ['x_axis']
            bearing_1['y_axis'] = 0
            bearing_2['bearing_id'] = 4
            
            bearing_3 = data[[2]]
            bearing_3.columns = ['x_axis']
            bearing_3['y_axis'] = 0
            bearing_3['bearing_id'] = 4
            
            bearing_4 = data[[3]]
            bearing_4.columns = ['x_axis']
            bearing_4['y_axis'] = 0
            bearing_4['bearing_id'] = 4
            
            bearing_df = pd.concat([bearing_1, bearing_2, bearing_3, bearing_4], ignore_index=True)
            if set == '2nd_test':
              bearing_df['set_id'] = 2
            else:
              bearing_df['set_id'] = 3
            bearing_df['record_time'] = file
            df = bearing_df[['record_time', 'set_id', 'bearing_id', 'x_axis', 'y_axis']]
            
        df.head()
        
        print ('File {0} Processed successfully!'.format(str(count)))
        count+=1        
        break
        
    print (" ")
```







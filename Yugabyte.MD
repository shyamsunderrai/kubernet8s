# Yugabyte Technical Interview 

## Required Questions

### Docker & Podman tasks
#### Process utilization & other tasks
```bash
docker exec -it yugabyte sh
export tserverpid=$(ps -eaf | egrep -i yb-tserver | grep -v grep | awk '{print $2}') && ps -p $tserverpid -o %cpu
%CPU
 1.5
ps -eo pid,pcpu,comm | egrep yb-tserver | awk '{print $2}'
1.5
 
docker exec -it yugabyte ip -4 -o address
1: lo    inet 127.0.0.1/8 scope host lo\       valid_lft forever preferred_lft forever
17: eth0    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0\       valid_lft forever preferred_lft forever

Shyams-MBP:kubernet8s samurai$ docker exec -it yugabyte ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 361  bytes 30460 (29.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 359  bytes 110880 (108.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 39883  bytes 6003993 (5.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 39883  bytes 6003993 (5.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### Setup customer table and insert records
```bash
create table customers (id serial primary key, name char(30) unique, email varchar(50) unique, customer_since date); 
yugabyte=# insert into customers (id,name,email,customer_since) values(82,'yugabyte','support@yugabyte.com','2020-08-31');
INSERT 0 1
yugabyte=# select * from customers where name='yugabyte';
 id |              name              |        email         | customer_since 
----+--------------------------------+----------------------+----------------
 82 | yugabyte                       | support@yugabyte.com | 2020-08-31
(1 row)


## 2c. yugabyte=# insert into customers (id,name,email,customer_since) values(83,'yugabyteA','support@yugabyteA.com','2020-08-31'),(84,'yugabyteB','support@yugabyteB.com','2020-08-31'),(85,'yugabyteC','support@yugabyteC.com','2020-08-31'),(86,'yugabyteD','support@yugabyteD.com','2020-08-31');
INSERT 0 4

yugabyte=# select count(*) from customers;
 count 
-------
     5
(1 row)
```

### Obtain output of pg_stat_activity
```bash
docker exec -it yugabyte /home/yugabyte/bin/ysqlsh 
ysqlsh (11.2-YB-2.11.0.0-b0)
Type "help" for help.

yugabyte=# \x
Expanded display is on.
yugabyte=# select datid,datname,client_addr,client_hostname, query from pg_stat_activity ;
-[ RECORD 1 ]---+--------------------------------------------------------------------------------
datid           | 13248
datname         | yugabyte
client_addr     | 127.0.0.1
client_hostname | 
query           | select datid,datname,client_addr,client_hostname, query from pg_stat_activity ;
-[ RECORD 2 ]---+--------------------------------------------------------------------------------
datid           | 
datname         | 
client_addr     | 
client_hostname | 
query           | 
```

## Linux Fundamental Questions
#### Most recent logs from /var/log/yugabyte 
```bash
ls -larth /var/log/yugabyte | tail  /// Last few entries are latest, we can check the timestamp and date 
find /var/log/yugabyte -type f -exec stat -c '%X %n' {} \; | sort -nr | awk 'NR==1,NR==4 {print $2}' /// Can change NR==n to list latest "n" files
```

#### Creating a file named "cat"
```bash
touch /tmp/my_animals/cat && echo '`name: Mittens `' > /tmp/my_animals/cat && chmod 0642 /tmp/my_animals/cat
//Same can be put inside a file replacing a newline for each "&&" command, should do the same thing
```

#### Ansible Playbook for automating the "cat" file creation
```yaml
shyamrai@Shyams-MBP ansible % cat task.yaml 
---
- name: Script cat
  hosts: localhost
  tasks:
    - name: Create the directory if not exists "/tmp/my_animals"
      file:
        path: /tmp/my_animals
        state: directory
    - name: To create the "cat" file
      shell: "touch /tmp/my_animals/cat && echo '`name: Mittens `' > /tmp/my_animals/cat && chmod 0642 /tmp/my_animals/cat"
```

##### Execution of Ansible Playbook
```bash
shyamrai@Shyams-MBP ansible % ansible-playbook task.yaml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Script cat] ************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************
ok: [localhost]

TASK [Create the directory if not exists "/tmp/my_animals"] ******************************************************************************************************************************
changed: [localhost]

TASK [To create the "cat" file] **********************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *******************************************************************************************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

shyamrai@Shyams-MBP ansible % 
shyamrai@Shyams-MBP ansible % 
shyamrai@Shyams-MBP ansible % cat /tmp/my_animals 
cat: /tmp/my_animals: Is a directory
shyamrai@Shyams-MBP ansible % ls /tmp/my_animals
cat
shyamrai@Shyams-MBP ansible % cat /tmp/my_animals/cat
`name: Mittens `
```

##### Postgres DB not running
```bash
///Output of the command shows that the service has been disabled 
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
///Enable the service using sudo privileges
   systemctl enable postgresql
///Start the service 
   systemctl start postgresql
///Validate if the service is enabled and active
   systemctl status postgresql
///Should show something like "Active: Active"         
///Alternatively, we can try 
  - Connecting to the services and valdating if we can run a simple select like "select * from pg_stat_activity"
  - ps -eaf | egrep postgresql 
  - Check the PID file location /var/lib/pgsql/<pid> file /// Not entirely sure about the location as it can be customized 
```  


##### Difference between “VSZ” and “RSS”
```bash
///RSS is the Resident Set Size memory, which is the actual memory allocated to process, here 11700 is the RSS allcation for /sbin/init process
///VSZ is virtual memory size, contains all the memory that the process can acccess including swapped out memory, allocated memory that is unused, memory from shared libraries
```

##### Run a single command as sudo and as another user
```bash
visudo 
myuser  ALL=NOPASSWD: /usr/bin/echo //Giving "myuser" the privilege to run /usr/bin/echo command

///runuser can be used to run command as another users
runuser -l 'desireduser' -c 'ls -l' // desired command can be run as another user 
```


## DATABASE FUNDAMENTALS
##### Schema for "users" table
```sql
create sequence seq_web_user_userid start 1; 
create table users (
userid bigint DEFAULT NEXTVAL('seq_web_user_userid') NOT NULL,
username varchar(20) NOT NULL,
email varchar(40) NOT NULL,
phone bigint NOT NULL,
fname char(20) NOT NULL,
lname char(20) NOT NULL,
"role" char(12),
address varchar(70) NOT NULL,
country char(20),
user_add_date date DEFAULT current_date, 
last_login timestamp DEFAULT current_timestamp,
CONSTRAINT web_user_email UNIQUE (email),
CONSTRAINT web_user_uname PRIMARY KEY (username));

//// Indexes are created by default with declaration of PRIMARY or UNIQUE constraints 

                                                                 Table "public.users"
    Column     |            Type             | Collation | Nullable |                 Default                  | Storage  | Stats target | Description 
---------------+-----------------------------+-----------+----------+------------------------------------------+----------+--------------+-------------
 userid        | bigint                      |           | not null | nextval('seq_web_user_userid'::regclass) | plain    |              | 
 username      | character varying(20)       |           | not null |                                          | extended |              | 
 email         | character varying(40)       |           | not null |                                          | extended |              | 
 phone         | bigint                      |           | not null |                                          | plain    |              | 
 fname         | character(20)               |           | not null |                                          | extended |              | 
 lname         | character(20)               |           | not null |                                          | extended |              | 
 role          | character(12)               |           |          |                                          | extended |              | 
 address       | character varying(70)       |           | not null |                                          | extended |              | 
 country       | character(20)               |           |          |                                          | extended |              | 
 user_add_date | date                        |           |          | CURRENT_DATE                             | plain    |              | 
 last_login    | timestamp without time zone |           |          | CURRENT_TIMESTAMP                        | plain    |              | 
Indexes:
    "web_user_uname" PRIMARY KEY, lsm (username HASH)
    "web_user_email" UNIQUE CONSTRAINT, lsm (email HASH)
    
/// With lack of enforcements from application teams, introduce the following
- NOT NULL on most of the columns ensuring there are no NULL values
- PRIMARY & UNIQUE constraints would ensure that the lookup values are unique to each user
- DEFAULT fillers and like timestamp and date and userID would be great alternatives to reduce the probabilities of reducing null and enforcing inserts
/// No need to add additional indexes as lookup values already have the indexes on them i.e. email and username 
/// Distribution would be ideal on integer or ID based columns, hence USER ID  was introduced. haracter/Varchar/Date/Timestamp are not ideal. Country would not be an ideal choice as there can be a lot of duplicates and distribution might be impacted due to this. This would impact both reads/writes.
    
```

##### LSB & B-Tree
###### LSM 
```bash
- Optimal for write-heavy/update loads due to the way it manages its operations using memtable and String Sorted Tables 
- Potential of low read performance when the number of SSTables increase and the duration between compaction of SSTables is high
- Can minimize storage overhead via compaction and it can interfere with performance
- Option of enabling bloom filter is available, but is purely probable
- Have a higher read and space amplification requiring more CPU/disk compared to B-tree 
- Used in tools like BigTable, Cassandra, Scylla, RocksDB, YugabyteDB, TiDB
```

###### B-Tree
```bash
- Better for short range lookup queries
- Complicated for inserts and deletes due to re-branching required when new values are either added or deleted. Effort is due to maintaining of ordered data with random writes reduces the the write performance
- Requires double writes to Write ahead log and to the actual page of btree itself, perhaps more to maintain consistency depending on DB engines
- Can leave empty spaces in storage causing fragmentation 
- A minimal change in data also requires for reading the entire block, modifying it and then rewriting it.
- Used in most of the popular RDBMS databases like Oracle, Postgres, MySQL, MSSQL server, IBM DB2, including Couchbase.  
```


## QUERY LANGUAGE
##### Performance issues when logging with phone number is slower compared to email or username. 
- Phone number may contain null values causing performance bottleneck on unnecessary reads/resources. Add "not null" constraint to avoid null values for new additions 
- Create an index on "phone" attribute to enhance reads, similar to preexisting indexes for email and username  
- Hash based sharding strategy on "phone" column can help with point based lookup since users lookup based on one phone number
- Can have a integer or bigint data type rather than varchar to avoid type casting 

```bash
explain select count(*) from dummy where id = 650661007;
                                      QUERY PLAN                                        
------------------------------------------------------------------------------------------
Aggregate  (cost=5.15..5.16 rows=1 width=8)
 ->  Index Only Scan using check_lsm_bigint on dummy  (cost=0.00..5.12 rows=10 width=0)
       Index Cond: (id = 650661007)
(3 rows)

explain select count(*) from dummy where ids = '650661007';
                                        QUERY PLAN                                        
------------------------------------------------------------------------------------------
 Aggregate  (cost=5.15..5.16 rows=1 width=8)
   ->  Index Only Scan using check_lsm_string on dummy  (cost=0.00..5.12 rows=10 width=0)
         Index Cond: (ids = '650661007'::text)
(3 rows)

```

**Skipping Cassandra quiz as I don't believe I'd have enough time to complete this.**


## Yugabyte
#### Snapshot isolation level error 
##### Session 1 
```bash
yugabyte=# select * from ttest;
select * from ttest;
 col1 
------
(0 rows)
```

##### Session 2
```bash
yugabyte=# begin transaction;
begin transaction;
BEGIN
yugabyte=# insert into ttest values (1),(2),(3);
insert into ttest values (1),(2),(3);
INSERT 0 3
```

##### Session 1:
```bash
yugabyte=# truncate ttest;   //// Replacing this with delete from ttest should not reproduce the error
truncate ttest;
TRUNCATE TABLE
```

##### Session 2:
```bash
yugabyte=# select * from ttest;
select * from ttest;
ERROR:  Operation failed. Try again.: Unknown transaction, could be recently aborted: 006c406b-04b5-44ae-874a-967747e4eb56
yugabyte=# 
```

** Truncate is not transactional in nature, hence, the workaround could be to use "DELETE from ttest"  **

### Sharding Strategies
Currently supported sharding strategies in YugabyteDB are Hash & Range Sharding

##### Hash Sharding
```bash
- Can have at most 65336 tablets/shards 
- Hash value of the given attribute is used to determine the hash value for generating shards
- Ideal for huge/scalable workloads by managing the allocation of tablets to new nodes being added to the cluster, without huge data reassignment 
- ">"" or "< or "between" types of lookups are inefficient for this hashing strategy 
- Great for point based lookups 
```

##### Range Sharding
```bash
- Range sharding starts with one shard and then grows according to the values in the primary KEY
- Great for range based lookups like "between" and/or "lower" bound and "upper" bound value lookups 
- Since the sharding starts with one, initially one node handles most of the load 
- Hot spots can be introduced due to some nodes taking more load as most frequently requested ranges might persist only on certain nodes 
```

#### Hash Sharding
##### Session 1
```bash
kubectl --namespace yb-demo exec -it yb-tserver-0 -- sh -c "cd /home/yugabyte && ysqlsh -h yb-tserver-0 --echo-queries"
mydemo=# create table dummy (id int, name varchar(20), primary key(id HASH));
create table dummy (id int, name varchar(20), primary key(id HASH));
CREATE TABLE
```

##### Session 2: Obtained the table UUID from yb-master UI 
```bash
Obtained Table ID: 0000402200003000800000000000405d
```

##### Session 3: Attached to tserver pod & validate WAL and SSTable file along with Shards/Tablet(s)
```bash
kubectl exec pod/yb-tserver-0 -c yb-tserver -it -n yb-demo -- /bin/bash
/mnt/disk1/yb-data/tserver/wals
[root@yb-tserver-0 wals]# ls -larth | tail
drwxr-xr-x  3 root root 4.0K Dec  9 13:20 table-0000402200003000800000000000405d  //// This is the WAL file 
drwxr-xr-x 25 root root 4.0K Dec  9 13:20 .
[root@yb-tserver-0 wals]# 
```

##### Files within this WAL entry when the table is empty 
```bash
[root@yb-tserver-0 cores]# /home/yugabyte/tserver/bin/log-dump  -print_entries=pb /mnt/disk1/yb-data/tserver/wals/table-0000402200003000800000000000405d/tablet-71723d8d67504b70bdf2c895361d8b2f/wal-000000001 
W1209 14:10:03.490593 49600 log_util.cc:221] Could not read footer for segment: /mnt/disk1/yb-data/tserver/wals/table-0000402200003000800000000000405d/tablet-71723d8d67504b70bdf2c895361d8b2f/wal-000000001: Not found (yb/consensus/log_util.cc:463): Footer not found. Footer magic doesn't match
Header:
major_version: 1
minor_version: 0
tablet_id: "71723d8d67504b70bdf2c895361d8b2f"
sequence_number: 1
schema {
  columns {
    id: 0
    name: "id"
    type {
      main: INT32
    }
    is_key: true
    is_hash_key: true
    is_nullable: false
    is_static: false
    is_counter: false
    sorting_type: 0
    order: 1
  }
  columns {
    id: 1
    name: "name"
    type {
      main: STRING
    }
    is_key: false
    is_nullable: true
    is_static: false
    is_counter: false
    sorting_type: 0
    order: 2
  }
  table_properties {
    contain_counters: false
    is_transactional: true
    consistency_level: STRONG
    use_mangled_column_name: false
    is_ysql_catalog_table: false
    retain_delete_markers: false
  }
  colocated_table_id {
  }
}
schema_version: 0
I1209 14:10:03.513645 49600 log_util.cc:620] Scanning /mnt/disk1/yb-data/tserver/wals/table-0000402200003000800000000000405d/tablet-71723d8d67504b70bdf2c895361d8b2f/wal-000000001 for valid entry headers following offset 202...
I1209 14:10:03.515038 49600 log_util.cc:666] Found no log entry headers
I1209 14:10:03.515374 49600 log_util.cc:543] Ignoring log segment corruption in /mnt/disk1/yb-data/tserver/wals/table-0000402200003000800000000000405d/tablet-71723d8d67504b70bdf2c895361d8b2f/wal-000000001 because there are no log entries following the corrupted one. The server probably crashed in the middle of writing an entry to the write-ahead log or downloaded an active log via remote bootstrap. Error detail: Corruption (yb/consensus/log_util.cc:739): Invalid checksum in log entry head header: found=0, computed=2351477386: Log file corruption detected. Failed trying to read batch #3 at offset 202 for log segment /mnt/disk1/yb-data/tserver/wals/table-0000402200003000800000000000405d/tablet-71723d8d67504b70bdf2c895361d8b2f/wal-000000001: Prior batch offsets: 124 176 202; Last log entries read: [REPLICATE (1.1)]
Entry:
type: REPLICATE
replicate {
  id {
    term: 1
    index: 1
  }
  hybrid_time: 6713585307021705216
  op_type: NO_OP
  committed_op_id {
    term: 0
    index: 0
  }
  noop_request {
  }
}

```

##### There is only one tablet ID/Shard even after multiple inserts
```bash
[root@yb-tserver-0 cores]# ls -lrth /mnt/disk1/yb-data/tserver/data/rocksdb/table-0000402200003000800000000000405d/
total 12K
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f.snapshots
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f.intents



#### Inserted multiple values 
mydemo=# insert into dummy values (99,'Random');
insert into dummy values (99,'Random');
INSERT 0 1
mydemo=# insert into dummy values (100,'Random');
insert into dummy values (100,'Random');
INSERT 0 1
mydemo=# select count(*) from dummy;
select count(*) from dummy;
 count 
-------
   100
(1 row)

/// Seems like multiple inserts went into the same shard and no new Tablet ID were allocated for this table even from yb-master UI interface 

[root@yb-tserver-0 cores]# ls -lrth /mnt/disk1/yb-data/tserver/data/rocksdb/table-0000402200003000800000000000405d/
total 12K
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f.snapshots
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f
drwxr-xr-x 2 root root 4.0K Dec  9 14:08 tablet-71723d8d67504b70bdf2c895361d8b2f.intents
```

** To test the layout, tried to create a new table with SPLIT INTO 3 SHARDS which does reflect three different Tablet IDs/Shards created for a given table **








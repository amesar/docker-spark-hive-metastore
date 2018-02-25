# Spark Hive Metastore

Demonstrates usage of Spark and Hive sharing a common MySQL metastore.

## Overview

**Files**
  * [conf/hive-site.xml](conf/hive-site.xml) - Shared between Spark and Hive. Lives in /opt/spark/conf and /opt/hive/conf.
  * [table_data/tpcds/customer/customer.psv](table_data/tpcds/customer/customer.psv) - 1000 lines from TPCDS customer table

**Persisting Data**

In order to save data between container runs, we use Docker's volume mount feature to persist data to the host local disk in the directory 'container_data'.
 * mysql - MySQL data from /var/lib/mysql
 * spark/warehouse - /shared_data/hive/warehouse - hive.metastore.warehouse.dir
 * hive/warehouse - /shared_data/hive/warehouse - hive.metastore.warehouse.dir

## Launch the containers

```
docker-compose -f docker-compose.yml up -d
```

List your containers
```
docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
472bf648d020        mysql-shm:5.6.38    "docker-entrypoint..."   15 minutes ago      Up 15 minutes       0.0.0.0:13306->3306/tcp                                    mysql-shm
c3b0762fc084        ubuntu-shm:latest   "/usr/sbin/sshd -D"      5 minutes ago       Up 5 minutes        22/tcp                                                     ubuntu-shm
d83018213e76        java8-shm:latest    "/usr/sbin/sshd -D"      4 minutes ago       Up 4 minutes        22/tcp                                                     java8-shm
3178ac9a165a        spark-shm:2.2.1     "/usr/sbin/sshd -D"      5 minutes ago       Up 5 minutes        22/tcp, 0.0.0.0:14040->4040/tcp, 0.0.0.0:18080->8080/tcp   spark-shm
afa8be41f5b6        hive-shm:1.2.2      "/usr/sbin/sshd -D"      5 minutes ago       Up 5 minutes        22/tcp, 0.0.0.0:10000->10000/tcp                           hive-shm
```

## Spark

Login to the Spark container
```
docker exec -i -t spark-shm /bin/bash
```

Start Spark SQL shell
```
/opt/spark/bin/spark-sql
```

Create first table
```
create database tpcds;
use tpcds;

drop  table if exists customer;
create table customer (
    c_customer_sk             bigint,
    c_customer_id             string,
    c_current_cdemo_sk        bigint,
    c_current_hdemo_sk        bigint,
    c_current_addr_sk         bigint,
    c_first_shipto_date_sk    bigint,
    c_first_sales_date_sk     bigint,
    c_salutation              string,
    c_first_name              string,
    c_last_name               string,
    c_preferred_cust_flag     string,
    c_birth_day               int,
    c_birth_month             int,
    c_birth_year              int,
    c_birth_country           string,
    c_login                   string,
    c_email_address           string,
    c_last_review_date        string
)
row format delimited fields terminated by '|'
LOCATION '/shared_data/table_data/tpcds/customer' ;
```

Check the table out
```
select count(*) from customer;

1000
```


To access the Spark web UI: [http://localhost:14040](http://localhost:14040).

## Hive

Login to the Hive container
```
docker exec -i -t hive-shm /bin/bash
```

Start Hive CLI
```
/opt/hive/bin/hive
```

Check the table out
```
use tpcds;

select count(*) from customer;

1000
```

## MySQL

The metastore database is called hive_metastore. It is lazily created when you first execute a Spark or Hive command.

Login to the MySQL container
```
docker exec -i -t mysql-shm /bin/bash
```

Launch MySQL client
```
mysql --password=mysecret
```

Run some interesting commands
```
use hive_metastore;

select * from TBLS;
```

## Links of interest

* [AdminManual MetastoreAdmin](https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin) - Apache Hive wiki
  * [Metastore Entity Relationship Diagram](https://issues.apache.org/jira/secure/attachment/12471108/HiveMetaStore.pdf) - PDF download
* [Configuring the Hive Metastore for CDH](https://www.cloudera.com/documentation/enterprise/5-14-x/topics/cdh_ig_hive_metastore_configure.html) - CDH 5.14
* [Integrating Your Central Apache Hive Metastore with Apache Spark on Databricks](https://databricks.com/blog/2017/01/30/integrating-central-hive-metastore-apache-spark-databricks.html) - Jan. 2017
* [External Hive Metastore](https://docs.databricks.com/user-guide/advanced/external-hive-metastore.html) - Databricks documentation


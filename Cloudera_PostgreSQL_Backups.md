---
title: Cloudera PostgreSQL Backups
keywords: Cloudera PostgreSQL Backup Restore
last_updated: 'November, 2016'
tags:
  - Cloudera PostgreSQL Backup Restore
summary: Backing up and restoring PostgreSQL to/from Cloudera. 
sidebar: mydoc_sidebar
permalink: cloudere_postgresql_backups.html
folder: mydoc
published: true
---

## Backing up PostgreSQL

* The PostgreSQL instance used by Cloudera is located on the `cdh-manager` host. 

* The database is installed by the `cloudera-manager-server-db-2` package, and is an postgreSQL database 8.0 listening on port 7432.

* The default root user is `cloudera-scm` and the default root password is located in the `/var/lib/cloudera-scm-server-db/data/generated_password.txt` file:

* The backup script is scheduled to run automatically and store log information in syslog, located in the `/var/log/messages` file. It is also possible to run the backup script manually as root user:
```
sudo su -
~/DumpPSQL.py
```

## Restoring PostgreSQL

To restore PostgreSQL databases or tables, you will need access to the AWS S3 *backups* bucket.

1. Download the tar.xz file (ENV_PSQL_YY-MM-DDUhh_mm_ss.tar.xz) containing PostgreSQL all dump (PSQL.all.out) from AWS S3, with the following command:<br\>
   `aws s3 cp s3://backups/[file] . `<br\>

   To list all files in the bucket use:<br\>
   `aws s3 ls s3://backups`

2. Unpack the tar:<br\>
   `tar -xvJf [file].tar.xz`

   To restore all databases:<br\>
   `~/RestorePSQL.py [PSQL.all.out] all`

   To restore a single database:<br\>
   `~/RestorePSQL.py -LIST=databasename [PSQL.all.out] dbs`

   To restore the specified databases:<br\>
   `~/RestorePSQL.py -LIST=databasename1,...,databasenameN [PSQL.all.out] dbs`

   To reload roles privileges (to create/recreate users, you have to use 'all' command):<br\>
   `~/RestorePSQL.py [PSQL.all.out] users`

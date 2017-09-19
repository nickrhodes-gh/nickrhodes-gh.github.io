---
layout: post
title: MySQL 'Can't find any matching row' replication error
description: >
  Shortly after setting up MySQL replication and testing that it works, it
  stops running transactions on the slave server after a permissions change on
  the main server.

---

Shortly after configuring a basic MySQL replication setup using a clustered
master replicating to a single slave, I hit an issue where the slave database
was stuck in a `Slave_SQL_Running: NO` state. The output from `SHOW SLAVE
STATUS\G` showed that the slave was able to connect into the master server,
however it was not replaying the transactions.

```
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: <master>
                  Master_User: <replication user>
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 320
               Relay_Log_File: mysqld-relay-bin.000019
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000007
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
...
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2013
                Last_IO_Error: Can't find any matching row in the user table'
                on query. Default database: ''. Query: 'grant all privileges
                on <DB>.* to '<USER>'@'<HOST>' identified by '<PASS>'
                Last_SQL_Errno: 0

```

There was a similar error shown in the `mysqld.log`.

```
[ERROR] Slave SQL: Error 'Can't find any matching row in the user table' on
query. Default database: ''. Query: 'grant all privileges on <DB>.* to
'<USER>'@'<HOST>' identified by '<PASS>'', Error_code: 1133
```

I tried restarting the `mysqld` process on the slave server but that had no
effect. I also tried restarting the slave process from within MySQL but again
no change.

The query run on the master server which was causing issues was a `GRANT
PRIVILEGES` query for a basic MySQL user. The query was failing to run on the
slave server even though the correct user was preset in the `mysql.user` table
on the slave. On a whim I ran `FLUSH PRIVILEGES` on the slave server,
restarted the slave service and bingo, the slave processed the query and
caught up with the master server.

Looking back at my steps:
1. Set server-id, log-bin and InnoDB options on the master server and restart
2. Dump the master server databases using: `mysqldump --all-databases
   --master-data > master-full.dump`
3. Set the server-id on the slave server and restart
4. Run the `CHANGE MASTER TO...` query on the slave
5. Import the dump on the slave server
6. Start the slave process with `START SLAVE;`

I have a feeling I should have run the dump import before running `CHANGE
MASTER TO...` and `FLUSH PRIVILEGES;` on the slave. Apart from that, I can't
see where it could have gone wrong.

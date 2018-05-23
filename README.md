Percona Server 5.7 running on ubuntu 16.04 xenial under Vagrant and virtual box.

```sh
vagrant up
```
In order to check fsync speed you can use:
```sh
sysbench --threads=2 --time=60 --db-driver=mysql --mysql-user=root --mysql-password=secret oltp_insert --tables=2 --table_size=100000 prepare
sysbench --threads=2 --time=60 --db-driver=mysql --mysql-user=root --mysql-password=secret oltp_insert --tables=2 --table_size=100000 run
```

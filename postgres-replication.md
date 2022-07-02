# ref:

https://www.howtoforge.com/how-to-setup-postgresql-streaming-replication-with-replication-slots-on-debian-10/

https://www.youtube.com/watch?v=LhPAg583pKc

https://severalnines.com/database-blog/converting-asynchronous-synchronous-replication-postgresql

https://www.claudiokuenzler.com/blog/723/how-to-monitor-postgresql-replication

postgres replication steps 10.21
--------------------------------

servers
-------
  master  	172.104.171.54
  
  slave        192.53.173.168
  
  
  on master
  ---------
  
  
cp /etc/postgresql/10/main/postgresql.conf /etc/postgresql/10/main/postgresql.conf.bak

vim  /etc/postgresql/10/main/postgresql.conf


Find the following line:

#listen_addresses = 'localhost'


change to 

Find the following line:

listen_addresses = '*'



# create replication user
------------------------

create user replicator  WITH  REPLICATION  PASSWORD 'admin@123';


verify user 
----------

\du

# check replication slot on master


SELECT * FROM pg_create_physical_replication_slot('replicator');

cp /etc/postgresql/10/main/pg_hba.conf /etc/postgresql/10/main/pg_hba.conf


vim  /etc/postgresql/10/main/pg_hba.conf

add bwlow line end of the file


host    replication     replicator      192.53.173.168/32       md5 


systemctl restart postgresql.service


on slave
---------

systemctl stop postgresql.service



su - postgres


cp -R /var/lib/postgresql/10/main/  cp -R /var/lib/postgresql/10/main.bak


rm -rf  /var/lib/postgresql/10/main


pg_basebackup -h 172.104.171.54 -D /var/lib/postgresql/10/main -U replicator -p 5432 -v -R -X stream   -c fast --slot=replicator



systemctl start postgresql.service


# create database in master and check replication

# if we create database in slave it throuh readonly mode error



# view replication status master
--------------------------------

select * from pg_stat_replication;
select * from pg_replication_slots;

on slave
-------

select pg_is_in_recovery();


# view postgres status
-----------------------
 psql -x -c "select * from pg_stat_replication;"
 
 # view sync state 
 ----------------
 
 psql -x -c "select sync_state from pg_stat_replication;"
 
 
 # view replication lag in slave server
 --------------------------------------
 
 select pg_is_in_recovery(),pg_is_wal_replay_paused(), pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp();







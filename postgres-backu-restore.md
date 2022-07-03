1.  pg_dump is a nifty utility designed to output a series of SQL statements that describes the schema and data of your database. You can control what goes into your backup by using additional flags.  
    **Backup:** `pg_dump -h localhost -p 5432 -U postgres -d mydb > backup.sql`

    **Restore:** `psql -h localhost -p 5432 -U postgres -d mydb < backup.sql`

    -h is for host.  
    -p is for port.  
    -U is for username.  
    -d is for database.

2.  **Single transaction restore:**  
    you can use –single-transaction in your restore command. It wraps out entire restore operation in one transaction, if something goes wrong it rollbacks all the changes.  
    `psql –single-transaction -h localhost -p 5432 -U postgres -d mydb < backup.sql`

3.  **Compressed Backup:**  
    If your backup is too big, you can use any compression utility to compress it. I prefer gzip.  
    **Backup:** `pg_dump -h localhost -p 5432 -U postgres -d mydb | gzip > backup.gz`

    **Restore:** `gunzip -c backup.gz | psql -h localhost -p 5432 -U postgres -d mydb`

4.  **Split Backup file:**  
    If you are going to email your backup files or transfer them via any medium on internet I will suggest splitting the files into short files. You can use split utility for splitting the files with a size limit. In the example I am usinf 2mb size limit.  
    **Backup:** `pg_dump -h localhost -p 5432 -U postgres -d mydb | split -b 2m – backup.sql`

    **Restore:** `cat backup.sql* | psql -h localhost -p 5432 -U postgres -d mydb`

5.  **Split compressed Backup file:**  
    This is just a combination of point 3 and 4\. We first compress the file then split it instead of splitting the plain file.  
    **Backup:** `pg_dump -h localhost -p 5432 -U postgres -d mydb | gzip | split -b 1m – backup.gz`

    **Restore:** `cat backup.gz* | gunzip | psql -h localhost -p 5432 -U postgres -d mydb`

6.  **Parallel Backup:**  
    You can allow pg_dump to dump the backup data in parallel by including the -j flag. It tells pg_dump the number of tables it can dump in parallel. Parallel backup only works when you use more than one files for writing backup data hence directory. -F d sets the format to directory and -f provides the directory name.  
    **Backup:** `pg_dump -F d -f backup -j 20 -h localhost -p 5432 -U postgres -d mydb`

    **Restore:** `pg_restore -F d -j 20 -h localhost -p 5432 -U postgres -d mydb backup`

7.  **Backup of a specific table:**  
    You can take backup of a specify table by adding -t flag.  
    **Backup:** `pg_dump -h localhost -p 5432 -U postgres -d mydb -t my_table > backup.sql`

    **Restore:** `psql -h localhost -p 5432 -U postgres -d mydb < backup.sql`

8.  **Take Backup of all databases:**  
    pg_dumpall is used to take backup of all of your postgresql database. I think it is just a wrapper around pg_dump. It will ask password for every database.  
    **Backup:** `pg_dumpall -h localhost -p 5432 -U postgres > backup.sql`

    **Restore:** `psql -h localhost -p 5432 -U postgres < backup.sql`

9.  **Custom format backup (-F c):**  
    Keith mentioned in the comment that -F c provides better options at the time of restoring the backup. You can take the backup of whole database and restore only selected tables by using -t flag. It also compresses the backup data for you.All these feature are also provided by the directory format (-F d) too. The main difference between directory and custom format is that custom format generates a single file and directory format generates a directory full of files. A Single file(single stream) gives us many advantages like outputting the backup data over SSH or transferring it to some other service is easy compared to a directory(multiple stream)

    One more difference is that you can not use parallel backup option (-j) with custom format backup. It is obvious since it uses only one stream.

    So how do you take backup using custom format and restore it.

    **Backup:** `pg_dump -F c -h localhost -p 5432 -U postgres -d mydb > backup.dat`

    **Restore:** `pg_restore -F c -h localhost -p 5432 -U postgres -t my_table -d mydb backup.dat`

attach volume and mount to fstab
--------------------------------
after attach volume

fdisk -l


format
------

mkfs.xfs /dev/xvdf


create directory
------------------
mkdir /postgres/data


mount /dev/xvdf /postgres/data


mount to fstab
--------------

blkid


replace uuid yours



UUID=95018470-28e4-488b-9c81-ba4cab6c96a4 /postgres/data xfs defaults 0 2


verify fstab
-----------

mount -a


To change the data directory in PostgreSQL 10.6, you will need to follow these steps:

stpe1 : verify current datadirectory

su - postgres

psql


SHOW data_directory;

/var/lib/postgresql/10/main

stop postgres service


systemctl stop postgresql.service

systemctl status postgresql.service

copy data to new directory inside byobu shell

byobu

sudo rsync -av /var/lib/postgresql/10/main /postgres/data/


Edit the PostgreSQL configuration file postgresql.conf and update the data directory path to the new location:

before take backup

cp /etc/postgresql/10/main/postgresql.conf  /etc/postgresql/10/main/postgresql.conf.orig

Edit and add new data directory

vim /etc/postgresql/10/main/postgresql.conf


data_directory = '/postgres/data/main'


# Reload the systemd configuration:

sudo systemctl daemon-reload

# Start the PostgreSQL service:

sudo systemctl start postgresql


SHOW data_directory;
   data_directory    
---------------------
 /postgres/data/main
(1 row)




# volume mount config in liode

To get started with a new volume, you’ll want to create a filesystem on it:
Create a Filesystem

mkfs.ext4 "/dev/disk/by-id/scsi-0Linode_Volume_postgres-datavolume"


Once the volume has a filesystem, you can create a mountpoint for it:
Create a Mountpoint


mkdir "/mnt/postgres-datavolume"


Then you can mount the new volume:
Mount Volume

mount "/dev/disk/by-id/scsi-0Linode_Volume_postgres-datavolume" "/mnt/postgres-datavolume"


If you want the volume to automatically mount every time your Linode boots, you’ll want to add a line like the following to your /etc/fstab file

/dev/disk/by-id/scsi-0Linode_Volume_postgres-datavolume /mnt/postgres-datavolume ext4 defaults,noatime,nofail 0 2








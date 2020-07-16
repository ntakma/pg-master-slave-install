# INSTALL POSTGRES MASTER - SLAVE
## NETWORK DIAGRAM 
![IMPORT FILE](./infrac.png)
## SYSTEM REQUIREMENT (APPLY ON MASSTER SERVER AND SLAVE SERVER)
- OS: UBUNTU 18.04 LTS 
- RAM: 8GB 
- VCPU: 5vCPU 
- SSD : 70GB 
## INSTALL POSGRESQL (ON THE MASTER SERVER AND SLAVE SERVER)

### B1: Add repo to source list 

```
$ echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main 11" |
sudo tee /etc/apt/sources.list.d/pgsql.list
```

### B2: Now run the following command to add the GPG key of the PostgreSQL package repository:
```
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

### B3: Update the APT package repository cache with the following command:
```
$ sudo apt update
```
### B4: PostgreSQL 11 (DON'T QUESTION WHY ? BECAUSE I LIKE :))  ):
```
$ sudo apt install -y postgresql-11
```
![install postgres](./install-postgres.png)
### B5 Set password **postgres** user 
```
$ sudo passwd postgres
```
Now enter the password. It should be set.\

## ON THE MASTER SERVER 
### B1: Login postgres user
```
$ su - postgres
```
### B2: create a new user replication:
```
$ psql -c "CREATE USER replication REPLICATION LOGIN CONNECTION LIMIT 1 ENCRYPTED PASSWORD 'YOUR_PASSWORD';"
```

### B3: open /etc/postgresql/11/main/pg_hba.conf with vi
```
$ vi /etc/postgresql/11/main/pg_hba.conf
```
### B4: Add the following line to the marked location:
```
host    replication     replication   192.168.80.116/24   md5
```
![update](./modify-pg-hba.png)
### B5:  open the main PostgreSQL configuration file with vi:
```
$ vi /etc/postgresql/11/main/postgresql.conf
```
Now find and change the following settings. If any line is commented out, uncomment it (removing #) as necessary.
```
listen_addresses = 'localhost,192.168.80.193'
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 64
```
### B5:  Restart Postgresql:
```
$ systemctl restart postgresql
```
## ON THE SLAVE SERVER 
### B1: login as postgres user:
```
$ su - postgres
```
### B2: Stop the PostgreSQL service on the pg-slave server:
```
$ systemctl stop postgresql
```
### B4: open /etc/postgresql/11/main/pg_hba.conf with vi:
```
$ nano /etc/postgresql/11/main/pg_hba.conf
```

### B5: Add the following line as you did on the pg-master server:
```
host    replication     replication     192.168.80.193/24   md5
```
![update](./modify-pg-hba.png)

### B6: Open /etc/postgresql/11/main/postgresql.conf with vi:

```
$ vi /etc/postgresql/11/main/postgresql.conf
```
Now find and change the following settings. If any line is commented out, uncomment it (removing #) as necessary.
```
listen_addresses = 'localhost,192.168.80.116'
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 64
```
### B7: Go to your data_directory:
```
$ cd /var/lib/postgresql/11/main
```
Remove everything from that directory:
```
$ rm -rfv *
```
### B8: Copy the data from the pg-master server to the pg-slave server’s data_directory:
```
$ pg_basebackup -h 192.168.80.193 -D /var/lib/postgresql/11/main/ -P -U replication --wal-method=fetch
```
Type in the password for the postgres user of the pg-master server and press <Enter>.
### B9: Create a recovery.conf file in the data_directory with vi:
```
$ vi recovery.conf
```
Now add the following line to it:
```
standby_mode          = 'on'
primary_conninfo      = 'host=192.168.80.137 port=5432 user=replication password=YOUR_PASSWORD'
trigger_file = '/tmp/MasterNow'
```
### B10: Restart postgresql

```
$ systemctl start postgresql
```
## TEST REPICALTION

### On the pg-master server, you can see that the Slave server is detected.

### SQL command for creating users table:
```
CREATE TABLE users (
name VARCHAR(30),
country VARCHAR(2)
);
```

SQL commands to insert dummy data into the users table:
```
INSERT INTO users VALUES('Shahriar', 'BD');
INSERT INTO users VALUES('Shovon', 'BD');
INSERT INTO users VALUES('Kelly', 'US');
INSERT INTO users VALUES('Nina', 'IN');
```
###  From the Slave server pg-slave, login to the PostgreSQL console:
```
$ psql
```
###  try to select the data we just added:
```
select * from users;
```
you can see:
**On server**
 ![update](./testdb-replica.png)
**On slave**
![update](./testdb-replica-slave.png)

## OKE. DONE. SUCCESS INSTALL POSTGRES MASTER-SLAVE. GOODLUCK FOR U 
### FROM TRUNG ANH KMA WITH LOVE <3
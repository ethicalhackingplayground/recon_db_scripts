# recon_db_scripts


### Install MariaDB

```bash
apt -y install mariadb-client
```

```bash
apt -y install mariadb-server
```

### Installing The Database

```bash
mysql_secure_installation
```

### Creating The Database

```sql
mysql -u root -p<password>
```

```sql
create database recon_test;
```

```sql
use recon_test;
```

```sql
create table if not exists subdomains(id INT AUTO_INCREMENT KEY, data VARCHAR(255) NOT NULL);
```

```sql
create table if not exists resolved(id INT AUTO_INCREMENT KEY, data VARCHAR(255) NOT NULL);
```

### Setting Up the Scripts

```bash
mkdir recontest
```


***nano insert_subs.sh***
```bash

#Insert new data into the database

if [[ -z $1 ]] ; then
        echo "Usage: file to insert into database"
        exit
fi

# append $1 to a1
cat $1 | grep -Ev "access.telenet.be|github|myshopify|shopify|facebook|google|microsoft|aliyun|amazoncl
oud|huaweicloud" >> a1  

# append db out to b
mysql recondb -e "select * from subdomains" | awk '{print $2}'  >> b1

# remove db duplicates and output it to file db
awk 'NR==FNR{a[FNR]=$0;next}$0!=a[FNR]{print}' a1 b1 >> db1

if [[ -z $(grep '[^[:space:]]' db1) ]] ; then

        cat a1 | while read line
        do
        mysql recondb -e "insert into subdomains(data) values ('$line')"
        done
else


        # Insert uniq sundomains into db
        cat db1 | while read line
        do
                mysql recondb -e "insert into subdomains(data) values ('$line')"
        done
fi

# clean up
rm a1
rm b1
rm db1
```


***nano insert_resolved.sh***
```bash
#Insert new data into the database

if [[ -z $1 ]] ; then
        echo "Usage: file to insert into database"
        exit
fi


# append $1 to a2
cat $1 | grep -Ev "access.telenet.be|github|myshopify|shopify|facebook|google|microsoft|aliyun|amazoncl
oud|huaweicloud" >> a2

# append db out to b2
mysql recondb -e "select * from resolved" | awk '{print $2}'  >> b2

# remove db duplicates and output it to file db
awk 'NR==FNR{a[FNR]=$0;next}$0!=a[FNR]{print}' a2 b2 >> db2

if [[ -z $(grep '[^[:space:]]' db2) ]] ; then

        cat a2 | while read line
        do

        mysql recondb -e "insert into resolved(data) values ('$line')"
        done
else


        # Insert uniq sundomains into db
        cat db2 | while read line
        do
                mysql recondb -e "insert into resolved(data) values ('$line')"
        done
fi

# clean up
rm a2
rm b2
rm db2
```

```bash
#Insert new data into the database

if [[ -z $1 ]]; then
echo "Usage: "
echo "    ./run_attack.sh resolved"
echo "    ./run_attack.sh subdomains"
exit
fi

if [[ "$1" != "subdomains" ]] && [[ "$1" != "resolved" ]]; then
        exit
fi
mysql recondb -e "select * from $1" | awk '{print $2}' | sort -u
```

### Download All Subdomains From Chaos
https://chaos.projectdiscovery.io/


```bash
find . -name="*.txt" | xargs -I@ bash -c '{ cat "@" >> chaos.txt ; }'
```

```bash
cat /root/recon/chaos/chaos.txt | rev | cut -d '.' -f1,2 | rev | sort -u >> /root/recon/chaos/root.txt
```

***nano run_scans.sh***

```bash
subfinder -dL /root/recon/chaos/roots.txt -silent >> new.txt
./insert_subs.sh new.txt
```

### Creating Cron Rules

```bash
crontab -e
```

### Visualising Cron Rules

https://crontab.guru/#0_1_*_*_1


### Starting the Cron Job
```bash
0 1 * * 1 bash /root/recontest/run_scans.sh
```

```bash
service cron restart
```

#### Done

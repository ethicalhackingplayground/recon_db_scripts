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

cat $1 | grep -vE "access.telenet|github|myshopify|shopify|facebook|google|microsoft|aliyun|amazoncloud|stanford.edu|huaweicloud" >> o1
mysql recondb -e "select * from subdomains" | awk '{print $2}' | grep -v "data" >> o

paste -d@ o o1 | while IFS="@" read -r f1 f2
do
        if [[ "$f2" != "$f1" ]]; then
                mysql recondb -e "insert ignore into subdomains (data) values('$f2')"
        fi
done
rm o
rm o1
```


***nano insert_resolved.sh***
```bash
#Insert new data into the database

if [[ -z $1 ]] ; then
        echo "Usage: file to insert into database"
        exit
fi

cat $1 | grep -vE "access.telenet|github|myshopify|shopify|facebook|google|microsoft|aliyun|amazoncloud|stanford.edu|huaweicloud" >> o1
mysql recondb -e "select * from resolved" | awk '{print $2}' | grep -v "data" >> o

paste -d@ o o1 | while IFS="@" read -r f1 f2
do
        if [[ "$f2" != "$f1" ]]; then
                mysql recondb -e "insert ignore into resolved (data) values('$f2')"
        fi
done
rm o
rm o1
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
cat new.txt | httpx -silent >> resolved.txt
./insert_resolved.sh resolved.txt
```

***nano run_attacks.sh***

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

# Run attacks all the time
while true; do
        # Kill all jobs first
        jobs -p | grep "nuclei" | xargs -n1 pkill -SIGINT -g
        mysql recondb -e "select * from $1" | awk '{print $2}' | nuclei -t /root/nuclei-templates/ -severity critical,high -exclude takeovers -c 200 | notify -silent
done
```

#### Run the attacks in the background

```bash
chmod +x run_attacks.sh; ./run_attacks.sh &
````

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

## FINAL TEST : PART 1

### linux setup

* Create user 'training' on HDFS and Linux for each node
```
# 계정 생성

sudo groupadd skcc
cat /etc/passwd | grep training
sudo useradd training
sudo passwd training
sudo usermod -aG skcc training

# 계정 sudo 권한추가

sudo usermod -aG wheel training

cat /etc/passwd | grep training
cat /etc/group  | grep skcc

# getent 확인

getent passwd training
training:x:1001:1002::/home/training:/bin/bash

getent group skcc
skcc:x:1001:training

getent group wheel
wheel:x:10:centos,training
```
![1](../BigdataEnginerring FinalTest/image/part1/0.PNG)

```
#selinux 활성화 여부 확인

getenforce

````

* /etc/hosts setting

```
sudo vi /etc/hosts

172.31.36.15    util.com util
172.31.47.225   mn.com mn
172.31.37.163   dn1.com dn1
172.31.39.246   dn2.com dn2
172.31.44.39    dn3.com dn3

```

* host 확인

getent hosts

* hostname modification each node
```
sudo hostnamectl set-hostname [노드명]

# hostname 설정 후 재부팅 (*)
reboot

# 변경된 hostname 확인
hostname
```

* linux version 확인
```
grep . /etc/*release
```

* filesystem / disk 확인
```
df -Th
sudo fdisk -l
```

* sshd_config
```
# SSH를 사용하여 EC2 인스턴스에 로그인할 때 키 페어 대신에 암호 로그인을 활성화하여 패스워드 인증 허용

sudo vi /etc/ssh/sshd_config
# PasswordAuthentication -> yes 로 변경 후 저장

# sshd 재시작 및 상태확인
sudo systemctl restart sshd.service
sudo systemctl status sshd.service [not found 인 경우도 있음]

```

* install yum
```
sudo yum update
sudo yum install -y wget

```

* yum list 확인
```
yum repolist
```

* config repository (all node)
```
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
```

* base url 수정 (all node)
```
[centos@util ~]$ sudo vi /etc/yum.repos.d/cloudera-manager.repo
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/
```
* rpm 에 key 추가
```
sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
```

* cloudera install [util]

```
sudo yum install cloudera-manager-daemons cloudera-manager-server
```

* install jdk
```
# 설치 가능한 jdk list 확인
sudo yum list oracle*

sudo yum install -y oracle-j2sdk1.7
```

* java 경로 설정
```
vi ~/.bash_profile

# 아래 두줄 추가
# :$PATH가 뒤에 있어야 java -version 했을 때 정상적인 버전 확인 가능
export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
export PATH=/usr/java/jdk1.7.0_67-cloudera/bin:$PATH

source ~/.bash_profile

# java version 확인
java -version
```

* install JDBC connector
```
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
cd /usr/share/java/
sudo yum install -y mysql-connector-java
```

* install mariaDB

```
# maria db 설치
sudo yum install -y mariadb-server

sudo systemctl enable mariadb
sudo systemctl start mariadb
# mariadb 상태 확인
sudo systemctl status mariadb

# 권한 설정 : 전체 Y 선택
sudo /usr/bin/mysql_secure_installation
```

* create databases for cloudrea sw
```
mysql -u root -p

# db, user 생성
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;
```

* start CM server
```
#모든 Node에 비밀번호 설정 (*중요*)
sudo passwd centos
```

```
#  set CM DB
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user password
# CM server start
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```

* Cloudera Manager start



*
```
# trianing 계정으로 접속

su training

# 폴더 생성 확인

hdfs dfs -ls /user/

drwxrwxrwx   - mapred   hadoop            0 2019-07-19 06:41 /user/history
drwxrwxr-t   - hive     hive              0 2019-07-19 06:44 /user/hive
drwxrwxr-x   - hue      hue               0 2019-07-19 06:45 /user/hue
drwxrwxr-x   - impala   impala            0 2019-07-19 06:51 /user/impala
drwxrwxr-x   - oozie    oozie             0 2019-07-19 06:48 /user/oozie
drwxrwxr-x   - sqoop2   sqoop             0 2019-07-19 06:46 /user/sqoop2
drwxr-xr-x   - training training          0 2019-07-19 06:57 /user/training

```

*
```
# zip파일이 있는 윈도우 경로에서 실행 >> 결과 값 호스트 홈 디렉토리에 업로드
$ scp -i ./skcc.pem authors.sql.zip training@<util 외부 ip>:.
$ scp -i ./skcc.pem posts.sql.zip training@<util 외부 ip>:.

# unzip [util]
sudo yum install -y unzip
unzip authors.sql.zip
unzip posts.sql.zip
```

* training user 권한 추가
```
mysql -u root -p
GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
show grants for 'training'@'%';
```

* create 'test' db
```
CREATE DATABASE test;
use test;
```
* Create 2 table in the test db
```
source ./authors-23-04-2019-02-34-beta.sql
source ./posts23-04-2019 02-44.sql
```

* The imported data should be saved in training’s HDFS home directory
```
sqoop import \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table posts \
--fields-terminated-by "\t" \
--target-dir /user/training/posts
```
```
sqoop import \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table authors \
--fields-terminated-by "\t" \
--target-dir /user/training/authors
```

* Create authors as and external table.
```
# hue 에서 쿼리 수행

CREATE EXTERNAL TABLE authors (
id int,
first_name string,
last_name string,
email string,
birthdate date,
added timestamp
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LOCATION "/user/training/authors/"

```
* Create posts as a managed table.
```
# hue 에서 쿼리 수행

CREATE TABLE posts (
id int,
author_id int,
title string,
description string,
content string,
date date
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LOCATION "/user/training/posts/"
```

* The output of the query should be saved in your HDFS home directory
  (Save it under “results” directory )

```
# hue 에서 쿼리 수행

INSERT OVERWRITE DIRECTORY '/user/training/results'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
SELECT a.id as id
, a.first_name as fname
, a.last_name as lname
, count(b.title) as num_posts
FROM authors a,
posts b
WHERE A.id = b.author_id
GROUP BY a.id, a.first_name, a.last_name;
```

* Create a MySQL table and name it “results”
```
mysql -u root -p

use test;

CREATE TABLE `results` (
`id` int,
`fname` varchar(100),
`lname` varchar(100),
`num_posts` int
)

```
* Finally, export into MySQL the results of your query
```
sqoop export \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table results \
--fields-terminated-by '\t' \
--export-dir hdfs://mn.com:8020/user/training/results
```

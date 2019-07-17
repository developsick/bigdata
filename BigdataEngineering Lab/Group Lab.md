# Team2

* 조장 : 이인선  
![이인선](https://user-images.githubusercontent.com/48976549/61349022-9b478c00-a89d-11e9-8ebf-7afcae0989e1.PNG)
* 조원 : 김중호  
![김중호](https://user-images.githubusercontent.com/48976549/61349020-9aaef580-a89d-11e9-9daa-fc1d4eb8e619.PNG)
* 조원 : 윤산성  
![윤산성](https://user-images.githubusercontent.com/48976549/61349027-9c78b900-a89d-11e9-8c56-dfe7dfab83b7.PNG)
* 조원 : 이선민  
![이선민](https://user-images.githubusercontent.com/48976549/61349018-997dc880-a89d-11e9-82eb-bf50f076a2f4.PNG)


## CM Install Lab

### System Configuration Checks <all nodes>

#### 1. Check vm.swappiness on all your nodes
```
# 시스템이 얼마나 자주 HDD의 SWAP을 사용할 지를 결정함
sudo sysctl vm.swappiness=1
# 재부팅 시에도 변경되지 않도록 설정
sudo sh -c "echo 'vm.swappiness=1'>> /etc/sysctl.conf"
```
![1](../Image/1.JPG)

#### 2. Show the mount attributes of your volume(s)

```
df -Th

sudo fdisk -l
```
![2](../Image/2.JPG)

#### 3. If you have ext-based volumes, list the reserve space setting

#### 4. Disable transparent hugepage support [all nodes!!]
```
# hugepage는 2MB 크기의 메모리 블록
sudo sh -c "echo never > /sys/kernel/mm/transparent_hugepage/defrag"
sudo sh -c "echo never > /sys/kernel/mm/transparent_hugepage/enabled"
sudo sh -c "echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' > /etc/rc.local" sudo sh -c "echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >
/etc/rc.local" sudo cat /etc/rc.local
```

![3](../Image/3.JPG)

#### 5. List your network interface configuration
```
ifconfig
```
![4](../Image/4.JPG)

#### 6. Show that forward and reverse host lookups are correctly resolved
* For /etc/hosts, use getent  
* For DNS, use nslookup  
```
vi /etc/hosts

# 아래 내용 추가
# <private ip> <fqdn>
172.31.13.194 util.com util
172.31.19.171 mn.com mn
172.31.13.237 dn1.com dn1
172.31.15.190 dn2.com dn2
172.31.5.181 dn3.com dn3

getent hosts
```
![5](../Image/5.JPG)
```
sudo yum install bind-utils net-tools -y

nslookup [도메인명]
```
![6](../Image/6.JPG)

* Hostname modification for each node
```
# 각각의 node ( 노드 명은 약어 말고 full name 으로 해주는 게 좋음)
sudo hostnamectl set-hostname <노드명>
예) sudo hostnamectl set-hostname util.com

# 변경된 hostname 확인
hostname
```

#### 7. Show the nscd service is running
```
# nscd(Name Service Cache Daemon)데몬은 가장 일반적인 네임서비스에 대한 캐쉬 기능 제공
# nscd start
sudo yum -y install nscd  
sudo systemctl enable nscd  
sudo systemctl start nscd  
sudo systemctl status nscd  
```
![7](../Image/7.JPG)

#### 8. Show the ntpd service is running
```
# ntpd는 ntp 서비스를 참조해 시스템 클록을 보정하면서 클라이언트에 시간을 제공하는 데몬
# ntpd start
sudo yum -y install ntp sudo chkconfig ntpd on  
sudo systemctl enable ntpd  
sudo systemctl start ntpd

# 확인
ntpq -p
```
![](../Image/8.JPG)
![](../Image/11.JPG)

#### 추가 작업
* sshd_config setting for each node  
SSH를 사용하여 EC2 인스턴스에 로그인할 때 키 페어 대신에 암호 로그인을 활성화
패스워드 인증 허용
( all nodes ! - 진행 안하면 클러스터 안올라감 )
```
sudo vi /etc/ssh/sshd_config
# PasswordAuthentication -> yes 로 변경 후 저장

# sshd 재시작 및 상태확인
sudo systemctl restart sshd.service
sudo systemctl status sshd.service [not found 인 경우도 있음]
```
![](../Image/10.JPG)
![](../Image/9.JPG)

* Install dependencies using yum - all node  
```
sudo yum update
sudo yum install -y wget

# 전체 y 선택
```
![](../Image/12.JPG)

## Cloudera Manager Install Lab
https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_cdh.html  

### Path B install using CM 5.15.x
* all nodes
```
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/

sudo vi /etc/yum.repos.d/cloudera-manager.repo
수정 => baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/
```
![](../Image/13.JPG)

* util
```
# rpm에 key 추가
sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera

# cloudera install
sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```
![](../Image/14.JPG)

#### • Install a supported Oracle JDK on your first node(util)
```
# 설치 가능한 jdk list 확인
sudo yum list oracle*

sudo yum install -y oracle-j2sdk1.7
```
![](../Image/16.JPG)

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
![](../Image/17.JPG)

#### • Install a supported JDBC connector on all nodes
sqoop 사용 시 모든 node 연결 위해 설치
```
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
cd /usr/share/java/
sudo yum install -y mysql-connector-java
```
* mysql connector 설치 확인
![](../Image/18.JPG)

#### • Create the databases and access grants you will need (util)
maria db 설치 시 이미 설치되어 있는 경우는 util node를 교체하여 충돌이 나지 않도록 함  
```
# maria db 설치
sudo yum install -y mariadb-server

sudo systemctl enable mariadb
sudo systemctl start mariadb
# mariadb 상태 확인
sudo systemctl status mariadb
```
* install 성공
![](../Image/15.JPG)
```
# 권한 설정 : 전체 Y 선택
sudo /usr/bin/mysql_secure_installation

```
![](../Image/20.JPG)

#### • Configure Cloudera Manager to connect to the database

* Creating Databases for Cloudera Software
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
![](../Image/21.JPG)
* database 생성 확인  
![](../Image/22.JPG)
#### • Start your Cloudera Manager server -- debug as necessary

```
#모든 Node에 비밀번호 설정 (*중요*)
sudo passwd centos
```
![](../Image/24.JPG)
```
#  set CM DB
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user password
# CM server start
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
![](../Image/23.JPG)

#### • Do not continue until you can browse your CM instance at port 7180

## Cloudera Manager Install Lab
```
<선택>
C:\Windows\System32\drivers\etc > hosts 파일 메모장에서 편집
15.164.82.177 util.com util <- 퍼블릭IP 로 도메인 추가
```
![](../Image/cm_hostname.JPG)
### Install a cluster and deploy CDH
```
http://util.com:7180
admin / admin
- trial 선택
```
* Could not connect to host. 라고 보이는 경우 ssh server가 제대로 설치되었는지 확인
```
# sudo yum install openssh-server
# /sbin/service sshd status
# /sbin/service sshd start
# ssh localhost
```

* Cloudera manager 시작  
![](../Image/25.JPG)
* CDH 클러스터 설치에 대한 호스트 지정  
![](../Image/26.JPG)
* 클러스터 설치 - 리포지토리 선택 (수정사항 없음)
![](../Image/27.JPG)
* 클러스터 설치 - JDK 설치 옵션  
직접 node에 java 설치 진행했기 때문에 uncheck
![](../Image/28.JPG)

#### • Do not use Single User Mode. Do not. Don't do it.
* 클러스터 설치 - 단일모드 비활성화
![](../Image/29.JPG)
* 클러스터 설치 - ssh 로그인 정보 제공
![](../Image/30.JPG)
```
heartbeat 오류시 hostname 재확인
```
![](../Image/31.JPG)

#### • Install CDH using parcels
![](../Image/32.JPG)
```
swapiness, huge pages 에러시 설정 확인
```
![](../Image/33.JPG)
![](../Image/34.JPG)
#### • Deploy only the Core set of CDH services.
```
HDFS, YARN, ZooKeeper 먼저 설치
```
![](../Image/35.JPG)
#### • Deploy three ZooKeeper instances.
![](../Image/36.JPG)
```
생성한 db 계정으로 연결
```
![](../Image/37.JPG)
![](../Image/38.JPG)
![](../Image/39.JPG)
![](../Image/40.JPG)
* 설치 완료 확인
![](../Image/41.JPG)

#### • Ignore any steps in the CM wizard that are marked (Optional)

#### • Install the Data Hub Edition
* Hue 서비스 추가
![](../Image/52.JPG)
![](../Image/53.JPG)
![](../Image/54.JPG)

#### • 추가 서비스 설치

* Hive 서비스 추가
![](../Image/42.JPG)
![](../Image/43.JPG)
![](../Image/44.JPG)
![](../Image/45.JPG)
* Oozie 서비스 추가
![](../Image/46.JPG)
![](../Image/47.JPG)
![](../Image/48.JPG)
![](../Image/49.JPG)
* Sqoop 서비스 추가
![](../Image/50.JPG)
![](../Image/51.JPG)

* 클러스터 세팅 완료
![](../Image/55_cm.JPG)

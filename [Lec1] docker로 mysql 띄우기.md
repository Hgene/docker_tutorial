# Lec1. Docker 환경에 Mysql을 띄워보기
Lecture 0에서 Docker를 설치하는 방법을 알았다면, 이제 docker를 활용해서 가상의 Database를 띄우고 파이썬에서 접속해보도록 하자.    

mysql을 도커환경에 가상으로 띄우는 것은 생각보다 간단하다.   
  1. mysql image(docker)를 불러온다.
  2. 해당 이미지로 도커를 실행하고(백그라운드 실행), 포트 및 유저환경을 셋팅한다.
  3. 터미널에서 접속 테스트 및  `유저` / `Database` / `table` 예제정보 생성
  4. 파이썬에서 접속 테스트.
<br/>


## Step1. 도커 image 확인
먼저 mysql을 활용하기 위해서 참조할만한 docker image파일이 있는지를 확인한다.  

```shell
$ docker images

>> 결과 화면
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
ubuntu       18.04     b280829784a6   11 days ago         56.6MB
```
<br/>

위의 결과처럼 mysql image가 없기 때문에 해당 이미지를 불러온다. 그 후 다시 확인하면 docker에서 관리하는 mysql이미지라 다운로드되어, 아래처럼 확인할 수 있다.
```shell
$ docker pull mysql
$ docker images

>> 결과 화면
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
mysql        latest    5e229524f286   3 days ago          494MB  **이 환경이 새로생기는 것을 확인할 수 있다.
ubuntu       18.04     b280829784a6   11 days ago         56.6MB
```
<br/><br/>



## Step2. 도커 생성

이어서, 해당 image를 통해서 mysql서버를 쉽게 만들수 있다. 아래의 코드는 mysql의 root계정 비밀번호와 서버이름을 셋팅하는 과정이다.  

```shell
#pw: root계정 패스워드, server_name: mysql 서버명
$ sudo docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD={pw} --name {server_name} mysql
# 포트번호를 변경하고 싶다면, 3306:3306 부분을 고쳐야한다. {외부포트}:{내부포트} 순이며, mysql의 내부포트는 3306만 가능하기 때문에,  3307:3306 형태로 바꾸면, 3307로 외부에서 접근이 가능하다.

>> 결과 화면: 서버ID
a726803f3917a62314ac1056b80849392187323aef7cf5842880b860cdb86669
```
<br/> 

이후 실행 중인 docker 컨테이너 리스트를 확인하면 해당 서버가 떠있는 것을 확인할 수 있다.
```shell
$ docker ps     # -a를 추가로 붙이면 전체목록

>> 결과 화면: port가 3306, 서버이름이 'mysql_jin'으로 정상적으로 생성
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                              NAMES
6f73867242b0   mysql          "docker-entrypoint.s…"   24 minutes ago   Up 24 minutes   3306/tcp, 33060/tcp, 0.0.0.0:3306->3306/tcp        mysql_jin
```
<br/><br/> 


## Step3. 터미널에서  접속 테스트
`docker exec` 함수를 이용하여 터미널에서 해당 환경을 실행한다. 이 때 위에 결과로 나온 `CONTAINER ID` 값을 활용하여 컨테이너 실행을 할 수 있다. 
이를 통해서 mysql서버가 정상적으로 생성된 것을 확인할 수 있다.  
```shell
$ sudo docker exec -i -t 6f73867242b0 bash
#(본인PC의 sudo password 입력)

bash-4.4#                                          # 정상실행 화면, 비밀번호는 본인pc의 sudo password
bash-4.4# mysql -u root -p                         # root 계정으로 접속
#(이전에 설정한 루트계정 비밀번호 입력)

mysql>                                             # 정상실행 화면
mysql> show databases;                             # database 리스트 확인
mysql> select user, host from mysql.user;          # 계정정보 확인

```
<br/>

이후 생성한 mysql 서버에 간단한 `유저` / `database` / `table` 을 생성해보도록 하자.  
(1) 유저생성
```mysql
mysql> create user 'hgene'@'%' identified by 'hgene1234';  // '%' 의 의미는 외부에서의 접근을 허용
mysql> select user, host from mysql.user;
# mysql -u hgene -p 로 계정 정상생성 확인가능

>> 결과화면   #이후 계정정보(mysql.user)를 확인하면, hgene 계정이 추가된 것을 확인할 수 있다.
Query OK, 0 rows affected (0.03 sec)
```

(2) `database` 생성
```mysql
mysql> create database jin_db;
mysql> show databases;

>> 결과화면   #이후 데이터베이스 정보(show databases)를 확인하면, jin_db 데이터베이스가 추가된 것을 확인할 수 있다.
Query OK, 0 rows affected (0.03 sec)
```

(3) `table` 생성
```mysql
mysql> CREATE TABLE jin_db.usertable_A (
      id        INT NOT NULL AUTO_INCREMENT,
      user_id     VARCHAR(20),
      user_pw    VARCHAR(20),
      address   VARCHAR(50),
      user_phone    VARCHAR(50),
      PRIMARY KEY(id)
) ENGINE=MYISAM CHARSET=utf8;
    
mysql> show tables from jin_db;
mysql> select * from jin_db.usertable_A;
mysql> desc jin_db.usertable_A;


>> 결과화면   #이후 jin_db 데이터베이스의 테이블 정보(show tables from jin_db;)를 확인하면, usertable_A 테이블이 추가된 것을 확인할 수 있다.
mysql> show tables from jin_db;
+------------------+
| Tables_in_jin_db |
+------------------+
| usertable_A      |
+------------------+

mysql> desc jin_db.usertable_A;
+------------+-------------+------+-----+---------+----------------+
| Field      | Type        | Null | Key | Default | Extra          |
+------------+-------------+------+-----+---------+----------------+
| id         | int         | NO   | PRI | NULL    | auto_increment |
| user_id    | varchar(20) | YES  |     | NULL    |                |
| user_pw    | varchar(20) | YES  |     | NULL    |                |
| address    | varchar(50) | YES  |     | NULL    |                |
| user_phone | varchar(50) | YES  |     | NULL    |                |
+------------+-------------+------+-----+---------+----------------+

mysql> select * from jin_db.usertable_A;
+----+---------+----------+---------+---------------+
| id | user_id | user_pw  | address | user_phone    |
+----+---------+----------+---------+---------------+
|  1 | hj park | 1234qwer | seoul   | 010-1234-5678 |
+----+---------+----------+---------+---------------+
```

(4) 유저계정(hgene)에 데이터베이스 권한부여
```
mysql> grant select, insert, update on jin_db.* to hgene@'%';  // '%'은 외부접속이 가능한 계정을 의미한다. 내부접속은 '%' > 'localhost'로 바꾸면 된다.
mysql> commit; 
mysql> flush privileges; 
```

<br/><br/>



## Step4. Database생성 및 파이썬에서 접속 테스트
이제 docker를 통해서 mysql서버를 정상적으로 생성하였으니 데이터베이스를 임의로 생성하고 테이블을 만든 후에 이를 파이썬에서 직접 연결해보는 작업을 해보자.  
먼저 파이썬에 mysql연결 패키지를 설치하거나 jdbc를 설치한다. 이번에는 편의성을 위해서 `pymysql` 패키지를 설치하고 활용해보도록 하겠다.


```shell
$ pip install pymysql
```

이후에 이전에 생성한 데이터베이스와 유저정보로 접근이 가능하다.
```python 
import pymysql
import pandas as pd

conn = con = pymysql.connect(host='localhost', 
                             user='hgene', 
                             password='hgene1234',
                             db='jin_db')
cursor = conn.cursor()


pd.read_sql('show databases;',conn)

pd.read_sql('show tables from jin_db;',conn)

pd.read_sql('select * from jin_db.usertable_A;',conn)

```

<img width="1197" alt="스크린샷 2022-09-19 오전 12 34 59" src="https://user-images.githubusercontent.com/47958965/190915313-f1010c04-7e5b-4df5-a690-1a9f93a66052.png">


이를 통해서 docker로 mysql 서버를 띄우고, 해당 서버를 파이썬을 통해서 접근해보는 시도를 할 수 있게 되었다.  
해당 방법은 자신이 원하는 가상의 DB 환경을 만들고, DB와 파이썬을 연결하여 모델링을 하는 것을 가능하게 만든다.  

이는 git을 활용한 협업시에 매우 유용하게 활용될 수 있으므로, 한번은 구현해보도록 하자..!

다음에는 힘들게 하나하나 만들었던 앞선과정을 docker-compose.yml과 파일 구성을 통해서 한번에 해결하는 과정을 담아보도록 하겠다.  
해당파일을 만들고나면 docker의 편의성을 크게 느낄 수 있게 될 것이다..!ㅎㅎㅎ


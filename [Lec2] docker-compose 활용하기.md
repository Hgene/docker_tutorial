## Docker-compose 를 활용하여 컨테이너 구성하기  

<img alt="Docker" src="https://www.docker.com/wp-content/uploads/2022/03/horizontal-logo-monochromatic-white.png">
<br/>

### Docker-compose란  
docker-compose란, 다중컨데이터 도커 어플리케이션을 정의하고 실행하는 툴을 말합니다.  
compose는 yaml 파일을 통해서 어플리케이션을 구성하고, 단일 커멘드 명령어를 통해서 구성에 포함된 모든 것을 실행할 수 있어 매우 유용합니다.  

yaml 파일만 구성을 잘하면 되니, 동일한 환경을 구성해야하는 개발자 입장에서 환영하지 않을 이유가 없습니다! 동일한 환경을 매번 실행하고 잇었다면 docker-compose를 반영하는 걸 반드시 고려해보시기 바랍니다.


---
### Docker-compose 실행 프로세스
이렇에 유용한 docker-compose를 사용하기 위해서는 아래의 3가지만 유의하여 구성/실행하면 됩니다.  
  1. Dockerfile을 활용하여 재생산이 필요한 앱 환경을 정의한다.  
  2. `docker-compose.yml` 파일을 정의하여 구현하고 싶은 구체적인 환경을 구성한다.  
  3. `docker compose up` 명령문을 통해 docker-compose를 실행한다. 혹은 standalone(docker-compose binary)를 구성하여 대채할 수도 있습니다.

#### 1. Dockerfile 준비 
혹시 도커환경 구성에 필요한 image가 있다면 따로 준비를 합니다.  
```shell
$ docker pull mysql  # 이번작업에서는 굳이 따로 필요하지 않습니다.
```
<br/><br/>

#### 2. `docker-compose.yml` 파일 구성

두번째로는 `docker-compose.yml` 을 구성해야하는데, 아래는 mysql 실행을 위한 `docker-compose.yml` 파일 및 관련 파일 `.env` 파일을 구성한 예제입니다.
도커의 버전과 mysql 관련 설정 정보를 yml 파일 형태로 적용하여 작성합니다. 세부 적용사항은 아래와 같습니다.  
  1. 도커버전 설정(3 이상 추천)
  2. mysql (도커이미지명) 설정  
    2-1. mysql 환경설정) 포트번호설정  
    2-2. mysql 기초 환경설정(.env)  
    2-3. mysql init시 수행파일 설정(initdb.d)  
    2-4. 도커 내 디렉토리 설정(volumes)

해당 정보를 추가하지 않더라고, 환경 구성 후에 수동으로 직접 쿼리문을 날려서 생성하던 부분인데, 이렇게 미리 작성해 놓음으로써 docker 컨테이너 생성싱 중복으로 환경구성을 하는 번거로움이 사라질 수 있습니다.  
<br/>

*docker-compose.yml 파일구성  
```
#docker-compose.yml

version: "3"                # 도커 버전을 의미합니다.

services:                   # 구성하고 싶은 구체적인 환경
  mysql:
    image: mysql:latest     # mysql의 도커이미지 이름
    ports:
      - 3306:3306           # mysql셋팅1: {외부}:{내부} 포트
      - 33060:33060
    volumes:                # mysql셋팅2: 대체 파일 리스트 {로컬_디렉토리}:{도커_내_디렉토리}
      - ./conf.d:/etc/mysql/conf.d
      - ./data:/var/lib/mysql
      - ./initdb.d:/docker-entrypoint-initdb.d
      - ./dml:/dml
    env_file: .env          # mysql셋팅3: 기초 환경셋팅, root계정 비밀번호 등
    environment:
      TZ: Asia/Seoul        # mysql셋팅4: Time zone
    networks:
      - mysql_network
      
networks:
  mysql_network:

```

*.env 파일구성  
```
.env

MYSQL_HOST=localhost       
MYSQL_PORT=3306               #포트설정
MYSQL_ROOT_PASSWORD=root!     #루트계정비밀번호
MYSQL_DATABASE=hgene_mysql    #데이터베이스명
MYSQL_USER=hgene              #유저생성
MYSQL_PASSWORD=hgene1234      #유저비밀번호 설정
MYSQL_ROOT_HOST: '%'          #외부접속 가능 설정

```
<br/><br/>

#### 3. `docker compose up` 명령문을 통해 docker-compose를 실행
이후 `docker-compose.yml` 파일이 존재하는 디렉토리로 이동하여 아래의 명령어를 입력하면 바로 수행이 가능합니다.
여기서 `-d` 옵션은 백그라운드에서 실행되는 옵션입니다. 
이후 `docker ps` 명령어를 통해서 실행여부를 확인 할 수 있으며, `netstat -ntlp` 명령어로 내부/외부 포트에서 접속가능한(Listen) 상태인지 또한 확인할 수 있습니다.  

*docker ps 실행  
```shell
$ cd {디렉토리}
$ docker-compose up -d
$ docker ps

>> 결과화면
$ docker ps 
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS        PORTS                                              NAMES
49ebf51a7d24   mysql:latest   "docker-entrypoint.s…"   23 hours ago   Up 23 hours   0.0.0.0:3306->3306/tcp, 0.0.0.0:33060->33060/tcp   docker_mysql_mysql_1

```

*netstat으로 포트정보 확인   
```shell
$ netstat -vanp tcp | grep LISTEN # < for MAC | 리눅스 에서는 유사하게 '$ netstat -ntlp' 명령어를 통해서 확인이 가능하다. 

>> 결과화면
tcp46      0      0  *.3306                 *.*                    LISTEN      131072 131072    771      0 0x0100 0x00000006

```

이제부터는 컨테이너가 만들어졌기 때문에, bash접근 혹은 파이썬과 연결작업 등을 통해서 mysql:latest 컨테이터 및 데이터베이스에 접속이 가능합니다.  

알면 알수록 개발환경이 편해질 수 있는 것이 도커환경이네요..!  




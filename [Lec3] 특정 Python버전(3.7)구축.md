이 페이지는 docker 환경내에서 특정파이썬 버전(예시 python3.8)을 구축하는 방법에 대해서 나와 있습니다.

해당 작업은 22년 12월 기점 pooding07 번에서 파이썬 버전이 3.6으로 고정되어 있어 발생하는 문제( import pydbml error )를 해결하기 위해서 작성되었습니다.



## STEP1. Dockerfile 과 requirements.txt 파일을 준비

### [ Dockerfile ]

ubuntu 18.04 는 Python default 버전이 3.6.9이고, python3.8까지만 제공함을 유의합니다.

```
FROM ubuntu:18.04
SHELL ["/bin/bash", "-c"]

ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

RUN apt update
RUN apt-get install -y python3.8
RUN apt-get install -y python3-pip python3.8-dev python3.8-venv build-essential
RUN apt-get install -y python3.8-distutils python3-setuptools
RUN apt-get install -y libsndfile1-dev
RUN ln -s /usr/bin/python3.8 /usr/bin/python
RUN ln -s /usr/bin/pip3 /usr/bin/pip
RUN python -m pip install --upgrade pip

RUN apt-get install -y vim
RUN apt-get install -y wget

COPY requirements.txt /pooding-schema-manager/requirements.txt
WORKDIR /pooding-schema-manager

RUN pip install -r requirements.txt
CMD ["bash"]
```

세부내용은 3.8 버전의 파이썬을 설정하고, default 파이썬을 3.8로 바꾸는 과정입니다.



### [ requirements.txt ]
```
flask==2.2.2
jinja2==3.1.2
sqlalchemy==1.4.4
psycopg2-binary
numpy
pandas
pytz
pydbml
```

<br/>


## STEP2. Dockerfile 로 이미지 생성
```
$ docker build -t {이미지 이름} {Dockerfile 디렉토리}
$ docker build -t pooding-schema-manager:0.1 .
```

### Docker image 확인
```
$ docker images -a
```




## STEP3. Dockerfile 이미지로 도커 컨테이너 실행
```
$ docker run {옵션} --name {컨테이너 이름} {도커이미지 이름} {초기명령어}
$ docker run -it -d --name pooding-schema-manager pooding-schema-manager:0.1 /bin/bash
(정상생성 시) 668af3dac61fa3197d49839ab79d636c35c092a8c0b8e718b33bca5047ed97c2
```

### 실행중인 Docker 컨테이너 확인
```
$ docker ps     # 실행중인 도커 컨테이너 리스트
$ docker ps -a  # 전체 리스트
```




<br/>

## STEP4. docker 컨테이너에 bash로 접속
```
$ docker exec -it {container_id} /bin/bash
$ docker exec -it 668af3dac61f /bin/bash
root@668af3dac61f: /home #                   (정상작동 시)
```






## [Option] Docker 컨테이너 삭제

### STEP1. 컨테이너 정지
```
$ docker stop {container_id}  # container_id 는 "docker ps -a" 로 확인
```

### STEP2. 컨테이너 삭제
```
$ docker rm {container_id}
```

### STEP3. 이미지 삭제
```
$ docker rmi {image_id}       # image_id 는 "docker images -a" 로 확인
```

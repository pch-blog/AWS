# AWS-EC2-SETUP-RHEL7.9.md
~~~
- RHEL 7.9 기준으로 EC2 인스턴스를 생성하고 개발환경 및 AWS 서비스를 사용하기위해 설정하는 과정을 정리한 내용
- Instance type : t3.xlarge
- 불륨 설정
  Size : 300GB, Type : gp3, IOPS : 10000, Throughput : 1000
- OS : RHEL-7.9_HVM-20221027-x86_64-0-Access2-GP2
~~~
<br>

## TIMEZONE 확인 및 변경
```shell
$ timedatectl set-timezone Asia/Seoul
```
<br>

## 계정 추가
```shell
$ useradd -d "홈 디렉토리" -s /bin/bash "계정명"
```
<br>

## 필요 패키지 설치
```shell
$ yum -y install wget gcc gcc-c++sysstat

# NATS.io
$ yum -y install git openssl-devel bzip2-devel

# docker
$ yum -y install yum-utils

# AWS CLI
$ yum -y install less unzip jq

# 한 줄
$ yum -y install wget gcc gcc-c++sysstat git openssl-devel bzip2-devel yum-utils less unzip jq
```

<br>

## AWS CLI 설치
- https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html
```shell
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip; cd aws
$ ./install
$ which aws
$ aws --version
```
<br>

## lz4-devel 설치
- 해당 OS 이미지의 yum 저장소에서 찾지 못하여 직접 설치
```shell
$ wget https://rpmfind.net/linux/centos/7.9.2009/os/x86_64/Packages/lz4-devel-1.8.3-1.el7.x86_64.rpm
$ rpm -i lz4-devel-1.8.3-1.el7.x86_64.rpm
```
<br>

## CMAKE 수동 업데이트
```shell
$ wget https://github.com/Kitware/CMake/releases/download/v2.26.4/cmake-3.26.4.tar.gz
$ tar -zxvf cmake-3.26.5.tar.gz
$ cd cmake-3.26.5
$ ./bootstrap
$ gmakemake
$ make install
```
<br>

## NATS.io C클라이언트 컴파일 및 설치
```shell
$ git clone https://github.com/nats-io/nats.c.git
$ cd nats.c
$ mkdir build; cd build
# cmake 컴파일 옵션은 README에서 확인 필요
$ cmake .. -DNATS_BUILD_WITH_TLS=OFF -DNATS_BUILD_STREAMING=OFF
$ make install
```
<br>

## docker 설치
- https://docs.docker.com/engine/install/centos/
```shell
# add repo
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ vi /etc/yum.repos.d/docker-ce.repo
```shell
# docker-ce.repo 파일에 아래 내용 추가 후 저장 (install 오류로 추가)
[centos-extras]
name=Centos extras - $basearch
baseurl=http://mirror.centos.org/centos/7/extras/x86_64
enabled=1
gpgcheck=1
gpgkey=http://centos.org/keys/RPM-GPG-KEY-CentOS-7
```
```shell
# install docker
$ yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# docker start & reboot auto start
$ systemctl start docker
$ systemctl enable docker

# docker version check
$ docker version
```
<br>

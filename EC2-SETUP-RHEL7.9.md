# EC2 Setup RHEL7.9
~~~
- RHEL 7.9 기준으로 EC2 인스턴스를 생성하고 개발 환경 및 AWS 서비스를 사용하기 위해 설정하고 AMI를 생성하는 과정을 정리한 내용입니다.
- 모든 과정을 진행할 필요 없으며 내용 확인 후 필요한 부분만 진행하세요.
- EC2 인스턴스 정보
  Instance type : t3.xlarge
  Disk Info : Size(300GB), Type(gp3), IOPS(10000), Throughput(1000)
  OS : RHEL-7.9_HVM-20221027-x86_64-0-Access2-GP2
~~~
<br>

## root 계정 비밀번호 설정
```shell
$ sudo passwd root
root 사용자의 비밀 번호 변경 중
새  암호:
새  암호 재입력:
```
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
$ yum -y install wget gcc gcc-c++ sysstat vim

# NATS.io
$ yum -y install git openssl-devel bzip2-devel

# docker
$ yum -y install yum-utils

# AWS CLI
$ yum -y install less unzip jq

# 한 줄
$ yum -y install wget gcc gcc-c++ sysstat vim git openssl-devel bzip2-devel yum-utils less unzip jq
```
<br>

## AWS CLI 설치
- https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html
- ec2-user 계정에서 작업
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
- ec2-user 계정에서 작업
```shell
$ wget https://rpmfind.net/linux/centos/7.9.2009/os/x86_64/Packages/lz4-devel-1.8.3-1.el7.x86_64.rpm
$ rpm -i lz4-devel-1.8.3-1.el7.x86_64.rpm
```
<br>

## CMAKE 수동 업데이트
- ec2-user 계정에서 작업
```shell
$ wget https://github.com/Kitware/CMake/releases/download/v2.26.4/cmake-3.26.4.tar.gz
$ tar -zxvf cmake-3.26.5.tar.gz
$ cd cmake-3.26.5
$ ./bootstrap
$ gmakemake
$ make install
```
<br>

## NATS.io 서버 및 C클라이언트 컴파일 및 설치
- ec2-user 계정에서 작업
- NATS.io 서버 설치
```shell
# RPM 다운로드 및 설치
$ wget https://github.com/nats-io/nats-server/releases/download/v2.10.7/nats-server-v2.10.7-amd64.rpm
$ rpm -i nats-server-v2.10.7-amd64.rpm
# 위치 확인
$ which nats-server
/usr/local/bin/nats-server
```
- NATS.io C 클라이언트 설치
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
- ec2-user 계정에서 작업
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

## EC2 이미지 생성 (AMI:Amazon Machine Image)
- EC2 대쉬보드에서 Instances(인스턴스) -> EC2 선택(체크 박스) -> Ations(작업) -> Image And templates(이미지 및 템플릿) -> Create image(이미지 생성)

![EC2 이미지 생성](./img/EC2-Setup-RHEL7.9-ImageCreate.png)
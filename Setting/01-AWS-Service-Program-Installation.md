# 01-AWS-Service-Program-Installation
- AWS의 서비스를 하기위해 RHEL 기준 각 서비스의 프로그램이나 에이전트, 유틸 설치 과정 정리
<br>

## Table of Contents
- [AWS CLI](#aws-cli)
- [AWS ECS Agent](#aws-ecs-agent)
- [AWS EFS, FSx](#aws-efs-fsx)
	- [EFS](#efs)
	- [FSx Lustre](#fsx-lustre)
	- [EFS 오류1. 패키지 저장소에서 nfs-utils를 찾지 못 하는 경우](#efs-오류1-패키지-저장소에서-nfs-utils를-찾지-못-하는-경우)
<br>

## AWS CLI
- [AWS - 최신 버전의 AWS CLI 설치 또는 업데이트](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html)
- ec2-user 계정에서 작업
```shell
$ sudo yum -y install less unzip jq
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip; cd aws
$ sudo ./install
$ which aws
$ aws --version
```
<br>

## AWS ECS Agent
- [AWS - RHEL9를 ECS에서 사용할 수 있는 방법이 있을까요?](https://repost.aws/questions/QUgn5lMAy3Qfye0cQAVPoLjg/rhel-9-를-ecs에서-사용할-수-있는-방법이-있을까요)
- [AWS - Amazon ECS에서 사용자 지정 AMI를 생성하고 사용하려면 어떻게 해야 하나요?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/ecs-create-custom-AMIs)
- ec2-user 계정에서 작업
- 다운로드 및 설치
```shell
$ curl -o amazon-ecs-init.rpm https://s3.ap-northeast-2.amazonaws.com/amazon-ecs-agent-ap-northeast-2/amazon-ecs-init-latest.x86_64.rpm
$ sudo yum install -y ./amazon-ecs-init.rpm
```
- ECS Agent 서비스 시작
```shell
$ sudo systemctl start ecs
```
- ECS Agent 서비스 상태 확인
```shell
$ systemctl list-units --type=service --state=running | grep ecs\
ecs.service                      loaded active running Amazon Elastic Container Service - container agent
```
- ECS Agent 서비스 중지
```shell
$ sudo systemctl stop ecs
```
- ECS 로그 및 Agent 파일 삭제
```shell
$ sudo rm -rf /var/log/ecs/*
$ sudo rm /var/lib/ecs/data/agent.db
```
<br>

## AWS EFS, FSx
### - EFS
- [다른 Linux 배포판에 Amazon EFS 클라이언트 설치](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/installing-amazon-efs-utils.html#installing-other-distro)
- ec2-user 계정에서 작업
- 기본적으로 EFS를 사용하기 위해서는 <b>nfs-utils, amazon-efs-utils</b>가 설치되어 있어야 합니다.
- amazon-efs-utils 설치
```shell
$ sudo yum -y install git rpm-build
$ git clone https://github.com/aws/efs-utils
$ cd efs-utils
$ sudo make rpm
$ sudo yum -y install ./build/amazon-efs-utils*rpm
```
- EFS를 ECS에서 컨테이너가 마운트하기 위해서 amazon-ecs-volume-plugin을 시작
```shell
$ sudo systemctl start amazon-ecs-volume-plugin
```
- amazon-ecs-volume-plugin 상태 확인
```shell
$ systemctl list-units --type=service --state=running | grep ecs
amazon-ecs-volume-plugin.service loaded active running Amazon Elastic Container Service Volume Plugin
```
- amazon-ecs-volume-plugin 중지
```shell
$ sudo systemctl stop amazon-ecs-volume-plugin
```
- 파일 시스템의 마운트 상태를 CloudWatch 모니터링이 필요한 경우 [botocore 설치](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/install-botocore.html)
- EFS 데이터 암호화를 하려면 [stunnel 설치 및 업그레이드](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/upgrading-stunnel.html)

### - FSx Lustre
- [Lustre 클라이언트 설치](https://docs.aws.amazon.com/ko_kr/fsx/latest/LustreGuide/install-lustre-client.html#lustre-client-rhel)에서 OS의 버전에 따라 설치 과정을 진행합니다.
- ec2-user 계정에서 작업
- Amazon FSx RPM 퍼블릭 키 다운로드 및 가져오기
```shell
$ curl https://fsx-lustre-client-repo-public-keys.s3.amazonaws.com/fsx-rpm-public-key.asc -o /tmp/fsx-rpm-public-key.asc
$ rpm --import /tmp/fsx-rpm-public-key.asc
```
- FSx Lustre 저장소 추가 및 패키지 관리자 업데이트
```shell
$ hostnamectl
# OR
$ cat /etc/redhat-release
```
```shell
# CentOS, RHEL, RockyLinux 9.0 이상
$ sudo curl https://fsx-lustre-client-repo.s3.amazonaws.com/el/9/fsx-lustre-client.repo -o /etc/yum.repos.d/aws-fsx.repo
# CentOS, RHEL 8.2 ~ 8.9, RockyLinux 8.4 ~ 8.9
$ sudo curl https://fsx-lustre-client-repo.s3.amazonaws.com/el/8/fsx-lustre-client.repo -o /etc/yum.repos.d/aws-fsx.repo
# CentOS, RHEL 7.7 ~ 7.9
$ sudo curl https://fsx-lustre-client-repo.s3.amazonaws.com/el/7/fsx-lustre-client.repo -o /etc/yum.repos.d/aws-fsx.repo
```
- 커널 버전 확인 후 저장소 구성파일을 수정해야 합니다.
```shell
$ hostnamectl
# OR
$ uname -r
```
- 각각의 OS 버전벼 설치 과정에 따라 [Lustre 클라이언트 설치](https://docs.aws.amazon.com/ko_kr/fsx/latest/LustreGuide/install-lustre-client.html#lustre-client-rhel)에 수정해야하는 버전을 확인하면 됩니다.
- specific_RHEL_version 값을 정하여 저장소 구성 파일을 수정합니다.
```shell
$ sudo sed -i 's#8#specific_RHEL_version#' /etc/yum.repos.d/aws-fsx.repo
# specific_RHEL_version가 8.9 이면
$ sudo sed -i 's#8#8.9#' /etc/yum.repos.d/aws-fsx.repo
```
- Lustre 클라이언트 설치
```shell
$ sudo yum -y install kmod-lustre-client lustre-client
$ rm -rf /tmp/fsx-rpm-public-key.asc
```
- RPM 퍼블릭 키 삭제
```shell
$ rm -rf /tmp/fsx-rpm-public-key.asc
```

### - EFS 오류1. 패키지 저장소에서 nfs-utils를 찾지 못 하는 경우
- AWS의 RHEL AMI와 도커허브에 잇는 RHEL 컨테이너 이미지의 라이센스 차이로 패키지 저장소 위치가 서로 다르기 때문에 컨테이너 이미지 빌드 시점에 nfs-utils 설치에 문제가 있으며 amazon-efs-utils 설치에도 영향이 있습니다.
- nfs-utils의 의존성 확인하고 yumdownloader를 사용하여 각 패키지 다운로드하여 수동 설치하는 방법으로 진행합니다.
- 패키지의 RPM을 다운로드 받기위해 yum-downloadonly을 설치합니다.
```shell
$ sudo yum install -y yum-downloadonly
```
- nfs-utils 수동 설치에 필요한 RPM 패키지 다운로드 합니다.
```shell
$ mkdir RPM; cd RPM
# 의존성 확인의 패키지 목록이 아니며 컨테이너 이미지 빌드 시점에 나오는 오류 기준으로 뽑은 패키지
$ sudo yumdownloader nfs-utils gssproxy keyutils kmod libevent libnfsidmap \
                     python3-pyyaml quota rpcbind libbasicobjects libcollection \
					 libini_config libref_array libverto-module-base libini_config \
					 libev e2fsprogs quota-nls e2fsprogs-libs fuse-libs libss-1.45.6-5.el8
$ sudo rm -f *i686.rpm; sudo chown $USER *rpm; sudo chgrp $USER *rpm
```
- nfs-utils 의존성 확인 및 의존성 기준 다운로드
```shell
$ sudo yum deplist nfs-utils
$ sudo yumdownloader --resolve nfs-utils
```
- 다운로드 받은 패키지들을 컨테이너 이미지 빌드 시점에 복사하여 설치 후 amazon-efs-utils 설치를 진행하였습니다.
```docker
# nfs-utils 패키지 수동 설치
WORKDIR RPM
COPY $LOCAL_HOME/RPM/*.rpm .
RUN yum -y install ./*rpm
WORKDIR /
RUN rm -rf RPM
```
<br>






# EC2-Instance-Setup-RHEL
- [인스턴스 생성 과정](./01-EC2-Instance-Start.md)을 RHEL의 AMI를 사용하여 인스턴스를 생성 후 설정하는 과정입니다.

- 사용 AMI
	- <b>EC2 -> Images -> AMI Catalog</b> 메뉴에서 검색

	- RHEL-7.9_HVM-20221027-x86_64-0-Access2-GP2 (ami-0de638ae3a9404d04)
	- RHEL-8.9.0_HVM-20240213-x86_64-3-Access2-GP3 (ami-09e760896c8f55abe)

<br>

## Table of Contents
- [기본 설정](#기본-설정)

- [프로그램 설치](#프로그램-설치)

- [개발환경 관련 프로그램 설치](#개발환경-관련-프로그램-설치)

- [AWS 서비스 프로그램 설치](#aws-서비스-프로그램-설치)

<br>

## 기본 설정
- root 계정 비밀번호 설정

    ```shell
    $ sudo passwd root
    root 사용자의 비밀 번호 변경 중
    새  암호:
    새  암호 재입력:
    ```

- TIMEZONE 확인 및 변경
    ```shell
    $ timedatectl set-timezone Asia/Seoul
    ```

- 계정 추가
    ```shell
    $ useradd -d "홈 디렉토리" -s /bin/bash "계정명"
    ```

- 패키지 업데이트 및 설치
    ```shell
    $ sudo yum -y update
    $ sudo yum -y install git wget sysstat strace vim systemd net-tools yum-utils
    ```

<br>

## 프로그램 설치
- [Docker 설치]()

<br>

## 개발환경 관련 프로그램 설치
- [NATS.io C클라이언트 설치](https://github.com/pch-blog/NATS/blob/main/Setting/03-Nats.c-Client-Installation.md)

<br>

## AWS 서비스 프로그램 설치
- [AWS CLI 설치](../01-AWS-Service-Program-Installation.md/#aws-cli)
- [AWS ECS Agent 설치](../01-AWS-Service-Program-Installation.md/#aws-ecs-agent)
- [AWS EFS - amazon-efs-utils 설치](../01-AWS-Service-Program-Installation.md/#efs)
- [AWS FSx Lustre 설치](../01-AWS-Service-Program-Installation.md/#fsx-lustre)

<br>

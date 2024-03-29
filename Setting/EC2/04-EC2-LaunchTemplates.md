# EC2-LaunchTemplates
- 인스턴스 구성을 재사용하기 위한 시작템플릿 생성 과정을 정리합니다.
- 시작템플릿은 제공 받은 AMI나 [직접 생성한 AMI](./03-EC2-AmazonMachineImage.md) 기반으로 인스턴스 구성을 미리 만들서 사용합니다.
- 하나의 시작템플릿은 여러 버전을 저장하여 관리가 가능합니다.

<br>

## Table of Contents
- [Auto Scaling용 시작템플릿 생성](#auto-scaling용-시작템플릿-생성)

<br>

## Auto Scaling용 시작템플릿 생성
- <b>"EC2 Auto Scaling에 사용할 수 있는 템플릿을 설정하는 데 도움이 되는 지침 제공"</b> 체크 후 설정을 시작합니다.
<br>

  ![04-EC2-LaunchTemplates-01](./img/04-EC2-LaunchTemplates-01.png)
</br>

- "내 AMI"에서 미리 [생성한 AMI](./03-EC2-AmazonMachineImage.md)를 선택합니다.
- 사용하려는 용도 기준으로 인스턴스 유형을 재설정합니다.
- SSH 접속용 키 페어를 설정합니다.
- 네트워크 설정은 VPC와 서브넷은 오토스켈링 생성 시점에 결정합니다. 시작템플릿에서는 사용 예정인 VPC의 보안 그룹을 선택합니다.
- 스토리지도 필요한 경우 스토리지 크기도 변경합니다.
- "고급 세부 정보"는 해당 오토스케일링 그룹의 용도에 따라 설정합니다.

## 고급 세부 정보 - 사용자 데이터 설정
- 디렉토리 생성
  ```shell
  sudo mkdir /data
  sudo mkdir /home/ec2-user/data
  ```

- ECS용 시작템플릿인 경우 해당 내용 추가
  ```shell
  #!/bin/bash
  echo ECS_CLUSTER="ECS 클러스터 이름" >> /etc/ecs/ecs.config;
  sudo systemctl start ecs
  sudo systemctl start amazon-ecs-volume-plugin
  ```

- 인스턴스 유형에 인스턴스 스토리지가 있는 경우 - [링크](./01-EC2-Instance-Start.md/#인스턴스-스토리지를-사용하는-경우)

- EFS를 탑재하는 경우
  ```shell
  # "file_system_id"를 실제 파일시스템 ID로 변경합니다.
  # amazon-efs-utils 사용하는 경우
  sudo mount -t efs -o tls "file_system_id":/ efs
  # NFS 클라이언트 사용하는 경우
  sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "file_system_id".efs.ap-northeast-2.amazonaws.com:/ efs
  ```

- FSx Lustre를 탑재하는 경우
  ```shell
  sudo mkdir -p /fsx
  # file_system_dns_name을 실제 파일 시스템의 DNS 이름으로 바꿉니다.
  # mountname을 파일 시스템의 마운트 이름으로 바꿉니다
  sudo mount -t lustre -o relatime,flock "file_system_dns_name"@tcp:/"mountname" /fsx
  ```

  <br>

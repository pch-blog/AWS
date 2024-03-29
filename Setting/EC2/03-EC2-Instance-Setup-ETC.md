# EC2-Instance-Setup-ETC
- OS 기준 설정을 제외한 공통적인 설정 부분을 정리합니다.
<br>

## Table of Contents
- [인스턴스 스토리지를 사용하는 경우](#인스턴스-스토리지를-사용하는-경우)
<br>

## 인스턴스 스토리지를 사용하는 경우
- [AWS EC2 인스턴스 스토리지 설명](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/add-instance-store-volumes.html)

- 인스턴스 시작시 생성, 종료시 같이 삭제되는 스토리지
- EC2 인스턴스 생성 및 시작시 바로 사용가능 하도록 해당 내용을 인스턴스 생성이나 시작 템플릿 작성시
하단에 <b>"고급세부정보 -> 사용자 데이터"</b>에 추가

    ```shell
    $ sudo mkfs -t xfs /dev/nvme1n1
    $ sudo mkdir /data
    $ sudo mount /dev/nvme1n1 /data
    ```

    ![EC2 인스턴스 스토리지 설정](./img/EC2-InstanceStoreSet.png)

<br>
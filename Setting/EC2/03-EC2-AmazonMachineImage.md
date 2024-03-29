# EC2-AmazonMachineImage
- 인스턴스 생성, 설정 후 AMI 생성을 위한 과정을 정리합니다.

<br>

## Table of Contents
- [AMI(Amazon Machine Image) 생성 사전 작업](#amiamazon-machine-image-생성-사전-작업)

- [AMI(Amazon Machine Image) 생성](#amiamazon-machine-image-생성)

<br>

## AMI(Amazon Machine Image) 생성 사전 작업
### Docker를 설치한 경우
- 필요한 경우 docker 서비스를 중지합니다.
    ```shell
    $ sudo systemctl stop docker.socket
    $ sudo systemctl stop docker.service
    ```

### ECS Agent를 설치한 경우
- 서비스 중지
    ```shell
    $ sudo systemctl stop ecs
    $ sudo systemctl stop amazon-ecs-volume-plugin
    ```
- 로그 및 Agent 관련 파일 삭제
    ```shell
    $ sudo rm -rf /etc/ecs/ecs.config
    $ sudo rm -rf /var/log/ecs/*
    $ sudo rm /var/lib/ecs/data/agent.db
    ```

### 불필요한 파일들을 제거를 위한 shell 생성 및 실행
- shell 생성
    ```shell
    $ vi cleanup.sh
    ```
    ```shell
    # 이미지 생성전 불필요한 파일 정리를 위해 내용 추가
    #!/bin/bash
    # Cleanup Function
    function cleanup() {
    FILES=("$@")
    for FILE in "${FILES[@]}"; do
        if [[ -f "$FILE" ]]; then
            echo "Deleting $FILE";
            sudo shred -zuf $FILE;
        fi;
        if [[ -f $FILE ]]; then
            echo "Failed to delete '$FILE'. Failing."
            exit 1
        fi;
    done
    };

    echo "============================ CleanUp Start ============================"
    # Clean up for cloud-init files
    CLOUD_INIT_FILES=(
        "/etc/sudoers.d/90-cloud-init-users"
        "/etc/locale.conf"
        "/var/log/cloud-init.log"
        "/var/log/cloud-init-output.log"
    )

    echo "Cleaning up cloud init files"
    cleanup "${CLOUD_INIT_FILES[@]}"
    if [[ $( sudo find /var/lib/cloud -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting files within /var/lib/cloud/*"
        sudo find /var/lib/cloud -type f -exec shred -zuf {} \;
    fi;

    if [[ $( sudo ls /var/lib/cloud | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/lib/cloud/*"
        sudo rm -rf /var/lib/cloud/* || true
    fi;

    # Clean up for temporary instance files
    INSTANCE_FILES=(
        "/etc/.updated"
        "/etc/aliases.db"
        "/etc/hostname"
        "/var/lib/misc/postfix.aliasesdb-stamp"
        "/var/lib/postfix/master.lock"
        "/var/spool/postfix/pid/master.pid"
        "/var/.updated"
        "/var/cache/yum/x86_64/2/.gpgkeyschecked.yum"
    )

    echo "Cleaning up instance files"
    cleanup "${INSTANCE_FILES[@]}"

    # Clean up for instance log files
    INSTANCE_LOG_FILES=(
        "/var/log/audit/audit.log"
        "/var/log/boot.log"
        "/var/log/dmesg"
        "/var/log/cron"
    )

    echo "Cleaning up instance log files"
    cleanup "${INSTANCE_LOG_FILES[@]}"

    # Clean up for ssm log files
    echo "Cleaning up ssm log files"
    if [[ $( sudo find /var/log/amazon/ssm -type f | sudo wc -l) -gt 0 ]]; then
        echo "Deleting files within /var/log/amazon/ssm/*"
        sudo find /var/log/amazon/ssm -type f -exec shred -zuf {} \;
    fi
    if [[ $( sudo find /var/log/amazon/ssm -type f | sudo wc -l) -gt 0 ]]; then
        echo "Failed to delete /var/log/amazon/ssm"
        exit 1
    fi
    if [[ -d "/var/log/amazon/ssm" ]]; then
        echo "Deleting /var/log/amazon/ssm/*"
        sudo rm -rf /var/log/amazon/ssm
    fi
    if [[ -d "/var/log/amazon/ssm" ]]; then
        echo "Failed to delete /var/log/amazon/ssm"
        exit 1
    fi

    if [[ $( sudo find /var/log/sa/sa* -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/log/sa/sa*"
        sudo shred -zuf /var/log/sa/sa*
    fi
    if [[ $( sudo find /var/log/sa/sa* -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Failed to delete /var/log/sa/sa*"
        exit 1
    fi

    if [[ $( sudo find /var/lib/dhclient/dhclient*.lease -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/lib/dhclient/dhclient*.lease"
        sudo shred -zuf /var/lib/dhclient/dhclient*.lease
    fi
    if [[ $( sudo find /var/lib/dhclient/dhclient*.lease -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Failed to delete /var/lib/dhclient/dhclient*.lease"
        exit 1
    fi

    if [[ $( sudo find /var/tmp -type f | sudo wc -l) -gt 0 ]]; then
        echo "Deleting files within /var/tmp/*"
        sudo find /var/tmp -type f -exec shred -zuf {} \;
    fi
    if [[ $( sudo find /var/tmp -type f | sudo wc -l) -gt 0 ]]; then
        echo "Failed to delete /var/tmp"
        exit 1
    fi
    if [[ $( sudo ls /var/tmp | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/tmp/*"
        sudo rm -rf /var/tmp/*
    fi

    # Shredding is not guaranteed to work well on rolling logs
    if [[ -f "/var/lib/rsyslog/imjournal.state" ]]; then
        echo "Deleting /var/lib/rsyslog/imjournal.state"
        sudo shred -zuf /var/lib/rsyslog/imjournal.state
        sudo rm -f /var/lib/rsyslog/imjournal.state
    fi

    if [[ $( sudo ls /var/log/journal/ | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/log/journal/*"
        sudo find /var/log/journal/ -type f -exec shred -zuf {} \;
        sudo rm -rf /var/log/journal/*
    fi

    # ECS log files
    echo "Cleaning up ECS log files"
    if [[ $( sudo find /var/log/ecs/* -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Deleting /var/log/ecs/*"
        sudo shred -zuf /var/log/ecs/*
    fi
    if [[ $( sudo find /var/log/ecs/* -type f | sudo wc -l ) -gt 0 ]]; then
        echo "Failed to delete /var/log/ecs/*"
        exit 1
        fi

    # touch
    sudo touch /etc/machine-id
    echo "============================ CleanUp  End  ============================"
    ```

- shell 실행

    ```shell
    $ chmod 755 cleanup.sh
    $ ./cleanup.sh
    $ rm cleanup.sh
    ```
<br>

## AMI(Amazon Machine Image) 생성
- AMI 생성 사전 작업 후 <b>권한 문제 및 기타 문제 방지를 위해 설정이 완료된 상태에서 인스턴스 재부팅 진행</b>
- "EC2 -> 인스턴스(Instances) -> EC2 선택(체크 박스) -> 작억(Ations) -> 이미지 및 템플릿(Image And templates) -> 이미지 생성(Create image)"
<br><br>
![EC2 이미지 생성](./img/03-EC2-AmazonMachineImage-01.png)

<br>
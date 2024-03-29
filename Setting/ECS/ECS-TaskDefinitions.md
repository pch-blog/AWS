# ECS-TaskDefinitions
- ECS의 클러스터가 서비스 시작에 사용하는 태스크를 정의하는 과정을 정리합니다.

- [ECS 작업 정의 설명서](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definitions.html)

- [ECS 태스크 정의 파라미터](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html)

<br>

## Table of Contents
- [태스크 정의 확인사항](#태스크-정의-확인사항)

- [JSON 수정으로만 구성 가능한 파라미터](#json-수정으로만-구성-가능한-파라미터)
	- [privileged 모드 추가](#privileged-모드-추가)

	- [pid 모드 추가](#pid-모드-추가)
	- [ipc 모드 추가](#ipc-모드-추가)

<br>

## 태스크 정의 확인사항
- 네트워크 모드를 awsvpc로 사용하면 해당 서브넷이 "인터넷 게이트웨이"를 사용하더라도 컨테이너에 퍼블릭IP 할당이 안됩니다.

- awsvpc모드는 태스크당 ENI 하나씩 가지며, EC2의 인스턴스 기본 인터페이스를 제외한 나머지 ENI 수 만큼 태스크가 실행 가능합니다.<br>인스턴스 유형 마다 ENI의 제한이 있으며 <b>"AWSVPC 트렁킹"을 활성화하면 더 많은 ENI 사용 가능합니다.</b> ([탄력적 네트워크 인터페이스 트렁킹](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/container-instance-eni.html?icmpid=docs_ecs_hp_account_settings))
- 태스크 크기(Task size)에서 vCPU, 메모리를 설정하면 컨테이너의 리소스 할당도 태스크 크기 내에서만 지정 가능하기 때문에 태스크의 vCPU, 메모리를 빈 칸으로 정의해야 EC2의 vCPU, 메모리를 컨테이너에 원활하게 할당 가능합니다.<br>(태스크 크기에서 10vCPU가 최대, 메모리는 vCPU 값에 따라 다름)

<br>

## JSON 수정으로만 구성 가능한 파라미터
- UI로 태스크 정의시 나오지 않는 메뉴 같은 경우 JSON 파일을 직접 수정해야 합니다.

### privileged 모드 추가
- [ECS 개발자가이드 - 태스크 정의 파라미터 - 보안 - privileged](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_security)

- containerDefinitions 영역에 privileged 모드 추가
```json
"containerDefinitions": [
	...,
	privileged : "true",
	...,
]
```

### pid 모드 추가
- [ECS 개발자가이드 - 태스크 정의 파라미터 - PID모드](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#task_definition_pidmode)

- AWS의 태스크 정의시 PID 모드를 선택하여 HOTS, TASK 모드 선택가능

- containerDefinitions 영역에 pidMode 모드 추가
```json
"containerDefinitions": [
	...,
	pidMode : "host" | "task"
	...,
]
```

- HOST에서 표시되는 PID와 컨테이너 내부 PID가 다름

- kill 명령어 결과
``` shell
# HOST
$ pt -ef | grep "PROCESS NAME"+
$ docker exec "CONTAINER ID" kill -15 "PID"
kill: sending signal to "PID" failed: No such process
```

- 프로세스 이름을 사용하는 pkill 명령어 실행 가능
``` shell
# HOST
$ docker exec "CONTAINER ID" pkill "PROCESS NAME"
```

### ipc 모드 추가
- [ECS 개발자가이드 - 태스크 정의 파라미터 - IPC모드](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#task_definition_ipcmode)

- containerDefinitions 영역에 ipcMode 모드 추가
```json
"containerDefinitions": [
	...,
	ipcMode : "host" | "task"
	...,
]
```

- host IPC 모드를 사용하는 태스크의 경우, IPC 네임스페이스 관련 systemControls는 지원 안함





# ECS-TaskDefinitions
- ECS의 클러스터가 서비스 시작에 사용할 태스크를 정의하는 과정 정리
- JSON 파일 수정으로 태스크 정의를 작성하는 방법 정리
<br>

## Table of Contents
- [JSON을 통한 구성(Configure via JSON)](#json을-통한-구성configure-via-json)
	- [privileged 모드 추가](#privileged-모드-추가)
	- [pid 모드 추가](#pid-모드-추가)
	- [ipc 모드 추가](#ipc-모드-추가)

<br>

## JSON을 통한 구성(Configure via JSON)
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





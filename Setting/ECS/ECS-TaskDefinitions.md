# ECS-TaskDefinitions

## privileged 사용
- [AWS - ECS 개발자가이드 - 태스크 정의 파라미터 - 보안 - privileged](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_security)
- privileged는 태스크 정의시 <b>JSON을 통한 구성(Configure via JSON)</b>로 직접 입력하여 설정
``` shell
privileged : "true"
```
- OS이미지 기반으로 컨테이너 구동시 /sbin/init, systemd를 사용하기 위해 옵션 설정
<br>

## PID MODE
- [AWS - ECS 개발자가이드 - 태스크 정의 파라미터 - PID모드](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#task_definition_pidmode)
- PID모드는 태스크 정의시 <b>JSON을 통한 구성(Configure via JSON)</b>로 직접 입력하여 설정
``` shell
pidMode : "host"
pidMode : "task"
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
- AWS의 태스크 정의시 PID 모드를 선택하여 HOTS, TASK 모드 선택가능
<br>

## IPC MODE
- [AWS - ECS 개발자가이드 - 태스크 정의 파라미터 - IPC모드](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/task_definition_parameters.html#task_definition_ipcmode)
- IPC모드는 태스크 정의시 <b>JSON을 통한 구성(Configure via JSON)</b>로 직접 입력하여 설정
``` shell
ipcMode : "host"
ipcMode : "task"
```
- host IPC 모드를 사용하는 태스크의 경우, IPC 네임스페이스 관련 systemControls는 지원 안함
<br>




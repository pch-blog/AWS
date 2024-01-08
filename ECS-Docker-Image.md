# ECS Dokcer Image
~~~
- 관련 AWS 서비스 : ECS, EKS, ECR, EC2
- 컨테이너 이미지 빌드 및 관리에 필요한 과정들을 정리한 내용입니다.
~~~
<br>

## Docker Image 정보 확인 및 태그 관리
- 이미지를 받은 후 이미지 ID 확인
```shell
$ docker pull redhat/ubi8
$ docker images
REPOSITORY    TAG        IMAGE ID       CREATED        SIZE
redhat/ubi8   latest     86b358a425da   2 months ago   205MB
```
- 이미지 정보 확인
```shell
# docker inspect 이미지ID
$ docker inspect 86b358a425da | grep "release\|version"
                "io.buildah.version": "1.29.0",
                "release": "1028",
                "summary": "Provides the latest release of Red Hat Universal Base Image 8.",
                "version": "8.9"
```
- 태그 변경 및 이미지 삭제 후 Pull
```shell
# 태그 변경
$ docker image tag redhat/ubi8:latest redhat/ubi8:8.9-1028

# 이미지 삭제 및 Pull
$ docker rmi 86b358a425da
$ docker pull redhat/ubi8:8.9-1028
```
<br>

## RHEL 7.9 Docker Image 다운로드
- DockerHub에 없음
- https://catalog.redhat.com/software/containers/ubi7/ubi/5c3592dcd70cc534b3a37814?architecture=amd64&tag=all
```shell
$ docker pull registry.access.redhat.com/ubi7/ubi:7.9-1255
$ docker image tag registry.access.redhat.com/ubi7/ubi:7.9-1255 redhat/ubi7:7.9-1255
```
```shell
$ docker images
REPOSITORY                            TAG        IMAGE ID       CREATED       SIZE
registry.access.redhat.com/ubi7/ubi   7.9-1255   3e42ff62a586   4 weeks ago   208MB
redhat/ubi7                           7.9-1255   3e42ff62a586   4 weeks ago   208MB
```
<br>

## RHEL 8.9 Docker Image 다운로드
```shell
$ docker pull redhat/ubi8:8.9-1028
```
```shell
$ docker images
REPOSITORY    TAG        IMAGE ID       CREATED        SIZE
redhat/ubi8   8.9-1028   86b358a425da   2 months ago   205MB
```
<br>

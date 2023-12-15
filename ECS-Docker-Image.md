# ECS Dokcer Image
~~~
- 관련 AWS 서비스 : ECS, EKS, ECR, EC2
- 컨테이너 이미지 빌드를 하기 위해 필요한 과정들을 정리한 내용입니다.
~~~
<br>

## RHEL 7.9 Docker Image 다운로드
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
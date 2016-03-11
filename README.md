# Start Here
## 개요
이 문서는 클라우드 기반의 빅데이터, 리얼타임 프로세싱을 수행하기 위한 소프트웨어의 관리, 개발, 실행 환경을 공유하기 위한 것이며, 아래의 사항들을 다루고 있다.
* GitLab CE - Git 기반 소스 공유 및 버전관리 소프트웨어
* Sonatype Nexus - Maven Central의 사내 proxy 및 사내 라이브러리 공유 저장소 관리 소프트웨어
* Docker Registry - Docker 이미지 저장소
* CoreOS Systems - Docker들을 운영하기 위한 시스템들

## GitLab CE
### 소개
[Github.com](https://github.com)과 유사한 분산 버전관리시스템을 사내에서 운영하기 위한 소프트웨어이다.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| gitlab.obzen.com | http://gitlab.obzen.com:8085|

## Sonatype Nexus
### 소개
[Maven Central](https://repo1.maven.org/maven2/)의 역할을 사내에 옮겨놓은 것이다. 주요 역할은
* Maven Central의 proxy/caching - Maven에서 관리되고 있는 라이브러리들을 캐싱함으로써, 반복적인 다운로드를 피할 수 있다.
* 라이브러리의 퍼블리싱 - Java jar들과 같은 라이브러리들을 외부 시스템이 아니라 사내의 시스템에 퍼블리싱함으로써 효과적으로 공유할 수 있도록 해준다.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| nexus.obzen.com | http://nexus.obzen.com:8081|

## Docker Registry
### 소개
[Docker Hub](https://hub.docker.com)의 역할을 사내에 옮겨놓은 것이다. 주요 역할은 docker 이미지의 관리.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| nexus.obzen.com | http://nexus.obzen.com:8081|

## CoreOS Systems
### 소개
클라우드 환경을 타겟으로 한 리눅스 OS이며, fleet을 통한 docker들의 운영이 주된 역할이다.
### 시스템 정보
| IP Address | Host Name | SSH Port |
|------------|-----------|-----|
|192.168.10.82|t-core-02 | 22|
|192.168.10.83|t-core-02 | 22|
|192.168.10.84|t-core-03 | 22|
### 이용방법
fleetctl로 docker들을 시작,중지, 모니터링하거나, CoreOS에 로긴하기 위해서는 사용자의 Pulbic RSA Key가 미리 시스템에 입력되어 있어야 한다. 따라서 이용자가 Key를 생성하여 CoreOS 관리자에게 전달하여야 한다.<br>
Key 생성방법은 https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2 을 참고한다.<br>
생성된 파일들 중, id_rsa.pub가 자신의 Public RSA Key 파일이며, 이를 관리자에게 전달하면 된다.
### CoreOS에 Public RSA Key의 추가
이미 로긴 권한을 가진 사용자가, CoreOS 시스템들에 로긴하여 다음과 같이 작업하면 된다.
```
sudo vi /var/lib/coreos-install/user_data
```

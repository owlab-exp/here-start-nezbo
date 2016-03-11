# Start Here
## 개요
이 문서는 클라우드 기반의 빅데이터, 리얼타임 프로세싱을 수행하기 위한 소프트웨어의 관리, 개발, 실행 환경을 공유하기 위한 것이며, 아래의 사항들을 다루고 있다.
* GitLab CE - Git 기반 소스 공유 및 버전관리 소프트웨어
* Sonatype Nexus - Maven Central의 사내 proxy 및 사내 라이브러리 공유 저장소 관리 소프트웨어
* Docker Registry - Docker 이미지 저장소
* CoreOS Systems - Docker들을 운영하기 위한 시스템들

이 중, GitLab CE, Synatype Nexus, Docker Registry는 현재 docker로 운영되고 있다. <br>
Docker를 이용하여 이들을 다른 시스템에서 운영하기 위해서는 [http://gitlab.obzen.com:8085/lab/docker-scripts](http://gitlab.obzen.com:8085/lab/docker-scripts)를 참조하도록 한다.<br>

**중요사항**: 이하의 본문에 기술된 **IP Address**들과 그에 대응하는 **Host Name**들은 DNS 서버에 등록되어 있지 않으므로, /etc/hosts (윈도우즈의 경우 다른 파일)에 직접 추가하는 것이 좋다.
## GitLab CE
### 소개
[Github.com](https://github.com)과 유사한 분산 버전관리시스템을 사내에서 운영하기 위한 소프트웨어이다.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| gitlab.obzen.com | http://gitlab.obzen.com:8085|
### 이용방법
[http://gitlab.obzen.com:8085](http://gitlab.obzen.com:8085)에 접속하여 사용자 ID를 생성한다. 이메일 시스템과 연계가 이루어지지 않았으므로 관리자들 중의 누군가가 activation을 해줄 것이며, 이후 프로젝트를 생성할 수 있게 된다.

## Sonatype Nexus
### 소개
[Maven Central](https://repo1.maven.org/maven2/)의 역할을 사내에 옮겨놓은 것이다. 주요 역할은
* Maven Central의 proxy/caching - Maven에서 관리되고 있는 라이브러리들을 캐싱함으로써, 반복적인 다운로드를 피할 수 있다.
* 라이브러리의 퍼블리싱 - Java jar들과 같은 라이브러리들을 외부 시스템이 아니라 사내의 시스템에 퍼블리싱함으로써 효과적으로 공유할 수 있도록 해준다.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| nexus.obzen.com | http://nexus.obzen.com:8081|

### 이용방법
#### Maven Proxy
Gradle의 예를 들면, build.gradle에 Maven 리포지토리를 다음과 같이 지정하게 된다.
```
...
    repositories {
        mavenCentral()
    }
...
```
이 경우에, project에서 참조하는 jar들을 Maven Repository에서 찾고, 다운로드하게 된다.<br>
이것을 아래와 같이 변경한다.
```
...
    repositories {
        maven {
            url 'http://nexus.obzen.com:8081/content/repositories/central/'
        }
    }
...
```
이제 새로 빌드를 하는 경우에 Gradle은 위의 URL로 주어진 리포지토리에서 라이브러리를 찾게 된다. 만일 Nexus에 캐싱되어 있는 라이브러리가 있다면, 곧바로 다운로드를 할 것이다. 하지만, Nexus에 없는 것이라면, Nexus가 먼저 Maven Central에서 라이브러리를 다운로드를 한 후에 클라이언트에 전달할 것이다. 이후에 다른 클라이언트가 동일한 라이브러리를 요청한다면 Maven Central에 다시 접속하지 않고도 서비스를 할 수 있게 된다.
#### 사내 라이브러리의 다운로드
Maven에 있는 라이브러리가 아니라, 사내에서 개발되고 Nexus에 퍼클리싱된 라이브러리를 프로젝트에서 참조해야 한다몀, 다음과 같이 리포지토리 URL들을 추가한다.
```
...
    repositories {
        // Maven Central Proxy
        maven {
            url 'http://nexus.obzen.com:8081/content/repositories/central/'
        }
        // Obzen Releases
        maven {
            url 'http://nexus.obzen.com:8081/content/repositories/releases/'
        }
        // Obzen Snapshots
        maven {
            url 'http://nexus.obzen.com:8081/content/repositories/snapshots/'
        }
    }
...
```
#### 사내 라이브러리의 퍼블리싱
Nexus에 퍼블리싱하는 작업은 Maven에 퍼블리싱하는 것과 같은 설정으로 이루어지며, 다만 url들만 달라질 뿐이다.<br>
https://docs.gradle.org/current/userguide/maven_plugin.html#uploading_to_maven_repositories 를 참조하면 일반적인 사항에 대해 알 수 있을 것이다.<br>
또는,<br>
gitlab.obzen.com의 프로젝트들 중 [obzen-cep](http://gitlab.obzen.com:8085/lab/obzen-cep)의 build.gradle을 참조하여도 된다.<br>
만일 Maven 프로젝트에서 퍼블리싱하는 방법을 알고 싶다면, gitlab.obzen.com의 [siddhi](http://gitlab.obzen.com:8085/lab/siddhi)를 참조하면 된다.

## Docker Registry
### 소개
[Docker Hub](https://hub.docker.com)의 역할을 사내에 옮겨놓은 것이다. 주요 역할은 docker 이미지의 퍼블리싱 및 공유이다.

### 시스템 정보
| IP Address | Host Name | URL |
|------------|-----------|-----|
|192.168.10.81| obzen-reg | http://obzen-reg:5000|

### 이용방법
Docker engine으로 하여금 Docker Hub가 아닌, 위의 사내 레지스트리를 이용할 수 있도록 하려면, **/etc/default/docker** 파일(Ubuntu 기준이며, 다른 OS 혹은 리눅스 배포판에서는 다른 위치의 파일일 수 있음)에 다음의 항목을 추가하여야 한다.
```
DOCKER_OPTS="--insecure-registry=obzen-reg:5000"
```
이후 docker daemon을 재시작하면 된다. 설정이 적용되었는지는 아래와 같이 확인한다.
```
ps -ef | grep docker
```
의 결과가 아래와 같이 나타나면 된다.
```
root  1635     1  0 10:04 ?        00:00:07 /usr/bin/docker daemon --insecure-registry=obzen-reg:5000
```
이후, 이름이 **obzen-reg:5000/** 으로 시작하는 docker 이미지들을 빌드하여 퍼블리싱하거나, 혹은 다운로드 받을 수 있게 된다.

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
fleetctl로 docker들을 시작,중지, 모니터링하거나, CoreOS에 로긴하기 위해서는 사용자의 Pulbic RSA Key가 미리 시스템에 입력되어 있어야 한다.<br>
따라서 이용자가 Key를 생성하여 CoreOS 관리자에게 전달하고, 관리자가 서버들에 입력하는 작업이 이루어져야만 한다.<br>

Key 생성방법은 https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2 을 참고한다.<br>
생성된 파일들 중, id_rsa.pub가 자신의 Public RSA Key 파일이며, 이를 관리자에게 전달하면 된다.<br>

관리자가 이용자의 credential 입력작업을 완료하였다면, 사용자는 아래와 같은 셋팅을 통해 fleetctl을 이용, docker들을 시작하거나 모니터링할 수 있을 것이다.
```
export FLEETCTL_TUNNEL=192.168.10.82:22
```
만일, 위 시스템들에 직업 로긴하여야 한다면, 아래처럼 core 유저로 로긴하면 된다.
```
ssh core@192.168.10.82
```

### CoreOS에 Public RSA Key의 추가
이미 로긴 권한을 가진 사용자가, 새로운 사용자가 접근할 필요가 있는 모든 CoreOS 시스템들에 로긴하여 다음과 같이 작업하면 된다.<br>

```
sudo vi /var/lib/coreos-install/user_data
```
user_data 파일에는 CoreOS를 설정하기 위한 모든 정보가 포함되어 있다. 이 중, **ssh-authorized-keys:** 섹션을 찾는다. 아래와 같은 부분이 나타날 것이다.
```
ssh-authorized-keys:
   - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCvMTJ19zoyZc56IUAi5L+zWKOgcbe0KnFUviFAVd+tmb2LCDTu5DOOqkhgXeB7vvs1HgVbSmVNgk34CZpj70E65T9Hdmv/oVP5GuC1iWnPHx4URFUvoZ5DVZhwdMTCBcU0KbRHZtYf1xPkeWRlk978Vq6+WukGZKQotA2ec/r9k8Ji+HElRq2kQsMBXhtQ/dm+Bh+gaybNCC0psMxdEETB3XoYGh10HzDpr8cbuX11rhfTFdg1CbMdiWzhFX19mrBrTlX1Cq85ceoKTupQjo/R+kNVNpuoUSFE94B+wQBXuWwl3C7VIyKcqfoqDHBl0z5YyzQxgDi7pQzsC213tsNx ernest@ernest-dev
```
여기에 새로운 id_rsa.pub의 내용을 추가한다. 추가하면, 다음과 같이 보일 것이다.

```
ssh-authorized-keys:
   - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCvMTJ19zoyZc56IUAi5L+zWKOgcbe0KnFUviFAVd+tmb2LCDTu5DOOqkhgXeB7vvs1HgVbSmVNgk34CZpj70E65T9Hdmv/oVP5GuC1iWnPHx4URFUvoZ5DVZhwdMTCBcU0KbRHZtYf1xPkeWRlk978Vq6+WukGZKQotA2ec/r9k8Ji+HElRq2kQsMBXhtQ/dm+Bh+gaybNCC0psMxdEETB3XoYGh10HzDpr8cbuX11rhfTFdg1CbMdiWzhFX19mrBrTlX1Cq85ceoKTupQjo/R+kNVNpuoUSFE94B+wQBXuWwl3C7VIyKcqfoqDHBl0z5YyzQxgDi7pQzsC213tsNx ernest@ernest-dev
   - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDa6kEwL057/FiJpnxbrT+pyxPclBjOi7WXDPjajXn4nDN7YD2JA6M/49dm53K+wCdAZZnVNpXa5GrWiSCQyBpUpnUBPdPq5qGS/+1rCKlIrEHuofWleQWaRTMecP9JQozo1rHxa3oKGMei+NIRQMabiqjhMeqyUZaDS25C26QWtcvVGf++wekt8l8ulm3Y/bfoxofV1/EbWAmeHkJHCrcFyIfwAacApwEtqK3/oVs5lcCJKkY3PeL7bNozlqBsY7oC854GYybdybZjazMR99SHPZAMkR6HcLhQp2hn4vmGOr7vA94em9hWAgeg8vPkeGqbsyR3qIv7QqevqULJG0iT inykang@inykang-System
```
CoreOS를 다시 시작한다.
```
sudo shutdown -r now
```

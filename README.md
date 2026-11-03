# AI/SW 개발 워크스테이션 구축

> 대전 코디세이 AI 올인원 입학연수과정 – 미션 1

> ⚠️ 작성 메모: `TODO` 표시된 부분은 아직 기록이 없는 항목이에요. 결과를 채운 뒤 이 줄과 TODO 표시를 지우세요.

## 1. 프로젝트 개요

터미널, Docker, Git/GitHub를 직접 설치·설정하여 AI/SW 개발을 위한 작업 환경을 구축한다.

- 터미널 기본 조작과 파일 권한 설정
- Docker 설치 확인 및 컨테이너 운영
- Dockerfile로 웹 서버 이미지를 만들고 포트 매핑으로 접속
- 바인드 마운트와 볼륨으로 데이터 연결·영속성 확인
- Git 설정 및 GitHub / VS Code 연동

## 2. 실행 환경

| 항목 | 내용 |
|---|---|
| OS | Microsoft Windows 11 Pro (10.0.26300, Build 26300) |
| 쉘 / 터미널 | Windows PowerShell 5.1.26100.9711, Git Bash (UCRT64), VS Code 통합 터미널 |
| Docker | Docker version 29.9.2, build a82e528 (Docker Desktop 4.97.0, WSL2 기반) |
| Git | git version 2.56.0.windows.4 |
| 에디터 | Visual Studio Code |

<details>
<summary>확인 명령 및 출력</summary>

```
PS C:\Users\User\webproject> git --version
git version 2.56.0.windows.4
PS C:\Users\User\webproject> systeminfo | findstr /B /C:"OS"
OS ??:                 Microsoft Windows 11 Pro
OS ??:                 10.0.26300 N/A ?? 26300
OS ????:             Microsoft Corporation
...
PS C:\Users\User\webproject> $PSVersionTable.PSVersion

Major  Minor  Build  Revision
-----  -----  -----  --------
5      1      26100  9711
```
- `systeminfo`의 한글 항목명은 터미널 인코딩 문제로 `??`로 표시됨 (값은 정상)

</details>

## 3. 수행 항목 체크리스트

| 항목 | 상태 | 확인 위치 |
|---|---|---|
| 터미널 기본 조작 | ✅ | [4. 터미널 조작 로그](#4-터미널-조작-로그) |
| 파일 권한 설정 (chmod) | ✅ | [4-2. 권한 설정](#4-2-파일-권한-설정) |
| Docker 설치 및 점검 | ✅ | [5. Docker 설치·점검](#5-docker-설치점검) |
| Dockerfile 웹 서버 | ✅ | [6. Dockerfile 웹 서버](#6-dockerfile-기반-웹-서버) |
| 포트 매핑 접속 | ✅ | [6-4. 포트 매핑 접속](#6-4-포트-매핑-접속-확인) |
| Docker 운영 명령 | ✅ | [7. Docker 운영 로그](#7-docker-운영-로그) |
| 바인드 마운트 | ✅ | [8. 바인드 마운트](#8-바인드-마운트-검증) |
| Docker 볼륨 영속성 | ✅ | [9. 볼륨 영속성](#9-docker-볼륨-영속성-검증) |
| Git 설정 | ✅ | [10. Git / GitHub](#10-git-설정-및-githubvs-code-연동) |
| GitHub / VS Code 연동 | ✅ | [10. Git / GitHub](#10-git-설정-및-githubvs-code-연동) |

---

## 4. 터미널 조작 로그

### 4-1. 리눅스(ubuntu) 컨테이너 접속 및 기본 명령

```
PS C:\WINDOWS\system32> docker exec -it mybox bash
root@da539359a65d:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@da539359a65d:/# echo "hello from container"
hello from container
root@da539359a65d:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
...
```

### 4-2. 파일 권한 설정

```
root@da539359a65d:/# mkdir temp
root@da539359a65d:/# cd temp
root@da539359a65d:/temp# touch file.txt
root@da539359a65d:/temp# mkdir mydir
root@da539359a65d:/temp# chmod 600 file.txt
root@da539359a65d:/temp# chmod 700 mydir
root@da539359a65d:/temp# ls -l
total 4
-rw------- 1 root root    0 Nov  3 07:48 file.txt
drwx------ 2 root root 4096 Nov  3 07:49 mydir
```

- `chmod 600 file.txt` → 소유자만 읽기/쓰기 가능 (`-rw-------`)
- `chmod 700 mydir` → 소유자만 읽기/쓰기/실행 가능 (`drwx------`)

### 4-3. 작업 폴더 생성 (PowerShell)

```
PS C:\WINDOWS\system32> cd ~
PS C:\Users\User> mkdir webproject
PS C:\Users\User> cd webproject
PS C:\Users\User\webproject> mkdir app
```

---

## 5. Docker 설치·점검

```
PS C:\WINDOWS\system32> docker --version
Docker version 29.9.2, build a82e528
```

```
PS C:\WINDOWS\system32> docker info
Client:
 Version:    29.9.2
 Context:    desktop-linux
 ...
Server:
 Containers: 3
  Running: 1
  Paused: 0
  Stopped: 2
 Images: 2
 Server Version: 29.9.2
 Storage Driver: overlayfs
 Cgroup Version: 2
 Kernel Version: 6.18.54.1-microsoft-standard-WSL2
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 7.625GiB
 Docker Root Dir: /var/lib/docker
 ...
```

```
PS C:\WINDOWS\system32> docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

- Docker 클라이언트와 서버(데몬)가 정상 동작하며, 이미지 다운로드와 컨테이너 실행이 가능함을 확인

![Docker Desktop 컨테이너 목록](images/docker_desktop.png)

---

## 6. Dockerfile 기반 웹 서버

### 6-1. 소스 코드 (`app/index.html`)

```html
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>My Docker Web</title></head>
<body>
<h1>Hello Docker!</h1>
<p>Dockerfile로 만든 웹서버입니다.</p>
</body>
</html>
```

### 6-2. Dockerfile (`app/Dockerfile`)

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

- `nginx:alpine` 경량 웹 서버 이미지를 기반으로, 내가 만든 `index.html`을 nginx 기본 웹 폴더에 복사

### 6-3. 빌드 및 실행

**이미지 빌드**
```
PS C:\Users\User\webproject> cd app
PS C:\Users\User\webproject\app> docker build -t myweb .
[+] Building 6.8s (7/7) FINISHED                         docker:desktop-linux
 => [internal] load build definition from Dockerfile                     0.1s
 => => transferring dockerfile: 115B                                     0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine          2.0s
 => [internal] load .dockerignore                                        0.0s
 => => transferring context: 2B                                          0.0s
 => [internal] load build context                                        0.1s
 => => transferring context: 230B                                        0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:785835ea65db4bba6f  3.3s
 => => resolve docker.io/library/nginx:alpine@sha256:785835ea65db4bba6f  0.0s
 => => sha256:b9b388a21ecfde6ac80505f23f527146a2129df 20.53MB / 20.53MB  2.5s
 ...
 => => extracting sha256:b9b388a21ecfde6ac80505f23f527146a2129dfb104e79  0.4s
 => [2/2] COPY index.html /usr/share/nginx/html/index.html               0.1s
 => exporting to image                                                   1.1s
 => => exporting layers                                                  0.1s
 => => exporting manifest sha256:104478331c8734c3b80b8745b3bb54e486d9a1  0.0s
 => => exporting config sha256:6d949a3db229dae04c29e707979faef48937af4f  0.0s
 => => exporting attestation manifest sha256:860c4538a5102358a26fb5b721  0.0s
 => => exporting manifest list sha256:c98264c27fc05e11a49e9c929d8588869  0.0s
 => => naming to docker.io/library/myweb:latest                          0.0s
 => => unpacking to docker.io/library/myweb:latest                       0.8s
```
- Dockerfile의 2단계(nginx:alpine 이미지 가져오기 → index.html 복사)가 실행되어 `myweb:latest` 이미지 생성

**컨테이너 실행 (포트 매핑)**
```
PS C:\Users\User\webproject\app> docker run -d -p 8080:80 --name mywebserver myweb
062f4da11fc5535773620c16c8e840aa4687b3285c7fdb10880e816d7eb2468a
```

**재빌드 후 최신 이미지로 다시 실행**
```
PS C:\Users\User\webproject\app> docker build -t myweb .
[+] Building 1.6s (7/7) FINISHED                                    docker:desktop-linux
 ...
 => CACHED [2/2] COPY index.html /usr/share/nginx/html/index.html                   0.0s
 => => naming to docker.io/library/myweb:latest                                     0.0s
PS C:\Users\User\webproject\app> docker rm -f mywebserver
mywebserver
PS C:\Users\User\webproject\app> docker run -d --name mywebserver -p 8080:80 myweb
f74acf786fa91e121825979b8fb7c9f4b3fa609212c28a0d493548903ab91e43
```
- 변경 사항이 없어 `CACHED`로 빠르게 빌드됨. 기존 컨테이너를 삭제하고 최신 `myweb` 이미지로 다시 실행

![Dockerfile 및 빌드](images/dockerfile_build.png)

### 6-4. 포트 매핑 접속 확인

- `-p 8080:80` → 내 PC의 8080 포트를 컨테이너의 80 포트(nginx)에 연결
- 브라우저에서 `http://localhost:8080` 접속 시 웹 페이지 정상 표시

![localhost:8080 접속 화면](images/port_8080.png)

---

## 7. Docker 운영 로그

**컨테이너 목록 (전체 작업 완료 후)**
```
PS C:\Users\User\webproject\app> docker ps -a
CONTAINER ID   IMAGE          COMMAND                   CREATED          STATUS                   PORTS                                     NAMES
f74acf786fa9   myweb          "/docker-entrypoint.…"   5 seconds ago    Up 5 seconds             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   mywebserver
293ce2f4c398   ubuntu         "sleep infinity"          27 minutes ago   Up 27 minutes                                                      voltest2
ac42f0da6ba4   nginx:alpine   "/docker-entrypoint.…"   42 minutes ago   Up 42 minutes            0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   bindweb
150152fc2831   hello-world    "/hello"                  4 hours ago      Exited (0) 4 hours ago                                             hungry_albattani
0b90c5f6fc4a   hello-world    "/hello"                  5 hours ago      Exited (0) 5 hours ago                                             ecstatic_sinoussi
da539359a65d   ubuntu         "/bin/bash"               5 hours ago      Up 5 hours                                                         mybox
7f90b1300ccb   hello-world    "/hello"                  5 hours ago      Exited (0) 5 hours ago                                             serene_nightingale
```
- 웹 서버(`mywebserver`, 8080), 바인드 마운트(`bindweb`, 8081), 볼륨(`voltest2`), 실습용 ubuntu(`mybox`) 컨테이너가 실행 중

**이미지 목록**
```
PS C:\Users\User\webproject> docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656       25.9kB         9.49kB    U
myweb:latest         ef2cf796b233       93.6MB         26.3MB    U
ubuntu:latest        6a06111c3b0a        163MB         45.9MB    U
```
- Dockerfile로 빌드한 `myweb` 이미지 생성 확인

**웹 서버 로그**
```
PS C:\Users\User\webproject> docker logs mywebserver
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/11/03 08:25:45 [notice] 1#1: nginx/1.31.8
2026/11/03 08:25:45 [notice] 1#1: start worker processes
...
172.17.0.1 - - [03/Nov/2026:08:26:08 +0000] "GET / HTTP/1.1" 200 191 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ... Edg/156.0.0.0" "-"
172.17.0.1 - - [03/Nov/2026:08:26:08 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" ...
2026/11/03 08:26:08 [error] 30#30: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory) ...
```
- 브라우저 접속 요청에 `200` 응답한 기록 확인

**자원 사용량**
```
PS C:\Users\User\webproject> docker stats --no-stream
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O      PIDS
062f4da11fc5   mywebserver   0.00%     8.719MiB / 7.625GiB   0.11%     3.54kB / 2.57kB   0B / 12.3kB    9
da539359a65d   mybox         0.00%     1.887MiB / 7.625GiB   0.02%     1.42kB / 126B     17MB / 4.1kB   1
```

---

## 8. 바인드 마운트 검증

호스트의 `bindtest` 폴더를 컨테이너의 nginx 웹 폴더에 연결한 뒤, 호스트 파일만 수정했을 때 컨테이너에 바로 반영되는지 확인했다.

**실행**
```
PS C:\Users\User\webproject> mkdir bindtest
PS C:\Users\User\webproject> Set-Content bindtest/index.html "<h1>Before</h1>"
PS C:\Users\User\webproject> docker run -d --name bindweb -p 8081:80 -v "${PWD}/bindtest:/usr/share/nginx/html" nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
Status: Downloaded newer image for nginx:alpine
ac42f0da6ba4a818c70026b410a1db352b1e9703c83f4eef79e7721218959527
```

**변경 전**
```
PS C:\Users\User\webproject> docker exec bindweb cat /usr/share/nginx/html/index.html
<h1>Before</h1>
```
![변경 전](images/bind_before.png)

**호스트 파일 수정 후** (컨테이너 재시작 없음)
VS Code에서 호스트의 `bindtest/index.html` 내용을 `<h1>after</h1>`로 수정·저장한 뒤 확인
```
PS C:\Users\User\webproject> docker exec bindweb cat /usr/share/nginx/html/index.html
<h1>after</h1>
```
![변경 후](images/bind_after.png)

- 결과: 호스트 파일 수정이 컨테이너와 브라우저(`localhost:8081`)에 즉시 반영됨

---

## 9. Docker 볼륨 영속성 검증

볼륨에 파일을 저장한 뒤 컨테이너를 삭제하고, 새 컨테이너에 같은 볼륨을 연결해 데이터가 유지되는지 확인했다.

**볼륨 생성 및 파일 저장**
```
PS C:\Users\User\webproject> docker volume create mydata
mydata
PS C:\Users\User\webproject> docker run -d --name voltest -v mydata:/data ubuntu sleep infinity
46fd12d0d3c451cabe23f43d46f83597de66225f1d4e7f5e665a39113ca5ace8
PS C:\Users\User\webproject> docker exec voltest sh -c "echo hello-volume > /data/test.txt"
PS C:\Users\User\webproject> docker exec voltest cat /data/test.txt
hello-volume
```

**컨테이너 삭제 후 새 컨테이너에서 확인**
```
PS C:\Users\User\webproject> docker rm -f voltest
voltest
PS C:\Users\User\webproject> docker run -d --name voltest2 -v mydata:/data ubuntu sleep infinity
293ce2f4c398dfa58075d227b9dbef6794d88fbc27c1cb76a9f0a7e0ce3dbbdc
PS C:\Users\User\webproject> docker exec voltest2 cat /data/test.txt
hello-volume
```

**볼륨 정보**
```
PS C:\Users\User\webproject> docker volume ls
DRIVER    VOLUME NAME
local     mydata
PS C:\Users\User\webproject> docker volume inspect mydata
[
    {
        "CreatedAt": "2026-11-03T09:11:43Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": null,
        "Scope": "local"
    }
]
```
- 결과: 컨테이너(`voltest`)를 삭제해도 볼륨 데이터가 유지되어, 새 컨테이너(`voltest2`)에서 같은 파일을 확인함

---

## 10. Git 설정 및 GitHub/VS Code 연동

**Git 사용자 정보 및 기본 브랜치 설정** (Git Bash)
```
$ git config --global user.name "yeoreumjoo"
$ git config --global user.email "****@****.***"
$ git config --global init.defaultBranch main
$ git config --list
...
init.defaultbranch=master
user.name=yeoreumjoo
user.email=****@****.***
init.defaultbranch=main
```
- `init.defaultbranch=master`는 Git 설치 시 시스템 기본값, 아래의 `main`은 내가 설정한 사용자(global) 값으로, 사용자 설정이 우선 적용됨
- 이메일은 개인정보 보호를 위해 가림

**VS Code GitHub 로그인**

![VS Code GitHub 로그인](images/vscode_github_login.png)

**저장소 연동**
```
TODO: git remote -v 결과
TODO: git log --oneline 결과
```
![VS Code 소스 제어](images/vscode_source_control.png)

![GitHub 저장소](images/github_repo.png)

---

## 11. 트러블슈팅

### 1) 컨테이너 이름 충돌
- **문제**: `docker run -it --name mybox ubuntu` 재실행 시 `Conflict. The container name "/mybox" is already in use` 에러
- **원인 가설**: 같은 이름의 컨테이너가 이미 존재함
- **확인**: `docker ps -a`로 `mybox`가 `Up` 상태로 실행 중인 것을 확인
- **해결**: `docker run`은 새로 만드는 명령이므로, 기존 컨테이너에 `docker exec -it mybox bash`로 접속 (새로 만들려면 `docker rm -f mybox` 후 재실행)

### 2) PowerShell 명령이 실행되지 않음 (`>>` 프롬프트)
- **문제**: `docker exec voltest sh -c "echo hello-volume > /data/test.txt` 입력 후 이어지는 명령이 아무 결과 없이 넘어감
- **원인 가설**: 명령이 끝나지 않은 것으로 인식되어 실행되지 않음
- **확인**: 프롬프트가 `PS C:\...>`가 아닌 `>>`로 바뀌어 있었고, 닫는 따옴표(`"`)가 빠져 있었음
- **해결**: `Ctrl + C`로 입력 취소 후 따옴표를 닫아 다시 입력 → `hello-volume` 정상 출력

### 3) `No such container: -` 에러
- **문제**: `docker exec - it mybox bash` 실행 시 `No such container: -`
- **원인 가설**: 옵션 입력 오류
- **확인**: `-`와 `it` 사이가 띄어져 있어 `-`를 컨테이너 이름으로 인식함
- **해결**: `docker exec -it mybox bash`로 붙여서 입력

### 4) 컨테이너 안에서 docker 명령 실패
- **문제**: `docker exec -it mybox bash` 입력 시 `bash: docker: command not found`
- **원인 가설**: 명령을 입력한 위치가 내 PC가 아님
- **확인**: 프롬프트가 `root@da539359a65d:/#`로, 이미 컨테이너 내부였음 (컨테이너에는 docker가 설치되어 있지 않음)
- **해결**: `exit`로 컨테이너에서 나온 뒤 PowerShell(`PS C:\...>`)에서 docker 명령 실행

### 5) `unknown shorthand flag: 't'` 에러
- **문제**: `docker biuld -t myweb .` 실행 시 `unknown shorthand flag: 't' in -t`
- **원인 가설**: `-t` 옵션을 지원하지 않는 명령으로 인식됨
- **확인**: `build`를 `biuld`로 잘못 입력해, docker가 하위 명령을 인식하지 못하고 `-t`를 잘못된 옵션으로 처리함
- **해결**: `docker build -t myweb .`로 철자를 고쳐 다시 실행

### 6) favicon.ico 404 에러
- **문제**: `docker logs mywebserver`에 `favicon.ico ... 404`, `No such file or directory` 기록
- **원인 가설**: 브라우저가 자동으로 요청하는 아이콘 파일이 웹 폴더에 없음
- **확인**: 메인 페이지(`GET /`)는 `200`으로 정상, 아이콘 요청만 실패
- **해결/대안**: 페이지 동작에 영향 없어 유지. 필요 시 `favicon.ico` 파일을 추가하면 해결

---

## 12. 검증 방법 요약

| 확인 내용 | 사용 명령 | 결과 위치 |
|---|---|---|
| Docker 설치 | `docker --version`, `docker info`, `docker run hello-world` | [5](#5-docker-설치점검) |
| 파일 권한 | `chmod`, `ls -l` | [4-2](#4-2-파일-권한-설정) |
| 이미지 빌드 | `docker build -t myweb .`, `docker images` | [6-3](#6-3-빌드-및-실행), [7](#7-docker-운영-로그) |
| 포트 매핑 | `docker run -p 8080:80`, 브라우저 접속 | [6-4](#6-4-포트-매핑-접속-확인) |
| 컨테이너 운영 | `docker ps`, `docker logs`, `docker stats` | [7](#7-docker-운영-로그) |
| 바인드 마운트 | `docker run -v`, `docker exec ... cat` | [8](#8-바인드-마운트-검증) |
| 볼륨 영속성 | `docker volume create`, `docker rm`, `docker exec ... cat` | [9](#9-docker-볼륨-영속성-검증) |
| Git 설정 | `git config --list` | [10](#10-git-설정-및-githubvs-code-연동) |
| GitHub 연동 | `git remote -v`, `git log --oneline` | [10](#10-git-설정-및-githubvs-code-연동) |

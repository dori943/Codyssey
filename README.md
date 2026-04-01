# Codyssey 1-1
Repository for integrating GitHub and Codyssey


# 🖥️ AI/SW 개발 워크스테이션 구축

## 1. 프로젝트 개요

본 프로젝트는 AI/SW 개발을 위한 표준 워크스테이션 환경을 직접 구축하는 미션입니다.

- **핵심 도구**: 리눅스 CLI(터미널), Docker(컨테이너), Git/GitHub(버전 관리)
- **목표**: 누구나 동일하게 실행·배포·디버깅할 수 있는 재현 가능한 개발 환경 구성
- **주요 학습**: 이미지·컨테이너 분리 개념, 포트 매핑, 볼륨 영속성, Git 협업 흐름

---

## 2. 실행 환경

| 항목 | 내용 |
|------|------|
| OS | macOS 15.7.4 (Apple Silicon) |
| Shell | zsh |
| Terminal | iTerm2 / 기본 터미널 |
| Docker | 28.5.2 (OrbStack) |
| Git | 2.53.0 |

> 확인 명령어:
> ```bash
> sw_vers           # OS 버전
> docker --version  # Docker 버전
> git --version     # Git 버전
> ```

---

## 3. 수행 항목 체크리스트

| 항목 | 상태 |
|------|------|
| 터미널 기본 조작 (pwd, ls, cd, mkdir, cp, mv, rm) | ✅ |
| 파일 권한 확인 및 변경 (chmod) | ✅ |
| Docker 설치 점검 (docker --version, docker info) | ✅ |
| hello-world 컨테이너 실행 | ✅ |
| ubuntu 컨테이너 실행 및 내부 진입 | ✅ |
| Dockerfile 작성 및 커스텀 이미지 빌드 | ✅ |
| 포트 매핑으로 웹 서버 접속 확인 | ✅ |
| 바인드 마운트 변경 반영 확인 | ✅ |
| Docker 볼륨 영속성 검증 | ✅ |
| Git 사용자 설정 및 기본 브랜치 설정 | ✅ |
| VSCode GitHub 로그인 및 저장소 연동 | ✅ |

---

## 4. 검증 방법 및 결과

### 4-1. 터미널 기본 조작
```bash
# 현재 위치 확인
pwd

# 목록 확인 (숨김 파일 포함)
ls -la

# 디렉토리 생성 및 이동
mkdir -p ~/workstation/practice
cd ~/workstation/practice

# 파일 생성
touch test.txt

# 파일 내용 작성 및 확인
echo "hello world" > test.txt
cat test.txt

# 파일 복사
cp test.txt test_copy.txt

# 파일 이름 변경 (mv는 이동도 됨)
mv test_copy.txt renamed.txt

# 파일 삭제
rm renamed.txt

# 디렉토리 삭제
# -r(recursive)-> 디렉토리 안의 파일들을 전부 들어가서 삭제
# -f -> 삭제 확인 메시지 없이 강제 삭제
# -rf -> 가장 높은 강제성
mkdir temp_dir
rm -r temp_dir
```

### 4-2. 파일 권한 확인 및 변경
```bash
# | rwx | r-x | r-x        | rw- | r-- | r--|
# 본인(7) 그룹(5) 전체(5)   본인(6) (4)  (4)
# (7) : 읽기+쓰기+실행 (모든 권한)
# (6) : 읽기+쓰기
# (5) : 읽기+실행
# (5) : 읽기만
# (0) : 없음
# 현재 권한 확인
ls -l test.txt

# ls -l  #파일 권한 확인할 때 -> 디렉토리 "안의 내용물"목록을 보여줌
# ls -ld #디렉토리 권한 확인할 때 -> 디렉토리 "자체"의 정보를 보여줌

# 파일 권한 변경
chmod 755 test.txt
ls -l test.txt
# -rwxr-xr-x  (755로 변경됨)

# 디렉토리 권한 변경
mkdir secret_dir
ls -ld secret_dir
# drwxr-xr-x

chmod 700 secret_dir
ls -ld secret_dir
# drwx------  (본인만 접근 가능)
```

### 4-3. Docker 설치 및 점검
```bash
docker --version
# Docker version 28.5.2

docker info
# Client : 
#   Version: 28.5.2
#   Context: orbstack
#   ...

# 이미지 목록
docker images

# 실행 중인 컨테이너
docker ps

# 전체 컨테이너 (중지 포함)
docker ps -a
```

### 4-4. hello-world 및 ubuntu 컨테이너 실행
```bash

#hello-world 컨테이너 실행
docker run hello-world
# Hello from Docker! ...

# ubuntu 컨테이너 실행 + 내부 진입
docker run -it ubuntu bash

#컨테이너 안에서 실행
root@abc123:/# ls
root@abc123:/# echo "나는 컨테이너 안에 있다"
root@abc123:/# cat /etc/os-release

# 컨테이너 종료 (프로세스도 종료됨)
root@abc123:/# exit

# 비교: exec는 실행 중인 컨테이너에 접속, exit해도 컨테이너는 살아있음
# run -it은 foreground, run -d는 background
docker run -d --name my-ubuntu ubuntu sleep infinity
docker exec -it my-ubuntu bash
root@abc123:/# exit   # 빠져나와도 컨테이너는 계속 실행 중
docker ps             # 확인

```

### 4-5. Dockerfile 작성 및 커스텀 이미지 빌드
```bash
#폴더 만들기
mkdir -p ~/workstation/my-web/site
cd ~/workstation/my-web

# site/index.html 파일 만들기
cat > site/index.html << 'EOF'
<!DOCTYPE html>
<html>
  <body>
    <h1>My Custom Web Server</h1>
    <p>Docker로 실행 중!</p>
  </body>
</html>
EOF

# Dockerfile 만들기
cat > Dockerfile << 'EOF'
FROM nginx:alpine

LABEL org.opencontainers.image.title="my-custom-nginx"

ENV APP_ENV=dev

COPY site/ /usr/share/nginx/html/
EOF

#확인
ls -l
#total 8
#-rw-r--r--  1 ehgml89458755  ehgml89458755  126 Apr  1 13:52 Dockerfile
#drwxr-xr-x  3 ehgml89458755  ehgml89458755   96 Apr  1 13:40 site

cat Dockerfile
cat site/index.html

#빌드 및 실행
docker build -t my-web:1.0 .
# [+] Building 7.6s (7/7) FINISHED ...

# 빌드된 이미지 확인
docker images

#
docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
docker run -d -p 8081:80 --name my-web-8081 my-web:1.0

curl http://localhost:8080
# <html>...
```

> 브라우저 접속 화면: [아래 스크린샷 참고](#스크린샷)

### 4-6. 바인드 마운트
```bash
docker run -d -p 8082:80 \
  -v $(pwd)/site:/usr/share/nginx/html \
  --name bind-test my-web:1.0

# 호스트에서 파일 수정 후 브라우저 새로고침으로 즉시 반영 확인
echo "<h1>변경됨</h1>" > site/index.html
```

### 4-7. Docker 볼륨 영속성
```bash
docker volume create mydata

docker run -d --name vol-test -v mydata:/data ubuntu sleep infinity
docker exec -it vol-test bash -c "echo 'hello volume' > /data/hello.txt"
docker rm -f vol-test

docker run -d --name vol-test2 -v mydata:/data ubuntu sleep infinity
docker exec -it vol-test2 bash -c "cat /data/hello.txt"
# hello volume  ← 컨테이너 삭제 후에도 데이터 유지됨
```

### 4-8. Git 설정
```bash
git config --global user.name "이름"
git config --global user.email "이메일"
git config --global init.defaultBranch main

git config --list
# user.name=이름
# user.email=이메일
# init.defaultBranch=main
```

---

## 5. 트러블슈팅

### 🔧 Case 1: [문제 제목]

| 단계 | 내용 |
|------|------|
| **문제** | docker: 'docker buildx build' requires 1 argument |
| **원인 가설** | 에러 메시지의 requires 1 argument는 "인자(데이터)가 하나 더 필요하다"는 뜻 -> 명령어 맨 뒤에 한 칸 띄우고 점(.)을 꼭 찍어주세요. 여기서 점(.)은 "현재 디렉토리"를 의미합니다.|
| **해결/대안** | 명령어 맨 뒤에 한 칸 띄우고 점(.) 찍어|
```bash
ehgml89458755@c4r5s7 my-web % docker build -t my-web:1.0
#ERROR: docker: 'docker buildx build' requires 1 argument

#Usage:  docker buildx build [OPTIONS] PATH | URL | -

#Run 'docker buildx build --help' for more information

```

---

### 🔧 Case 2: [문제 제목]

| 단계 | 내용 |
|------|------|
| **문제** | 어떤 오류가 발생했는지 |
| **원인 가설** | 왜 발생했을 것이라 생각했는지 |
| **확인** | 어떤 명령/방법으로 확인했는지 |
| **해결/대안** | HTML 파일을 어떻게 웹 서버로 돌릴지 정의하는 Dockerfile을 먼저 만드셔야 합니다. |
```bash
ehgml89458755@c4r5s7 my-web % docker build -t my-web:1.0 .
[+] Building 0.4s (1/1) FINISHED                                                                                                          docker:orbstack
 => [internal] load build definition from Dockerfile                                                                                                 0.2s
 => => transferring dockerfile: 2B                                                                                                                   0.0s
ERROR: failed to build: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

---

## 6. 스크린샷

### 포트 매핑 접속 화면
<!-- 여기에 스크린샷 이미지 추가 -->
![포트 8080 접속](./screenshots/port-8080.png)
![포트 8081 접속](./screenshots/port-8081.png)

### VSCode GitHub 연동
<!-- 여기에 스크린샷 이미지 추가 -->
![VSCode GitHub 연동](./screenshots/vscode-github.png)

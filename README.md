# Docker 개발 워크스테이션 구축

## 1) 프로젝트 개요
- 목표: 터미널로 작업 디렉토리/권한을 정리하고, Docker를 설치·점검한 뒤 Dockerfile 기반 웹 서버를 컨테이너화한다.
- 포트 매핑 접속, 바인드 마운트(변경 반영), Docker 볼륨(데이터 영속성)을 실행 결과로 직접 검증한다.
- 이미지/컨테이너 분리, 격리된 실행 환경, 포트·스토리지 연결 방식의 설계 이유를 설명한다.

### 학습 목표 (과제를 마치면 스스로 설명할 수 있어야 하는 것)
- [ ] 절대 경로와 상대 경로의 차이를 예시와 함께 설명한다. → [13) 개념 정리](#13-개념-정리)
- [ ] 파일 권한 r/w/x와 755, 644 같은 표기의 해석 규칙을 설명한다. → [6) 권한 실습](#6-권한-실습), [13) 개념 정리](#13-개념-정리)
- [ ] 기존 Dockerfile을 기반으로 커스텀 이미지를 만든다. → [8) 커스텀 이미지](#8-커스텀-이미지-dockerfile)
- [ ] 포트 매핑이 필요한 이유를 설명한다. → [9) 포트 매핑](#9-포트-매핑-접속-증거-2회)
- [ ] Docker 볼륨(영속 데이터)을 설명한다. → [11) 볼륨 영속성](#11-docker-볼륨-영속성)
- [ ] Git과 GitHub의 역할 차이를 설명한다. → [13) 개념 정리](#13-개념-정리)

## 2) 실행 환경
- OS: Windows (버전 기록)
- Shell: PowerShell (버전: `$PSVersionTable.PSVersion` 출력)
- Docker: (`docker --version` 출력)
- Git: (`git --version` 출력)

```powershell
# 여기에 위 3개 명령의 출력 붙여넣기
```

## 3) 수행 체크리스트
- [ ] 터미널 기본 조작 (위치/목록/이동/생성/복사/이름변경/삭제/내용 확인)
- [ ] 권한 변경 실습 (파일 1개 + 디렉토리 1개, 전/후 비교)
- [ ] Docker 설치/점검 (`docker --version`, `docker info`)
- [ ] Docker 기본 운영 (`images`, `ps`, `ps -a`, `logs`, `stats`)
- [ ] hello-world 실행
- [ ] ubuntu 컨테이너 진입 + 간단 명령
- [ ] Dockerfile 커스텀 이미지 빌드/실행
- [ ] 포트 매핑 접속 (2회, 브라우저 스크린샷 + curl)
- [ ] 바인드 마운트 반영 (전/후 비교)
- [ ] Docker 볼륨 영속성 (컨테이너 삭제 전/후)
- [ ] Git 설정 (`git config --list`)
- [ ] VSCode GitHub 연동 + push

## 4) 디렉토리 구성
```
.
├── README.md        # 기술 문서
├── Dockerfile       # 커스텀 이미지 정의
├── site/            # 웹 서버 정적 콘텐츠 (index.html)
└── evidence/        # 스크린샷 (브라우저 접속, GitHub 연동 등)
```

## 5) 터미널 조작 로그 (PowerShell)
```powershell
Get-Location                      # 현재 위치 (pwd)
New-Item -ItemType Directory -Path practice
Set-Location practice
Get-ChildItem -Force              # 숨김 파일 포함 목록 (ls -la)
New-Item empty.txt -ItemType File # 빈 파일 생성
"hello" | Out-File a.txt -Encoding utf8
Get-Content a.txt                 # 내용 확인 (cat)
Copy-Item a.txt b.txt
Move-Item b.txt c.txt             # 이동/이름변경
Remove-Item c.txt
# 각 명령 아래에 실제 출력 붙여넣기
```

## 6) 권한 실습
> 과제가 `chmod`/rwx를 요구합니다. PowerShell에는 `chmod`가 없으므로 **WSL Ubuntu 또는 Git Bash**에서 수행하고, 사용한 환경을 여기에 명시하세요.

```bash
ls -l a.txt && chmod 600 a.txt && ls -l a.txt
mkdir testdir && ls -ld testdir && chmod 700 testdir && ls -ld testdir
# 실제 출력 붙여넣기
```

**rwx와 숫자 표기**: r=4, w=2, x=1이며 소유자/그룹/기타 순으로 합산한다. 예) `755` = 소유자 rwx(7), 그룹 r-x(5), 기타 r-x(5).

## 7) Docker 점검 및 기본 운영
```powershell
docker --version
docker info
docker run hello-world
docker run -it --name u1 ubuntu bash   # 내부에서 ls, echo 실행 후 exit
docker ps -a
docker images
docker logs web1
docker stats --no-stream
# 실제 출력 붙여넣기
```

**attach / exec 차이 관찰**: (직접 해본 결과를 2~3줄로 정리)

## 8) 커스텀 이미지 (Dockerfile)
- 선택한 베이스: (예: `nginx:alpine`, 방식 A)
- 커스텀 포인트와 목적:
  - `COPY site/ /usr/share/nginx/html/`: nginx 기본 정적 파일 경로에 내 페이지를 넣기 위함
  - (추가한 것과 이유)

```powershell
docker build -t my-web-server:v1 .
docker run -d -p 18881:80 --name web1 my-web-server:v1
# 출력 붙여넣기
```

## 9) 포트 매핑 접속 증거 (2회)
```powershell
docker run -d -p 18882:80 --name web2 my-web-server:v1
curl.exe http://127.0.0.1:18881
curl.exe http://127.0.0.1:18882
# 출력 붙여넣기
```
![브라우저 접속](evidence/port-18881.png)  <!-- 주소창이 보이게 캡처 -->

## 10) 바인드 마운트
```powershell
docker run -d -p 18883:80 -v "${PWD}/site:/usr/share/nginx/html" --name bind1 nginx:alpine
curl.exe http://127.0.0.1:18883
# site/index.html 수정
curl.exe http://127.0.0.1:18883
# 수정 전/후 출력 붙여넣기
```

## 11) Docker 볼륨 영속성
```powershell
docker volume create mydata
docker run -d --name vol1 -v mydata:/data ubuntu sleep infinity
docker exec vol1 bash -c "echo hi > /data/hello.txt && cat /data/hello.txt"
docker rm -f vol1
docker run -d --name vol2 -v mydata:/data ubuntu sleep infinity
docker exec vol2 cat /data/hello.txt
# 삭제 전/후 출력 붙여넣기
```
**백업 권장**: `docker run --rm -v mydata:/data -v ${PWD}:/backup ubuntu tar czf /backup/mydata.tar.gz /data`

## 12) Git / GitHub
```powershell
git config --list      # 이메일 등 개인정보 마스킹
git remote -v
git push
```
![GitHub 연동](evidence/github.png)

## 13) 개념 정리

### 절대 경로 vs 상대 경로
- **절대 경로**: 루트(드라이브)부터 시작하는 전체 경로로, 현재 위치와 무관하게 항상 같은 대상을 가리킨다.
- **상대 경로**: 현재 작업 디렉토리를 기준으로 한 경로다. `.`은 현재 위치, `..`은 상위 디렉토리다.
- 예시 (현재 위치가 `C:\Users\me\practice`일 때):

| 구분 | Windows | Linux/WSL |
|---|---|---|
| 절대 경로 | `C:\Users\me\practice\a.txt` | `/home/me/practice/a.txt` |
| 상대 경로 | `.\a.txt`, `..\other\b.txt` | `./a.txt`, `../other/b.txt` |

- 이 과제에서의 사용: 바인드 마운트(`-v`)와 Dockerfile 경로는 절대 경로로 쓰면 어디서 실행해도 같은 결과가 나와 재현성이 높고, 상대 경로는 짧지만 실행 위치에 따라 달라진다.

### 파일 권한 (r/w/x, 755, 644)
- **r(읽기)=4, w(쓰기)=2, x(실행)=1**이며, 소유자(user) / 그룹(group) / 기타(others) 순서로 3자리를 쓴다. 각 자리는 해당 권한 값을 더한 합이다.
- `755` = 소유자 7(rwx=4+2+1), 그룹 5(r-x=4+1), 기타 5(r-x) → `rwxr-xr-x`. 실행 파일과 디렉토리에 흔히 쓴다.
- `644` = 소유자 6(rw-=4+2), 그룹 4(r--), 기타 4(r--) → `rw-r--r--`. 일반 파일에 흔히 쓴다.
- 디렉토리에서 x는 "그 안으로 들어갈 수 있는 권한"을 뜻한다.

### 이미지 vs 컨테이너
- 이미지는 변경되지 않는 읽기 전용 템플릿, 컨테이너는 이를 실행한 인스턴스다. 빌드는 이미지를, 실행은 컨테이너를 만들고, 컨테이너 안의 변경은 이미지에 반영되지 않는다.

### 커스텀 이미지
- 기존 베이스 이미지(예: `nginx:alpine`) 위에 `COPY`, `ENV`, `LABEL` 등을 얹어 내 목적에 맞는 이미지를 만든다. 베이스를 재사용하므로 처음부터 만들 필요가 없고, Dockerfile로 빌드 과정을 재현할 수 있다.
- `/usr/share/nginx/html`은 nginx의 기본 정적 파일 경로라서 이 위치에 콘텐츠를 넣는다.

### 포트 매핑이 필요한 이유
- 컨테이너는 별도의 네트워크 네임스페이스에서 격리되어 실행되므로 호스트에서 컨테이너 포트로 바로 접근할 수 없다. `-p 호스트:컨테이너`로 호스트 포트를 컨테이너 포트에 연결해야 접속된다. (예: `-p 18881:80`)

### Docker 볼륨 (영속 데이터)
- 컨테이너의 쓰기 계층은 컨테이너를 삭제하면 함께 사라진다. 볼륨은 컨테이너 밖에서 Docker가 관리하는 저장소라서, 컨테이너를 삭제해도 데이터가 남고 새 컨테이너에 다시 연결할 수 있다.
- 바인드 마운트는 호스트의 특정 폴더를 그대로 연결해 "변경 즉시 반영"에 적합하고, 볼륨은 Docker가 관리하는 "데이터 보존"에 적합하다.

### Git vs GitHub
- **Git**: 내 컴퓨터에서 파일 변경 이력을 관리하는 로컬 버전관리 도구다. 인터넷 없이도 커밋, 브랜치, 되돌리기가 가능하다.
- **GitHub**: Git 저장소를 올려 두고 공유·협업(원격 저장소, Pull Request, 이슈, 리뷰)을 하는 원격 플랫폼이다.
- 관계: Git으로 로컬에 커밋하고, `git push`로 GitHub의 원격 저장소에 올린다.

## 14) 트러블슈팅
### (1) 한글 깨짐
- 문제 → 원인 가설 → 확인 → 해결 (HTML에 `<meta charset="UTF-8">` 추가 등 실제 경험 기록)

### (2) 포트 충돌
- 문제: `docker run -p 18881:80` 시 포트 사용 중 오류
- 원인 가설: 이미 해당 포트를 쓰는 프로세스/컨테이너 존재
- 확인:
```powershell
netstat -ano | findstr :18881
docker ps
```
- 해결: 점유 프로세스 종료(`Stop-Process -Id <PID>`) 또는 다른 포트로 실행

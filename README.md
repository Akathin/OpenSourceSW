# Linux 프로세스 작업 도구 정리 (top / ps / jobs / kill)

![header](https://capsule-render.vercel.app/api?type=venom&color=auto&height=200&section=header&text=리눅스%20명령어&fontSize=50)

---

## 목차
1. [개요](#개요)  
2. [top](#top) — 실시간 시스템/프로세스 모니터  
3. [ps](#ps) — 프로세스 스냅샷 조회  
4. [jobs](#jobs) — 현재 셸의 백그라운드 작업 관리  
5. [kill](#kill) — 프로세스에 신호 보내기  

---

# 개요
간단 요약:

- `top` : 실시간으로 CPU / 메모리 사용량과 프로세스 목록을 보여줌. (인터랙티브)
- `ps`  : 현재 시점의 프로세스 상태(스냅샷)를 출력. (비대화형, 스크립트용)
- `jobs`: 현재 셸(예: bash/zsh)에서 시작한 작업의 상태(포그라운드/백그라운드)를 보여줌.
- `kill`: 프로세스에 시그널을 보내 종료하거나 상태를 변경함.

---

# top
`top`은 시스템 상태와 프로세스 리스트를 실시간으로 보여주는 도구입니다.

## 주요 특징
- 실시간 갱신(기본 3초)  
- 상단에 시스템 요약(로드, 사용률, 메모리 등)  
- 하단에 프로세스 목록 — 정렬, 필터링, 신호 전송 가능

## 자주 쓰는 키 (인터랙티브)
- `P` : CPU 사용량으로 정렬  
- `M` : 메모리 사용량으로 정렬  
- `T` : 실행 시간(TIME)으로 정렬  
- `k` : 프로세스 종료 (PID 입력 후 신호)  
- `r` : 프로세스 우선순위(renice) 변경  
- `q` : 종료

## 편리한 옵션
- `top -b -n 1` : 배치 모드(스크립트용), 1회 출력 후 종료  
- `top -o %MEM` : 시작 시 메모리 순으로 정렬 (버전에 따라 `-O` 또는 `-o` 차이 있음)

## 예제
```bash
# 실시간 확인 (대화형)
top

# 배치모드로 상위 10개 프로세스를 비동기 로그로 저장
top -b -n 1 | head -n 20 > top_snapshot.txt

# 메모리 사용 상위 정렬
top -o %MEM
```

---

# ps
`ps`는 프로세스 정보를 스냅샷으로 보여줍니다. 자동화 / 스크립트에 자주 사용됩니다.

## 기본형
```bash
ps aux
# 또는
ps -ef
```

> `ps aux`는 BSD 스타일, `ps -ef`는 SysV 스타일. 출력 컬럼 이름이 조금 다릅니다.

## 주요 옵션(자주 쓰이는 조합)
- `a` : 모든 사용자 프로세스(터미널 없는 프로세스 포함)  
- `u` : 사용자/CPU/메모리 등 가독성 있는 포맷  
- `x` : 터미널 없는 프로세스 포함  
- `-e` : 모든 프로세스 (POSIX 스타일)  
- `-f` : 풀 포맷(long listing)  
- `-o` : 출력 칼럼 지정 (예: `ps -eo pid,ppid,user,%mem,%cpu,command`)

## 프로세스 찾기 (예: 명령어로)
```bash
# 특정 이름을 가진 프로세스 찾기 (grep 방식)
ps aux | grep -i nginx | grep -v grep

# 더 깔끔한 방법 (pid만 출력)
ps -C nginx -o pid,cmd

# 특정 패턴의 pid만 뽑기
ps -eo pid,cmd | grep '[n]ginx'
```

## 예제
```bash
# 모든 프로세스를 긴 포맷으로 (UID PID PPID C STIME TTY TIME CMD)
ps -ef

# 출력 컬럼을 직접 지정 (스크립트용으로 유용)
ps -eo pid,ppid,user,%mem,%cpu,comm --sort=-%mem | head -n 15
```

---

# jobs
`jobs`는 **현재 사용 중인 셸 세션**에서 시작한 작업(job)의 상태를 보여줍니다.

## 출력 예시
```
$ sleep 1000 &
[1] 12345
$ jobs
[1]+  Running                 sleep 1000 &
```

## 주요 옵션
- `jobs -l` : PID까지 표시  
- `jobs -p` : job의 PID만 출력

## 관련 명령
- `fg %1` : job 번호 1을 포그라운드로 가져오기  
- `bg %1` : job 번호 1을 백그라운드로 재개  
- `disown %1` : 셸의 job 관리에서 제거(로그아웃에도 계속 실행)

## 예제
```bash
# 백그라운드로 실행
long_running_task &

# 중단 후 백그라운드로 (Ctrl+Z -> bg)
# 작업 확인
jobs -l

# 포그라운드로 복귀
fg %1
```

---

# kill
`kill`은 프로세스(또는 PID들)로 시그널을 보내는 도구입니다.

## 자주 쓰는 시그널
- `SIGTERM` (15) : 정상 종료 요청  
- `SIGKILL` (9)  : 강제 종료  
- `SIGINT` (2)   : Ctrl+C  
- `SIGHUP` (1)   : 터미널 연결 종료 시 발생

## 기본 사용법
```bash
kill 1234
kill -9 1234
kill -s SIGTERM 1234
```

## 프로세스 이름으로 종료
```bash
killall nginx
pkill -f 'java -jar myapp.jar'
```

---

# Cheat Sheet

| 명령어 | 목적 | 예제 |
|-------|------|-------|
| top | 실시간 모니터링 | `top -o %MEM` |
| ps | 프로세스 조회 | `ps aux`, `ps -ef` |
| jobs | 셸 작업 관리 | `jobs -l` |
| kill | 프로세스 종료 | `kill -9 1234` |

---

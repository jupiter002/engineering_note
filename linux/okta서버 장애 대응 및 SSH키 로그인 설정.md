---
title: worldokta 서버 장애 대응 및 SSH 키 로그인 전환
date: 2026-11-10
server: 211.45.163.115 (worldokta, cafe24 KVM, Rocky Linux 8)
tags: [server, tomcat, ssh, 장애대응, worldokta]
---

# worldokta 서버 장애 대응 및 SSH 키 로그인 전환

## 서버 정보
| 항목 | 값 |
|---|---|
| WAS 서버 | `211.45.163.115` (hostname `worldokta`), SSH 포트 **1212** |
| DB 서버 | `211.45.163.136:3306` (MariaDB, Termius 이름 `oktadb`) |
| OS | Rocky Linux 8 (cafe24 KVM 가상머신) |
| Tomcat | 8.5.82, 경로 `/tomcat8.5`, 포트 HTTP 8088 / HTTPS 8443 |
| 앱 | Spring Boot 2.7.7, context `/jwt`, Host `oktajob.net` |
| Java | OpenJDK **1.8.0_422** (**JRE만 설치됨**, javac 없음) |
| Java 경로 | `/usr/lib/jvm/jre` → `/etc/alternatives/jre` → `/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.422.b05-2.el8.x86_64/jre` |
| 환경변수 | `/tomcat8.5/bin/setenv.sh` (DB·메일 계정 등) |

---

## 1. 장애: 서버 통신 오류 (2026-11-05 ~ 11-06)

### 원인 1 — DB 연결 거부 (11-05)
```
Socket fail to connect to host:address=(host=211.45.163.136)(port=3306) Connection refused
HikariPool-1 - Connection is not available, request timed out after 9000ms
```
- `Connection refused` = DB 서버의 3306 포트에서 응답하는 프로세스가 없음. MariaDB 중지나 재시작으로 추정
- 11-05 13:51, 16:20 등에 간헐적으로 발생 (34건) → 이후 회복

### 원인 2 — 서버 재부팅 후 Tomcat 미기동 (11-06)
- **00:35:16** Tomcat 정상 종료 (`[Thread-5]` = JVM 종료 훅 → 외부 SIGTERM)
- **00:43:38** OS 재부팅 (커널 부팅 로그 확인, sshd PID 초기화)
- 재부팅 직전 로그인·cron·종료 명령 기록이 없음 → **호스팅사(cafe24)에서 VM을 재시작한 것으로 추정**
- OOM·JVM 크래시는 아님 (OutOfMemoryError 0건, hs_err 파일 없음)
- **Tomcat이 systemd에 등록돼 있지 않아서** 재부팅 후 꺼진 채로 남아 있었음
- 10-24에도 기동 1초 만에 종료된 기록이 있음

> [!todo] 남은 확인
> - [x] cafe24에 2026-11-06 00:35 전후 VM 재시작 이력 문의
> - [x] DB 서버에서 11-05 13:51 전후 MariaDB 재시작·OOM 로그 확인

### 참고: 장애와 무관한 로그
- `Invalid character found in method name [0x16 0x03 ...]` → HTTP 포트(8088)에 HTTPS로 접속한 흔적
- `ThreadLocal ... memory leak` (종료 시) → netty 경고, 무시해도 됨
- `Unknown default host [localhost]` → `server.xml`의 `<Engine defaultHost="localhost">`와 `<Host name="oktajob.net">` 이름이 달라서 나옴. HTTP/1.0 요청에만 영향. 없애려면 `defaultHost="oktajob.net"`

---

## 2. 조치: Tomcat systemd 서비스 등록

### `/etc/systemd/system/tomcat.service`
```ini
[Unit]
Description=Tomcat 8.5
After=network.target mariadb.service

[Service]
Type=forking
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/tomcat8.5
ExecStart=/tomcat8.5/bin/startup.sh
ExecStop=/tomcat8.5/bin/shutdown.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable tomcat
systemctl start tomcat
```

> [!warning] JAVA_HOME 주의
> `.../java-1.8.0-openjdk-1.8.0.422.b05-2.el8.x86_64`(jre의 상위 폴더)를 지정하면 `bin/java`가 없어 **기동 실패**.
> 링크 경로 `/usr/lib/jvm/jre`를 쓰면 `dnf update`로 Java 패치 버전이 바뀌어도 수정할 필요가 없음.

### 운영 명령 (앞으로는 startup.sh 대신 사용)
```bash
systemctl status tomcat
systemctl restart tomcat
systemctl stop tomcat
tail -f /tomcat8.5/logs/catalina.out   # "Server startup in ... ms" 확인
```
- 확인 결과: 프로세스가 `system.slice/tomcat.service`에서 실행 중 ✅
- `startup.sh`를 직접 실행하면 systemd가 관리하지 못함
- 셸에서 `startup.sh`만 치면 `command not found` → `./startup.sh`로 실행해야 함

---

## 3. 조치: SSH 키(ppk) 로그인 전환

### 순서
1. **PuTTYgen**에서 키 생성 (RSA 4096 또는 Ed25519) → `worldokta.ppk` 저장
2. 공개키 한 줄(`ssh-rsa AAAA...`)을 서버에 등록
   ```bash
   mkdir -p /root/.ssh && chmod 700 /root/.ssh
   echo 'ssh-rsa AAAA...전체 한 줄...' > /root/.ssh/authorized_keys
   chmod 600 /root/.ssh/authorized_keys
   restorecon -Rv /root/.ssh
   wc -c /root/.ssh/authorized_keys   # RSA 2048 ≥ 380, RSA 4096 ≈ 720
   ```
3. Termius: Keychain에 키 Import → Host(okta) 편집 → **Key 선택, Password 비우기**
4. **기존 창을 유지한 채** 새 탭으로 키 로그인 확인
5. 성공한 다음에만 비밀번호 로그인 끄기

### 최종 `/etc/ssh/sshd_config` 핵심값
```
Port 1212
PermitRootLogin prohibit-password
PasswordAuthentication no
ChallengeResponseAuthentication no
AuthorizedKeysFile .ssh/authorized_keys
```
```bash
sshd -t && systemctl restart sshd
sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
# permitrootlogin without-password   (= prohibit-password의 옛 이름)
# passwordauthentication no
```

### 겪은 문제와 해결
| 증상 | 원인 | 해결 |
|---|---|---|
| Termius `Authentication failed (publickey)` | `authorized_keys`가 **60바이트로 잘림** (붙여넣기 실패) | 공개키 전체 한 줄을 다시 등록 |
| 비밀번호도 거부됨 (`Authentication that can continue: gssapi-with-mic,publickey`) | `PasswordAuthentication no`가 먼저 적용됨 | 열린 세션에서 `yes`로 되돌림 |
| `PasswordAuthentication yes`로 바꿨는데도 root 비밀번호 거부 | `PermitRootLogin prohibit-password`(=`without-password`)가 남아 있었음 | `PermitRootLogin yes`로 임시 복구 → 키 확인 후 다시 변경 |

> [!danger] 교훈
> - 비밀번호 로그인을 끄기 전에 **반드시 새 창으로 키 로그인 성공을 확인**할 것
> - 설정은 **하나씩 바꾸고, 바꿀 때마다 확인**할 것
> - 확인 명령은 파일이 아니라 `sshd -T` (실제 적용값)
> - 모두 막혔을 때는 **cafe24 웹 콘솔(VNC)**로 복구

### passphrase
- `.ppk` 파일 자체를 잠그는 비밀번호. **서버로 전송되지 않음**
- 키 파일이 유출돼도 passphrase를 모르면 사용할 수 없음
- 잊으면 복구 불가 → 비밀번호 관리자에 저장. 잊었으면 키를 새로 만들어 재등록
- 매번 입력하기 번거로우면 **Pageant**에 등록

### ppk → pem 변환
- PuTTYgen: Load `.ppk` → **Conversions → Export OpenSSH key**
- CLI: `puttygen worldokta.ppk -O private-openssh -o worldokta.pem`
- RSA → `BEGIN RSA PRIVATE KEY` / Ed25519 → `BEGIN OPENSSH PRIVATE KEY` (오래된 도구는 못 읽을 수 있음)
- Windows OpenSSH에서 쓰면 권한 정리 필요: `icacls worldokta.pem /inheritance:r /grant:r "$env:USERNAME:R"`

### 비밀번호 로그인을 끈 뒤 바꿔야 하는 것
- [ ] **SSHFS-Win 드라이브**: passphrase 없는 키를 `C:\Users\A\.ssh\id_rsa`에 두고 `\\sshfs.k\root@211.45.163.115!1212`로 재연결 (`.r` → `.k`)
- [ ] WinSCP / FileZilla 등에도 키 지정
- [ ] **oktadb(211.45.163.136)**에 키 로그인을 적용하려면 같은 공개키를 그 서버의 `authorized_keys`에도 등록. 그 전에는 Termius에서 "Create an identity"로 두 호스트를 합치지 말 것

---

## 4. 보안 TODO
- [ ] `/var/log/secure`에 외부 IP의 root 비밀번호 대입 공격이 지속됨 → 키 로그인 전환으로 차단 (fail2ban 동작 중)
- [ ] `bin/setenv.sh`에 DB root 비밀번호·Gmail 앱 비밀번호가 평문으로 있음 → `chmod 600` 적용, DB는 앱 전용 계정으로 교체
- [ ] (선택) `sshd_config`: `X11Forwarding no`, `MaxAuthTries 3`, `GSSAPIAuthentication no`
- [ ] (선택) 서버에서 진단 도구(jstack, jmap)가 필요하면 `dnf install java-1.8.0-openjdk-devel`

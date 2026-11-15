
rmfjaNov 11, 2026 · @sonet

## 요약

NEO SSH-Win Manager로 마운트한 E: 드라이브는 경로 확인(realpath)이 실패해 Claude Code가 작업 범위 밖으로 처리했다. `net use`로 E:를 네트워크 드라이브(`\\sshfs.r\...`)로 다시 연결하자 `--add-dir` 없이도 접근이 됐다.

## 증상

예전에는 마운트한 드라이브에 Claude Code가 바로 접근했는데, 지금은 `--add-dir`을 붙여야만 접근됐다.

- 환경: Windows 11 Pro, Claude Code 2.1.322 (자동 업데이트 `latest` 채널)
- 대상: `E:\var\www\html\vitamall_dev` (서버 175.125.21.247의 `/var/www/html`)
- CLI: 다른 폴더에서 시작하면 `claude --add-dir E:\var\www\html\vitamall_dev`로만 접근 가능
- `settings.json`의 `additionalDirectories`에 E: 경로를 넣어도 CLI와 VS Code 확장 모두 접근 불가

## 진단 과정

설정과 확장을 정리해도 해결되지 않았고, 경로 확인 테스트에서 원인이 드러났다.

1. `~/.claude/settings.json`에 `permissions.additionalDirectories`를 추가했다. → 여전히 접근 불가
2. VS Code 확장 폴더에 2.1.283과 2.1.322가 함께 있어 구버전 2.1.283을 삭제했다. → 변화 없음
3. Node.js `fs.realpathSync.native`로 경로를 확인했다.
   - `E:\`, `E:\var\www\html\vitamall_dev` → `UNKNOWN` 오류
   - `Z:\var\www\html` → `\\sshfs.r\root@175.125.21.247!22\var\www\html` (정상)
4. E:는 `net use`, `subst`, 볼륨 목록 어디에도 없었다. NEO SSH-Win Manager가 네트워크 드라이브가 아닌 형태로 마운트한 것이었다.
5. Z:는 예전에 `net use`로 만든 매핑이 남아 있던 것이다. 명령줄에서는 동작했지만 파일 탐색기에는 보이지 않았다.

## 원인

Claude Code는 접근을 허용하기 전에 경로의 실제 위치를 확인하는데, NEO 매니저의 E: 마운트에서는 이 확인이 실패한다.

| 드라이브 | 마운트 방식 | 경로 확인 | 파일 탐색기 | Claude Code 설정 적용 |
| --- | --- | --- | --- | --- |
| E: (NEO 매니저) | `sshfs.exe` 직접 마운트 | 실패 | 보임 | 안 됨 |
| Z: (예전 `net use`) | `\\sshfs.r\...` 네트워크 드라이브 | 정상 | 안 보임 | 됨 |

`--add-dir`로 넣은 경로는 이 확인을 거치지 않거나 실패해도 넘어가는 것으로 보인다. 예전에 되던 이유는 Claude Code 업데이트로 경로 확인이 엄격해졌기 때문으로 추정하며, 변경 기록으로 확인하지는 못했다.

### NEO 매니저 연결에서 무엇이 문제였나

서버 연결은 정상이었고, Windows에 드라이브가 등록되는 형태가 달랐던 것이 문제였다.

- NEO 매니저는 `sshfs.exe`로 드라이브 문자에 직접 마운트한다. 이렇게 붙인 드라이브는 네트워크 드라이브로도, 일반 볼륨으로도 등록되지 않는다.
- 파일을 열고 읽고 쓰는 일반 작업은 정상이다. 그래서 파일 탐색기와 VS Code 편집기에서는 문제가 없어 보였다.
- 하지만 경로의 실제 위치(realpath)를 물으면 Windows가 답하지 못하고 `UNKNOWN` 오류를 낸다.
- Claude Code는 바로가기나 심볼릭 링크로 작업 범위를 우회하지 못하도록 접근 전에 이 실제 위치를 확인한다. 확인이 실패하면 범위 밖으로 처리해서 `settings.json`의 경로가 무시됐다.
- 예전 Claude Code는 확인이 실패하면 입력한 경로를 그대로 쓴 것으로 보인다. 최근 업데이트에서 막는 방식으로 바뀐 것으로 추정하며, 변경 기록으로 확인하지는 못했다.

정리하면 NEO 매니저 연결은 파일을 다루는 데는 문제가 없지만, 경로를 엄격하게 검사하는 프로그램과는 맞지 않는다.

## 해결

E:를 네트워크 드라이브로 다시 연결하고 사용자 설정에 작업 폴더를 등록해 해결했다.

1. NEO SSH-Win Manager에서 vitamall 연결을 해제한다.
2. 명령 프롬프트에서 남은 Z: 매핑을 지우고 E:를 네트워크 드라이브로 연결한다.

```
net use Z: /delete
net use E: \\sshfs.r\root@175.125.21.247!22 /persistent:yes
```

3. `~/.claude/settings.json`에 작업 폴더를 등록한다.

```json
"permissions": {
  "additionalDirectories": ["E:\\var\\www\\html\\vitamall_dev"]
}
```

4. Claude Code(CLI, VS Code)를 다시 시작한다. `--add-dir` 없이 접근되는 것을 확인했다.

## 참고 및 남은 확인 사항

- 다른 서버(H:, K:, D:, F:, I:)도 NEO 매니저로 마운트하면 같은 문제가 생길 수 있다. 같은 방법으로 `net use 드라이브: \\sshfs.r\사용자@호스트!포트`를 쓴다.
- 경로 확인 테스트: `node -e "console.log(require('fs').realpathSync.native('E:\\'))"`. 오류가 나면 Claude Code 설정이 적용되지 않는다.
- 설정을 바꾼 뒤에는 VS Code도 완전히 종료했다가 다시 실행해야 반영된다.
- 재부팅 후 `/persistent:yes` 매핑이 자동으로 다시 연결되는지는 아직 확인하지 않았다.

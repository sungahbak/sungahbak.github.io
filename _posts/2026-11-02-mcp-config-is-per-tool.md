---
title: "Antigravity에 Notion MCP를 깔았는데 Claude Code에는 안 보인다"
description: "MCP 설정이 계정 단위가 아니라 도구 단위라는 걸 모르고 세 번 오진한 기록."
date: 2026-11-02 23:50:00 +0900
categories: [트러블슈팅, 개발환경]
tags: [mcp, claude-code, antigravity, notion, oauth]
# 아래 두 항목은 나중에 에러 메시지 검색 색인에 쓰기 위해 쌓아두는 값이다.
error_signatures:
  - "Needs authentication"
  - "CONNECTION_CLOSED"
  - "Skipping connection (recent failure cached"
verified: 2026-11-02
---

> 증상 1개짜리 트러블슈팅. MCP 설정이 "계정 단위"가 아니라 "도구 단위"라는 걸 모르고 세 번 오진한 기록.
{: .prompt-info }

## 1. 문제 상황

Antigravity(Gemini 기반 IDE)에 Notion 플러그인을 설치했다. 설치는 성공했고 플러그인 목록에도 떴다.

그런데 **같은 머신, 같은 계정**에서 Claude Code를 열고 MCP 서버 목록을 확인하면 Notion이 없다.

```bash
$ claude mcp list
Checking MCP server health…

claude.ai Claude Docs: https://api.anthropic.com/v1/pages/mcp - ✔ Connected
claude.ai Google Drive: https://drivemcp.googleapis.com/mcp/v1 - ✔ Connected
plugin:playwright:playwright: npx @playwright/mcp@latest - ✔ Connected
plugin:figma:figma: https://mcp.figma.com/mcp (HTTP) - ! Needs authentication
plugin:postman:postman: https://mcp.postman.com/mcp (HTTP) - ! Needs authentication
```

같은 `https://mcp.notion.com/mcp`를 가리키는 서버인데도 한쪽에만 있다.

## 2. 환경

- Windows 11 Home (10.0.26200)
- Claude Code (CLI)
- Antigravity — Notion 플러그인 `0.1.0` (`antigravity-plugins-partners` 마켓플레이스)
- Notion MCP 서버: `https://mcp.notion.com/mcp` — HTTP transport, OAuth 인증
- 확인 시점: 2026-11-02

## 3. 원인을 좁혀나간 과정

### 3-1. 오진 ①: "설치 자체가 실패했다"

근거는 이 파일이었다.

```bash
$ ls -la ~/.gemini/config/mcp_config.json
-rw-r--r-- 1 pasun 197609 0 Nov  2 04:31 mcp_config.json
```

**0바이트.** MCP 설정 파일이 비어 있으니 설치가 안 된 것으로 읽었다.

**왜 틀렸나.** `~/.gemini/config/mcp_config.json`은 **사용자가 손으로 서버를 등록할 때 쓰는 파일**이고, 플러그인으로 설치한 MCP 서버는 여기에 쓰이지 않는다. 플러그인은 자기 디렉터리 안에 자기 설정을 들고 있다.

```bash
$ cat ~/.gemini/config/plugins/notion/mcp_config.json
{
  "mcpServers": {
    "notion": {
      "serverUrl": "https://mcp.notion.com/mcp"
    }
  }
}
```

설치 목록은 또 다른 파일에 있다.

```bash
$ cat ~/.gemini/config/config.json
{
  "plugins": {
    "notion": {
      "installedFrom": {
        "id": "antigravity-plugins-partners/plugin/notion",
        "marketplace": "antigravity-plugins-partners",
        "version": "0.1.0"
      }
    },
    ...
  }
}
```

즉 Antigravity에서 "MCP 서버가 뭐가 붙어 있나"를 알려면 **세 곳**을 봐야 한다.

| 경로 | 역할 |
|---|---|
| `~/.gemini/config/config.json` | 설치된 플러그인 목록 |
| `~/.gemini/config/plugins/<name>/mcp_config.json` | 플러그인이 등록하는 MCP 서버 |
| `~/.gemini/config/mcp_config.json` | 수동 등록용 (안 쓰면 0바이트) |

0바이트 파일 하나를 설치 여부의 근거로 삼은 게 오진의 전부였다.

### 3-2. 오진 ②: "한쪽에서 인증했으니 다른 쪽도 된다"

OAuth 엔드포인트가 같으니(`mcp.notion.com`) 한 번 브라우저 인증을 통과하면 다른 도구도 그 토큰을 쓸 거라고 생각했다.

**왜 틀렸나.** 설정만 분리된 게 아니라 **토큰 저장소도 분리**돼 있다. Antigravity 쪽 인증 데이터는 설정 디렉터리가 아니라 여기 쌓인다.

```bash
$ ls -la ~/.gemini/antigravity/plugin_data/notion/
total 0
drwxr-xr-x 1 pasun 197609 0 Nov  2 06:53 .
drwxr-xr-x 1 pasun 197609 0 Nov  2 06:53 ..
```

비어 있다 = 아직 인증 안 됨. 이 플러그인은 **첫 툴 호출 시점에** 브라우저 인증창을 띄우는 구조라, 설치만 해두고 한 번도 호출하지 않으면 계속 빈 디렉터리로 남는다. **"설치 완료"와 "사용 가능"이 다른 상태다.**

Claude Code 쪽은 아예 키가 없었다.

```bash
$ python -c "import json,io; print(json.load(io.open('.claude.json',encoding='utf-8')).get('mcpServers'))"
None
```

`~/.claude.json`에 `mcpServers` 키 자체가 없었다. 그러면 1번 항목의 `claude mcp list`에 뜬 다섯 개는 어디서 왔나 — **직접 등록한 게 하나도 없었던 것**이다. 둘은 claude.ai 커넥터(Claude Docs, Google Drive), 셋은 플러그인이 자동 등록한 것이다. `plugin:` 접두사가 그 표시다.

### 3-3. 오진 ③: "playwright 설정도 깨졌다"

작업 중에 이게 떴다.

```text
plugin:playwright:playwright (CONNECTION_CLOSED): "Connection closed"
```

다시 확인했을 때는 메시지가 바뀌었다.

```text
plugin:playwright:playwright: "Skipping connection (recent failure cached retries automatically in 15 min, or edit the plugin config to retry now)"
```

**왜 틀렸나.** 설정 문제가 아니었다. playwright는 `npx @playwright/mcp@latest`로 띄우는 **로컬 stdio 서버**라서, 패키지를 처음 내려받는 동안 핸드셰이크가 타임아웃될 수 있다. 그리고 한 번 실패하면 **15분간 재시도 결과를 캐시**해서, 그 사이에 목록을 다시 봐도 계속 실패로 보인다. 잠시 뒤 재확인했더니 정상이었다.

```text
plugin:playwright:playwright: npx @playwright/mcp@latest - ✔ Connected
```

원격 HTTP 서버와 로컬 stdio 서버는 실패 양상이 다르다. HTTP 서버의 실패는 보통 인증(`Needs authentication`), stdio 서버의 실패는 보통 프로세스 기동이다. 같은 "안 붙는다"로 묶으면 안 된다.

## 4. 왜 도구별로 분리되는가

MCP(Model Context Protocol)에서 **MCP 클라이언트는 각 AI 도구 자신**이다. Claude Code와 Antigravity는 서로 모르는 독립 프로세스이고, 각자 자기 설정 파일을 읽고 자기 토큰 저장소에 쓴다. 공유되는 건 서버 쪽 URL뿐이다.

OAuth 관점에서 보면 더 분명하다. 같은 `mcp.notion.com`을 보더라도 **클라이언트가 다르면 별개의 OAuth 클라이언트**다. 각각 따로 동의를 받고 따로 토큰을 받는다. 한쪽 토큰을 다른 쪽이 집어다 쓰면 그게 오히려 사고다.

정리하면 이렇다.

| | Antigravity | Claude Code |
|---|---|---|
| 설정 위치 | `~/.gemini/` | `~/.claude.json` |
| 플러그인 MCP | `config/plugins/<name>/mcp_config.json` | `plugin:<plugin>:<server>`로 자동 등록 |
| 수동 MCP | `config/mcp_config.json` | `mcpServers` 키 |
| OAuth 토큰 | `~/.gemini/antigravity/plugin_data/<name>/` | Claude Code 자체 저장소 |
| 인증 시점 | 첫 툴 호출 시 | `/mcp`에서 명시적으로 |

**도구 × 서버 조합마다 등록 1번 + 인증 1번이 필요하다.**

## 5. 직접 확인하는 법

Claude Code 쪽:

```bash
# 붙어 있는 서버와 연결 상태를 한 번에
$ claude mcp list

# 내가 직접 등록한 것만 (플러그인·커넥터 제외)
$ python -c "import json,io; print(json.load(io.open('.claude.json',encoding='utf-8')).get('mcpServers'))"
```

Antigravity 쪽:

```bash
# 설치된 플러그인
$ cat ~/.gemini/config/config.json

# 플러그인이 등록한 MCP 서버
$ cat ~/.gemini/config/plugins/<name>/mcp_config.json

# 인증 여부 — 비어 있으면 미인증
$ ls -la ~/.gemini/antigravity/plugin_data/<name>/
```

읽는 요령은 하나다. **`plugin:` 접두사가 붙어 있으면 내가 등록한 게 아니라 플러그인이 데려온 것**이고, `Needs authentication`은 등록은 됐고 인증만 안 된 상태다.

## 6. 해결

Claude Code 쪽에 따로 등록했다.

```bash
$ claude mcp add --transport http notion https://mcp.notion.com/mcp --scope user
Added HTTP MCP server notion with URL: https://mcp.notion.com/mcp to user config
File modified: C:\Users\pasun\.claude.json
```

`--scope user`는 이 머신의 모든 프로젝트에서 쓰겠다는 뜻이다. 특정 프로젝트 한정은 `--scope project`, 로컬 전용은 `local`.

등록 직후 상태는 "연결은 됐지만 인증 대기"다.

```text
notion: https://mcp.notion.com/mcp (HTTP) - ! Needs authentication
```

인증은 `/mcp`에서 한다. 서버를 고르면 브라우저로 Notion OAuth가 열리고, 워크스페이스 접근을 승인하면 끝난다.

```text
Authentication successful. Connected to notion.
```

이제 없던 키가 생겼다.

```text
mcpServers: {"notion": {"type": "http", "url": "https://mcp.notion.com/mcp"}}
```

최종 상태.

```bash
$ claude mcp list
claude.ai Claude Docs: ... - ✔ Connected
claude.ai Google Drive: ... - ✔ Connected
plugin:playwright:playwright: npx @playwright/mcp@latest - ✔ Connected
plugin:postman:postman: https://mcp.postman.com/mcp (HTTP) - ! Needs authentication
notion: https://mcp.notion.com/mcp (HTTP) - ✔ Connected
```

### 덤: 안 쓰는 플러그인은 서버째로 치운다

`plugin:` 접두사가 붙은 서버는 플러그인이 데려온 것이다. 그래서 **서버만 지울 수 없고, 플러그인을 지우면 서버와 스킬이 함께 사라진다.**

figma는 인프라 작업에 쓸 일이 없는데 세션마다 `Needs authentication` 경고로 남고 연결 시도 비용도 들어서 제거했다.

```bash
$ claude plugin uninstall figma@claude-plugins-official
✔ Successfully uninstalled plugin: figma (scope: user)
```

MCP 서버 1개와 스킬 14개가 같이 내려갔다. 되돌리려면 `claude plugin install figma@claude-plugins-official`.

## 7. 재발 방지

1. **어느 도구의 설정인지 먼저 정한다.** "MCP가 안 붙는다"는 질문은 도구를 특정하지 않으면 답이 없다.
2. **단일 파일을 근거로 "설치 안 됨"을 결론내지 않는다.** 플러그인 경로와 수동 등록 경로가 다르다.
3. **"설치됨"과 "인증됨"을 구분한다.** 빈 `plugin_data/<name>/`은 설치 실패가 아니라 미인증이다.
4. **stdio 서버의 첫 실패는 한 번 더 확인한다.** 재시도 캐시 때문에 같은 실패 메시지가 15분간 계속 보일 수 있다.
5. **쓰지 않는 플러그인은 지운다.** 인증 대기 경고가 쌓이면 진짜 고장이 묻힌다.

## 8. 정리

세 단계로 확인해봤다.

**설명** — MCP 설정이 도구별로 분리된 이유는, MCP 클라이언트가 각자 자기 설정 파일과 자기 토큰 저장소를 갖는 독립 프로세스이기 때문이다. 서버 URL이 같아도 클라이언트가 다르면 별개의 OAuth 클라이언트다.

**예측** — 지금 Gemini CLI에 Notion을 붙이면? 또 따로 해야 한다. `~/.gemini/`를 공유하더라도 Antigravity의 `plugin_data` 경로를 읽지 않으면 인증은 별개다.

**파괴** — `~/.claude.json`의 `mcpServers`에서 `notion` 항목만 지우면 `claude mcp list`에서 사라지고, 토큰은 남지만 참조할 서버가 없다. 반대로 항목은 두고 토큰만 지우면 `! Needs authentication`으로 되돌아간다. **두 증상이 다르다는 게 "등록"과 "인증"이 별개 상태라는 증거다.**

이번 건의 교훈은 기술적인 것보다 진단 습관 쪽이었다. 0바이트 파일 하나를 보고 "설치 실패"로 단정한 게 시작이었고, 설정 파일이 여러 곳에 나뉘어 있다는 전제를 세우자마자 5분 만에 끝났다.

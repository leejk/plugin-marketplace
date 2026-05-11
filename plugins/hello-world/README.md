# hello-world plugin

Claude Code 플러그인 강의용 예제. 한 플러그인 안에 **slash command, subagent,
skill, MCP 서버** 네 가지 기능을 모두 담아두었습니다.

```
plugins/hello-world/
├── .claude-plugin/plugin.json   # 플러그인 매니페스트
├── .mcp.json                    # MCP 서버 설정 (PyPI 패키지 버전 고정)
├── commands/hello.md            # /hello-world:hello — slash command
├── agents/welcome-bot.md        # welcome-bot — subagent
└── skills/code-greeter/SKILL.md # code-greeter — skill
```

설치는 마켓플레이스 등록 후:

```
/plugin marketplace add git@github.com:dimz119/plugin-marketpalce.git
/plugin install hello-world@my-plugins
/reload-plugins
```

---

## 1. Slash command — `/hello-world:hello`

**무엇을 하나**: 이름을 받아 인사하고, 현재 디렉터리와 git 브랜치를 찍어줍니다.

**사용법**:

```
/hello-world:hello Joon
```

**기대 출력**:

```
Hello, Joon! 👋

You are currently working in:
- 📁 Directory: /Users/seungjoonlee/git/plugin-marketpalce
- 🌿 Branch: main

Have a great coding session!
```

이름을 생략하면 `friend` 로 인사합니다.

---

## 2. Subagent — `welcome-bot`

**무엇을 하나**: 새 저장소에 들어왔을 때 프로젝트 종류, 기술 스택, 폴더 구조,
실행 방법을 250단어 이내로 요약해줍니다. 모델은 `haiku` 를 사용합니다.

**사용법** — 다음 중 어떤 표현이든 가능:

```
use welcome-bot to explain this project
welcome-bot 으로 이 코드베이스 한 번 정리해줘
이 레포 처음 보는데 welcome-bot 한테 가이드 시켜줘
```

또는 명시적으로 Task 도구를 호출하는 방식도 됩니다 (다른 에이전트 안에서 위임할 때):

```
Task(subagent_type="welcome-bot", prompt="이 레포 요약해줘")
```

**기대 출력**: 다음 5개 섹션이 있는 짧은 보고서.

- 🎯 What this project is
- 🛠️ Tech stack
- 📂 Key directories
- ▶️ To get started
- 💡 What to look at next

---

## 3. Skill — `code-greeter`

**무엇을 하나**: 소스 파일 맨 위에 표준화된 인사 코멘트 블록을 추가하거나
이미 있으면 `Updated:` 날짜만 갱신합니다. 언어별 주석 문법(`/* */`, `#`,
`<!-- -->`) 을 자동으로 골라요.

**사용법** — 트리거 문구 예시:

```
add a greeting to src/index.ts
apply code-greeter to README.md
code-greeter 로 main.py 헤더 좀 정리해줘
```

**기대 결과**: 파일 맨 위에 다음 블록이 생기거나 갱신됩니다.

```js
/*
👋 Hello from code-greeter!

File:    src/index.ts
Updated: 2026-05-04
*/
```

작업이 끝나면 한 줄로 `code-greeter applied to {filename}` 이라고 보고합니다.

> 적용하지 않는 파일: `.json`, `.lock`, `.toml`, 바이너리 (이미지/PDF 등).
> 그런 파일을 지정하면 적용을 거부하고 그 사실을 알려줍니다.

---

## 4. MCP servers — `time`, `fetch`

**무엇을 하나**: 플러그인을 설치하면 두 개의 PyPI 기반 MCP 서버가 자동으로
연결되어, Claude 에게 새 **도구(tool)** 가 생깁니다. 슬래시 커맨드가 아니라
**자연어로 요청하면 Claude 가 알아서 적절한 도구를 호출**하는 방식이에요.

| 서버 | PyPI 패키지 | Claude 한테 노출되는 도구 |
|------|------------|--------------------------|
| `time`  | `mcp-server-time==0.6.2`  | `get_current_time`, `convert_time` |
| `fetch` | `mcp-server-fetch==2025.4.7` | `fetch` (URL → 페이지 내용) |

### 사전 준비

`uvx` 가 필요합니다 (없으면 MCP 서버가 시작되지 않아요):

```bash
brew install uv      # macOS
# 또는
pipx install uv      # 그 외 OS
```

설치 직후 `/mcp` 로 서버 상태가 `connected` 인지 확인하세요. 첫 실행에는
`uvx` 가 PyPI 에서 패키지를 받아오기 때문에 몇 초 걸릴 수 있습니다.

### 4-1. `time` 사용법

`time` 서버는 **타임존 변환과 현재 시간 조회만** 합니다. URL 이나 일반
질문을 주면 안 됩니다.

자연어로 이렇게 요청하세요:

```
지금 서울 몇 시야?
현재 도쿄 시간 알려줘
오후 3시 (Asia/Seoul) 를 New York 시간으로 변환해줘
What time is it in Europe/London right now?
```

Claude 가 호출하는 도구:

- `get_current_time(timezone="Asia/Seoul")`
- `convert_time(source_timezone, time, target_timezone)`

`timezone` 은 IANA 이름(`Asia/Seoul`, `America/New_York`, `Europe/London`,
`UTC` 등) 을 사용합니다. `KST`, `JST` 같은 약어는 동작하지 않을 수 있어요.

### 4-2. `fetch` 사용법

`fetch` 서버는 **URL을 받아 그 페이지 내용을 가져옵니다**. 시간 질문이나
일반 문장을 넣으면 "URL이 아니다" 라는 에러가 나요 (이번 강의 데모에서
실제로 본 그 에러입니다).

자연어로 이렇게 요청하세요:

```
https://example.com 페이지 내용 가져와서 요약해줘
https://news.ycombinator.com 에서 오늘 1면 제목 5개 뽑아줘
fetch this page: https://docs.anthropic.com/claude/docs
```

Claude 가 호출하는 도구:

- `fetch(url="https://...", max_length=5000, raw=false)`

기본값은 HTML → markdown 으로 정리해서 반환합니다. 원본 HTML 이 필요하면
"raw 로 가져와줘" 라고 말하세요.

### 자주 하는 실수

| 잘못된 호출 | 왜 실패하는가 |
|-----------|--------------|
| `/mcp__plugin_hello-world_fetch__fetch "현재 서울 시간 알려줘"` | `fetch` 는 URL 전용. 시간은 `time` 서버 담당 |
| `get_current_time(timezone="KST")` | 약어 미지원 — `Asia/Seoul` 사용 |
| `fetch("example.com")` | 스킴 누락 — `https://example.com` 처럼 `http(s)://` 필요 |

> 일반적으로는 위 두 서버의 도구를 **직접 슬래시로 호출하지 말고**, 그냥
> 자연어로 부탁하면 Claude 가 알아서 도구를 고릅니다. 슬래시 호출
> (`/mcp__...`) 은 디버깅용이라고 생각하세요.

---

## MCP 버전 컨트롤 (왜 `==0.6.2` 처럼 적나)

`.mcp.json` 안의 `args` 를 보면 패키지명에 `==<version>` 이 붙어 있어요:

```json
"args": ["mcp-server-time==0.6.2", "--local-timezone=Asia/Seoul"]
```

| 표기 | 동작 | 재현성 |
|------|------|--------|
| `uvx mcp-server-time` | 항상 최신 버전 | ❌ 사용자마다 달라질 수 있음 |
| `uvx mcp-server-time==0.6.2` | 정확히 그 버전 | ✅ 강의/배포에 권장 |
| `uvx mcp-server-time@latest` | 매 실행마다 최신 확인 | ⚠️ 느림, 깨질 수 있음 |

플러그인 사용자는 `npm`/`pip` 같은 별도 의존성 관리를 하지 않습니다.
`uvx` 가 매니페스트의 `==<version>` 만 보고 격리된 환경에 그 버전을 받아와
실행해 줘요. 그래서 `.mcp.json` 한 파일이 사실상의 lock 파일 역할을 합니다.

> npm 기반 MCP 서버를 추가하고 싶다면 같은 패턴이 그대로 적용됩니다:
> `"command": "npx", "args": ["-y", "@scope/pkg@1.2.3"]`.

---

## 한꺼번에 시연하는 흐름 (강의용)

1. `/plugin marketplace add git@github.com:dimz119/plugin-marketpalce.git`
2. `/plugin install hello-world@my-plugins` → `/reload-plugins`
3. `/hello-world:hello Joon` — slash command 동작 확인
4. "use welcome-bot to explain this project" — subagent 동작 확인
5. "apply code-greeter to README.md" — skill 동작 확인
6. `/mcp` 로 `time`, `fetch` 가 connected 인지 확인
7. "지금 서울 몇 시야?" → `time` 서버가 호출되는지 확인
8. "https://example.com 가져와서 요약해줘" → `fetch` 서버가 호출되는지 확인
9. `.mcp.json` 의 `==0.6.2` 부분을 보여주며 "이게 PyPI 버전 고정" 이라고 설명

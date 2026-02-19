# Claude Code Permission 시스템

> **요약**: Claude Code의 permission 시스템은 tool 허용/거부 규칙, 5가지 모드, sandbox를 조합해 세밀한 보안 제어를 제공한다.
> **출처**: https://code.claude.com/docs/en/permissions · https://code.claude.com/docs/en/sandboxing · https://code.claude.com/docs/en/security
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-19

---

## 1. 기본 구조

Claude Code는 3단계 권한 시스템을 사용한다:

| Tool 유형 | 예시 | 기본 동작 |
|-----------|------|-----------|
| Read-only | 파일 읽기, Grep | 자동 허용 |
| Bash 명령 | Shell 실행 | 승인 필요 |
| 파일 수정 | Edit/Write | 세션 종료까지 승인 후 자동 허용 |

---

## 2. Permission 관리 방법

### `/permissions` 커맨드
- 현재 모든 permission 규칙과 적용 파일 위치를 UI로 표시
- 규칙 추가/삭제/확인 가능

### `settings.json` 직접 설정
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ],
    "ask": [
      "Bash(git push *)"
    ]
  }
}
```

**평가 순서**: `deny` → `ask` → `allow` (먼저 매칭되는 규칙 적용)

---

## 3. Permission Modes

`defaultMode`로 설정:

| 모드 | 동작 | 사용 상황 |
|------|------|-----------|
| `default` | 첫 사용 시 확인 프롬프트 | 일반 개발 |
| `acceptEdits` | 파일 편집 자동 허용 | 파일 수정이 많을 때 |
| `plan` | 분석만 허용, 수정/실행 불가 | Plan Mode |
| `dontAsk` | 사전 허용된 것만 허용 | 엄격한 제어 |
| `bypassPermissions` | 모든 확인 생략 ⚠️ | 격리된 환경에서만 |

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

---

## 4. Permission Rule 문법

### Tool별 기본 문법
```
Bash(npm run *)          → npm run으로 시작하는 명령
Bash(git * main)         → git X main 형태의 명령
Read(./.env)             → 현재 디렉토리의 .env 파일
Edit(/src/**)            → settings 파일 기준 src/ 하위 전체
WebFetch(domain:ex.com)  → example.com 요청
mcp__puppeteer           → puppeteer MCP 서버의 모든 tool
Task(Explore)            → Explore subagent
```

### 경로 패턴 4종

| 패턴 | 의미 | 예시 |
|------|------|------|
| `//path` | 절대 경로 | `Read(//Users/alice/secrets/**)` |
| `~/path` | 홈 디렉토리 기준 | `Read(~/.zshrc)` |
| `/path` | settings 파일 기준 상대 | `Edit(/src/**/*.ts)` |
| `path` 또는 `./path` | 현재 작업 디렉토리 기준 | `Read(*.env)` |

> ⚠️ `/Users/alice/file`은 절대 경로가 아님. 절대 경로는 `//Users/alice/file`

### 와일드카드 주의사항
- `Bash(ls *)` → `ls -la` 매칭 O, `lsof` 매칭 X (공백 뒤 `*`는 단어 경계)
- `Bash(ls*)` → `ls -la`, `lsof` 모두 매칭
- curl 같은 명령은 URL 패턴으로 제한하기 어려움 → deny 후 WebFetch 사용 권장

---

## 5. Settings 우선순위

```
Managed settings (최고, 오버라이드 불가)
  ↓
Command line arguments
  ↓
.claude/settings.local.json (로컬)
  ↓
.claude/settings.json (프로젝트 공유)
  ↓
User settings (최하위)
```

---

## 6. Sandbox

### 개념
OS 레벨에서 Bash 명령의 파일시스템 + 네트워크 접근을 격리.
Permission 규칙(Claude 레벨)과 별개의 보안 레이어.

### 활성화
```
> /sandbox
```
메뉴에서 모드 선택.

### 지원 플랫폼
| OS | 기술 |
|----|------|
| macOS | Seatbelt |
| Linux | bubblewrap + socat |
| WSL2 | bubblewrap + socat |
| WSL1 | ❌ 미지원 |

Linux/WSL2 설치:
```bash
sudo apt-get install bubblewrap socat  # Ubuntu/Debian
sudo dnf install bubblewrap socat      # Fedora
```

### Sandbox 모드 2가지
- **Auto-allow 모드**: sandbox 내 명령 자동 승인, 밖은 일반 permission 흐름
- **Regular permissions 모드**: sandbox 내도 일반 승인 절차 유지

### settings.json 설정
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker", "git"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "allowUnixSockets": ["~/.ssh/agent-socket"],
      "allowLocalBinding": true
    }
  }
}
```

### Permission vs Sandbox
| 구분 | Permission | Sandbox |
|------|-----------|---------|
| 레벨 | Claude 의사결정 | OS 강제 |
| 대상 | 모든 tool | Bash만 |
| 역할 | 허용/거부 결정 | 경계 강제 |
| 관계 | 보완적 (함께 사용) | 보완적 |

---

## 7. 핵심 포인트

- **deny → ask → allow** 순서로 평가, 먼저 매칭된 규칙 적용
- `bypassPermissions` 모드는 격리 환경에서만 — 잘못 쓰면 모든 보안 무력화
- Sandbox는 Permission과 별개: 둘 다 설정해야 defense-in-depth
- curl/wget은 URL 패턴 제한이 어려움 → deny 후 WebFetch로 대체
- `/permissions` UI에서 현재 규칙 전체를 파일 위치와 함께 확인 가능
- sandbox에서 `docker`, `watchman`은 호환 안 됨 → `excludedCommands` 설정

---

## 메타

- 관련 문서: [[claude-md-guide.md]], [[hooks.md]]
- 태그: #permissions #sandbox #security #settings

# Claude Code Hooks 완벽 가이드

> **요약**: Hooks는 Claude Code 라이프사이클의 특정 시점에 자동 실행되는 shell 명령어 또는 LLM prompt다. 파일 편집 후 자동 포매팅, 위험 명령 차단, 알림 전송 등을 결정론적으로 보장한다.
> **출처**: https://code.claude.com/docs/en/hooks , https://code.claude.com/docs/en/hooks-guide
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-19

---

## 1. Hooks란 무엇인가

Hooks는 사용자가 정의한 **shell 명령어 또는 LLM prompt**로, Claude Code의 라이프사이클 중 특정 시점에 자동 실행된다.

**핵심**: LLM이 판단하여 실행하는 것이 아니라, **결정론적(deterministic)으로 항상 실행**된다.

### 주요 사용 사례

- 파일 편집 후 자동 포매팅 (Prettier, ESLint)
- 위험한 명령어 차단 (`rm -rf` 등)
- 알림 전송 (Claude가 입력 대기 중일 때)
- 보호된 파일 수정 차단
- 세션 시작 시 컨텍스트 주입
- 컨텍스트 압축 후 중요 정보 재주입

---

## 2. 14가지 이벤트 타입

| 이벤트 | 발동 시점 | Blocking (exit 2) |
|--------|-----------|-------------------|
| **`SessionStart`** | 세션 시작/재개 시 | No |
| **`UserPromptSubmit`** | 사용자 prompt 제출 시, Claude 처리 전 | Yes |
| **`PreToolUse`** | 도구 호출 실행 전 | Yes |
| **`PermissionRequest`** | 권한 요청 다이얼로그 표시 시 | Yes |
| **`PostToolUse`** | 도구 호출 성공 후 | No |
| **`PostToolUseFailure`** | 도구 호출 실패 후 | No |
| **`Notification`** | Claude가 알림을 보낼 때 | No |
| **`SubagentStart`** | 서브에이전트 생성 시 | No |
| **`SubagentStop`** | 서브에이전트 완료 시 | Yes |
| **`Stop`** | Claude가 응답 완료 시 | Yes |
| **`TeammateIdle`** | 팀원이 idle 상태가 될 때 | Yes |
| **`TaskCompleted`** | Task 완료 표시 시 | Yes |
| **`PreCompact`** | 컨텍스트 압축 전 | No |
| **`SessionEnd`** | 세션 종료 시 | No |

### 주요 이벤트 상세

**`PreToolUse`**: 도구 호출 직전. `permissionDecision`으로 `allow`, `deny`, `ask` 결정. `updatedInput`으로 도구 입력 수정도 가능.

**`PostToolUse`**: 도구 성공 후. 되돌릴 수 없지만 Claude에게 피드백 제공 가능.

**`Stop`**: Claude 응답 완료 시. `stop_hook_active` 필드로 무한 루프 방지 필수. `decision: "block"`으로 계속 작업하게 할 수 있다.

**`SessionStart`**: matcher 값은 `startup`, `resume`, `clear`, `compact`. stdout이 Claude 컨텍스트에 추가됨.

---

## 3. 세 가지 Hook 타입: Command vs Prompt vs Agent

### Command Hook (`type: "command"`)

```json
{
  "type": "command",
  "command": "./my-script.sh",
  "timeout": 600,
  "async": false
}
```

- Shell 명령어 실행
- stdin으로 JSON 입력, exit code + stdout/stderr로 결과 반환
- `async: true` 설정 시 백그라운드 비차단 실행

### Prompt Hook (`type: "prompt"`)

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude should stop: $ARGUMENTS. Check if all tasks are complete.",
  "model": "haiku",
  "timeout": 30
}
```

- LLM(기본: Haiku)에 prompt를 보내 단일 턴 평가
- `$ARGUMENTS` placeholder로 hook 입력 JSON 주입
- 응답: `{"ok": true}` 또는 `{"ok": false, "reason": "..."}`
- `TeammateIdle`에서는 미지원

### Agent Hook (`type: "agent"`)

```json
{
  "type": "agent",
  "prompt": "Verify that all unit tests pass. $ARGUMENTS",
  "timeout": 120
}
```

- 서브에이전트 생성하여 Read, Grep, Glob 등 도구 사용
- 최대 50턴 도구 사용 가능
- 실제 코드베이스 상태를 확인해야 할 때 사용

---

## 4. Hook 설정 방법

### 설정 파일 위치

| 위치 | 범위 | 공유 |
|------|------|------|
| `~/.claude/settings.json` | 모든 프로젝트 | No |
| `.claude/settings.json` | 단일 프로젝트 | Yes (커밋 가능) |
| `.claude/settings.local.json` | 단일 프로젝트 | No (gitignored) |
| Managed policy settings | 조직 전체 | Yes |
| Skill/Agent frontmatter | 컴포넌트 활성화 중 | Yes |

### 설정 구조 (3단계 중첩)

```json
{
  "hooks": {
    "이벤트명": [
      {
        "matcher": "패턴",
        "hooks": [
          {
            "type": "command",
            "command": "스크립트 경로",
            "timeout": 600,
            "statusMessage": "실행 중 메시지"
          }
        ]
      }
    ]
  }
}
```

**구조**: `이벤트명` → `matcher group` (필터링) → `hook handler` 배열

---

## 5. 입력(stdin)과 출력(stdout) 형식

### 공통 입력 (모든 이벤트)

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/home/user/my-project",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse"
}
```

### 이벤트별 추가 입력

**PreToolUse**: `tool_name`, `tool_input`, `tool_use_id`
**PostToolUse**: `tool_name`, `tool_input`, `tool_response`
**Stop**: `stop_hook_active`, `last_assistant_message`

### JSON 출력 형식

| 필드 | 기본값 | 설명 |
|------|--------|------|
| `continue` | `true` | `false`이면 Claude 완전히 멈춤 |
| `stopReason` | - | `continue: false`일 때 표시 메시지 |
| `suppressOutput` | `false` | stdout 숨김 여부 |

**PreToolUse 전용** (`hookSpecificOutput`):

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "위험한 명령어입니다",
    "updatedInput": { "command": "safer-command" }
  }
}
```

---

## 6. Matcher 문법

Matcher는 **정규식(regex) 문자열**로, hook 발동 조건을 필터링한다.

### 기본 규칙

- `"*"`, `""`, 또는 생략 → 모든 경우 매칭
- 정규식 지원: `Edit|Write`는 Edit 또는 Write 매칭
- 대소문자 구분 (case-sensitive)

### 이벤트별 매칭 대상

| 이벤트 | 매칭 대상 | 예시 |
|--------|-----------|------|
| `PreToolUse`, `PostToolUse` | tool name | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | 시작 방법 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | 종료 이유 | `clear`, `logout` |
| `Notification` | 알림 타입 | `permission_prompt`, `idle_prompt` |
| `SubagentStart/Stop` | 에이전트 타입 | `Bash`, `Explore`, `Plan` |
| `Stop`, `UserPromptSubmit` | matcher 미지원 | 항상 발동 |

### MCP 도구 매칭

`mcp__<server>__<tool>` 형식:
- `mcp__memory__.*` → memory 서버의 모든 도구
- `mcp__.*__write.*` → 모든 서버의 write 관련 도구

---

## 7. Exit Code와 실행 흐름

### Exit Code

| Exit Code | 의미 | 동작 |
|-----------|------|------|
| **0** | 성공 | stdout JSON 파싱, 정상 진행 |
| **2** | 차단(blocking) | stderr가 Claude에게 에러로 전달, 도구/동작 차단 |
| **기타** | 비차단 에러 | stderr는 verbose 모드에서만 표시, 실행 계속 |

### 실행 특성

- 매칭되는 모든 hook은 **병렬 실행**
- 동일 handler는 **자동 중복 제거**
- Hook 설정은 **시작 시 스냅샷** 캡처 (세션 중 외부 수정 시 경고)

---

## 8. Hooks in Skills

Skill/Agent의 YAML frontmatter에서 hooks를 직접 정의할 수 있다. 컴포넌트가 활성화된 동안에만 실행되고, 완료 시 자동 정리된다.

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

### 주요 특성

- settings 기반 hook과 동일한 형식
- Subagent에서 `Stop` hook → 자동으로 `SubagentStop`으로 변환
- `once: true` 옵션으로 세션당 한 번만 실행 (Skills 전용)

---

## 9. 보안 고려사항

**핵심 경고**: Hook은 시스템 사용자의 **전체 권한**으로 실행된다.

### 보안 모범 사례

1. **입력 검증**: 입력 데이터를 맹목적으로 신뢰하지 말 것
2. **Shell 변수 따옴표 처리**: `"$VAR"` 사용 (`$VAR` 아님)
3. **경로 순회 차단**: 파일 경로에서 `..` 확인
4. **절대 경로 사용**: `"$CLAUDE_PROJECT_DIR"` 활용
5. **민감한 파일 제외**: `.env`, `.git/`, 키 파일 등
6. **Enterprise 관리**: `allowManagedHooksOnly`로 사용자/프로젝트 hook 차단 가능

---

## 10. 실용적인 예제

### 예제 1: 파일 편집 후 자동 포매팅 (Prettier)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### 예제 2: 위험한 명령어 차단

`.claude/hooks/block-rm.sh`:
```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0
fi
```

설정:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/block-rm.sh" }
        ]
      }
    ]
  }
}
```

### 예제 3: 데스크톱 알림 (macOS)

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 예제 4: 보호된 파일 수정 차단

`.claude/hooks/protect-files.sh`:
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

---

## 디버깅

| 방법 | 설명 |
|------|------|
| `claude --debug` | hook 실행 상세 로그 |
| `Ctrl+O` | verbose 모드 토글 |
| `/hooks` | 대화형 hook 관리 메뉴 |

## 유용한 환경변수

| 변수 | 설명 |
|------|------|
| `$CLAUDE_PROJECT_DIR` | 프로젝트 루트 경로 |
| `$CLAUDE_ENV_FILE` | SessionStart에서 환경변수 영구화 파일 경로 |
| `$CLAUDE_CODE_REMOTE` | 원격 웹 환경에서 `"true"` |

---

## 핵심 포인트

- Hooks는 결정론적 실행 — LLM 판단이 아닌 항상 실행되는 보장
- 3가지 타입: command (셸), prompt (LLM 단일턴), agent (도구 사용 서브에이전트)
- Exit code 2로 도구 호출, prompt 처리, 응답 완료 등을 차단 가능
- Matcher는 정규식으로 도구/이벤트 필터링
- Skill/Agent frontmatter에서 스코프된 hook 정의 가능
- 보안 주의: 전체 사용자 권한으로 실행됨

## 메타

- 관련 문서: [[skills.md]], [[permissions.md]], [[context-management.md]]
- 태그: #hooks #automation #lifecycle #security #formatting

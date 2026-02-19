# Claude Code Context 관리

> **요약**: Context window를 효율적으로 관리해 비용을 줄이고 성능을 유지하는 방법. /clear, /compact, status line, auto-compaction이 핵심 도구다.
> **출처**: https://code.claude.com/docs/en/costs · https://code.claude.com/docs/en/statusline
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-19

---

## 1. Context Window 기본 개념

- **크기**: 기본 200,000 토큰 (일부 모델은 1,000,000)
- **비용 원리**: context가 클수록 → 매 메시지마다 더 많은 토큰 소비
- **자동 최적화**: Prompt caching + Auto-compaction이 자동으로 동작
- **평균 비용**: $6/개발자/일 (90%는 $12 이하)

---

## 2. 핵심 커맨드

### `/clear` — 컨텍스트 초기화
- 새 작업으로 전환할 때 사용 → 이전 대화가 다음 메시지 비용을 낭비하지 않도록
- **패턴**: 세션 전환 전 `/rename`으로 이름 저장 → `/clear` → 나중에 `/resume`으로 복귀
- **언제**: 무관한 작업으로 넘어갈 때, 대화가 잘못된 방향으로 흘렀을 때

### `/compact [지시]` — 컨텍스트 압축
- 대화 히스토리를 요약해 토큰 절감
- **커스텀 지시**: `/compact Focus on code samples and API usage`
- **CLAUDE.md에서 영구 설정**:
  ```markdown
  # Compact instructions
  When you are using compact, please focus on test output and code changes
  ```

### `/cost` — 현재 세션 비용 확인
```
Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    0 lines added, 0 lines removed
```
> API 사용자용. Max/Pro 구독자는 `/stats`로 사용 패턴 확인.

### `/context` — 컨텍스트 점유 현황 확인
- MCP 서버 등 무엇이 공간을 차지하는지 파악

---

## 3. Status Line으로 토큰 실시간 추적

### 설정 방법 1: `/statusline` 커맨드 (가장 쉬움)
```
/statusline show model name and context percentage with a progress bar
```
Claude가 스크립트를 자동 생성 + 설정 파일 업데이트.

### 설정 방법 2: `settings.json` 직접 설정
```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2
  }
}
```

### 스크립트 예시 (Bash)
```bash
#!/bin/bash
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

FILLED=$((PCT * 10 / 100)); EMPTY=$((10 - FILLED))
BAR=$(printf "%${FILLED}s" | tr ' ' '▓')$(printf "%${EMPTY}s" | tr ' ' '░')

echo "[$MODEL] $BAR $PCT%"
```

### Status line에서 사용 가능한 주요 데이터
| 필드 | 설명 |
|------|------|
| `model.display_name` | 현재 모델 이름 |
| `context_window.used_percentage` | 컨텍스트 사용률 (%) |
| `context_window.context_window_size` | 최대 토큰 수 |
| `cost.total_cost_usd` | 세션 누적 비용 |
| `cost.total_duration_ms` | 총 경과 시간 (ms) |
| `workspace.current_dir` | 현재 작업 디렉토리 |

> Status line은 로컬 실행 — API 토큰 소비 없음

---

## 4. 토큰 절감 전략 (우선순위 순)

### 즉각 효과 (바로 실천)
1. **작업 전환 시 `/clear`** — 가장 간단하고 효과 큰 방법
2. **구체적인 프롬프트** — "이 코드베이스 개선해줘" X → "auth.ts의 login 함수에 input validation 추가" O
3. **Plan Mode 활용** — `Shift+Tab`으로 탐색 후 구현 → 잘못된 방향 재작업 방지

### 구조적 최적화
4. **CLAUDE.md → Skills로 이전** — CLAUDE.md는 세션 시작부터 로드됨. 특정 워크플로우 지시는 Skills로 분리 → 필요할 때만 로드 (목표: CLAUDE.md 500줄 이하)
5. **MCP 서버 최소화** — `/mcp`로 미사용 서버 비활성화. 각 서버가 tool 정의를 context에 항상 추가

### 고급 최적화
6. **Subagent에 위임** — 로그 분석, 문서 fetch 등 verbose 작업을 subagent에서 실행 → 결과 요약만 메인 context에 전달
7. **Hooks으로 전처리** — 10,000줄 로그 파일을 Claude가 읽기 전에 hook으로 ERROR 줄만 필터링
8. **Extended thinking 조정** — Opus의 thinking budget은 기본 31,999 토큰 (output 토큰으로 과금). 단순 작업 시 낮추거나 비활성화

---

## 5. Auto-compaction

Claude Code가 context limit에 도달하면 **자동으로 대화를 요약**한다.

- 자동 동작 — 별도 설정 불필요
- `/compact`로 수동 트리거 가능 (커스텀 지시 포함 가능)
- compaction 후에도 작업 연속성 유지

---

## 6. Course Correction (잘못된 방향 수정)

| 상황 | 해결 |
|------|------|
| Claude가 엉뚱한 방향으로 가는 중 | `Esc`로 즉시 중단 |
| 이미 실수를 했을 때 | `Esc+Esc` (더블 탭) 또는 `/rewind`로 이전 체크포인트로 복귀 |
| 복잡한 작업 시작 전 | `Shift+Tab`으로 Plan Mode → 탐색 후 구현 |

> `Esc` 중단이 빠를수록 낭비하는 토큰이 적음 — 잘못됐다 싶으면 즉시 멈춰라

---

## 7. 핵심 포인트

- **Context = 비용**: context가 클수록 매 메시지 비용 증가. 적극적으로 관리해야 함
- **`/clear` 습관화**: 작업 전환 = `/clear`. 가장 쉽고 효과적인 절감법
- **Status line**: 컨텍스트 사용률을 항상 보이게 — `/statusline`으로 30초 설정
- **MCP 서버 정리**: 안 쓰는 서버가 context를 계속 먹음
- **CLAUDE.md 500줄 이하**: 초과 분은 Skills로 분리
- **Subagent 활용**: verbose output은 subagent에서 → 요약만 메인으로

---

## 메타

- 관련 문서: [[permissions.md]], [[prompt-caching.md]]
- 태그: #context #tokens #cost #statusline #optimization

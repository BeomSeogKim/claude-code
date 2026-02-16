# Prompt Caching: Claude API 비용 최적화의 핵심

> **요약**: Prompt Caching은 반복되는 프롬프트 내용을 캐싱하여 처리 시간과 비용을 획기적으로 절감하는 기능 (최대 90% 비용 절감, 85% 속도 향상)
> **출처**: Anthropic 공식 문서
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-16

## 핵심 개념

Prompt Caching은 **자주 사용되는 프롬프트 내용을 캐시에 저장**하여, 동일한 내용을 반복 처리하지 않도록 하는 기능입니다.

### 동작 원리

1. **첫 번째 요청**: 프롬프트를 처리하고 지정된 위치까지 캐시에 저장
2. **후속 요청**: 캐시된 내용을 찾아 재사용 (재처리 없음)
3. **캐시 갱신**: 캐시 사용 시마다 5분 수명 자동 연장

**내부 메커니즘**:
- 모델은 프롬프트를 읽을 때 "attention states" (단어 간 연결 맵) 생성
- 캐싱 없이는 매번 재구성 필요
- 캐싱 활성화 시 이 맵을 저장하고 재사용

---

## 비용 구조

### 가격표 (Claude Sonnet 4.5 기준)

| 토큰 유형 | 비용 (백만 토큰당) | 배수 |
|-----------|-------------------|------|
| 일반 Input | $3 | 1.0x |
| 5분 Cache Write | $3.75 | 1.25x |
| 1시간 Cache Write | $6 | 2.0x |
| Cache Read (히트) | $0.30 | **0.1x** |
| Output | $15 | - |

**핵심**: Cache Read는 일반 Input의 **10%만 과금** → 최대 90% 절감

### 실제 비용 예시

**시나리오**: 100,000 토큰 문서 + 50 토큰 질문 (Claude Sonnet 4.5)

| 요청 | Cache Write | Cache Read | Input | 총 비용 |
|------|-------------|-----------|-------|---------|
| 1차 (캐시 생성) | 100,000 ($0.375) | 0 | 50 ($0.00015) | $0.37515 |
| 2차 (캐시 히트) | 0 | 100,000 ($0.03) | 50 ($0.00015) | $0.03015 |

**절감 효과**: 2차 요청부터 92% 비용 절감 ($0.375 → $0.03)

---

## 구현 방법

### 1. 기본 사용법

`cache_control` 파라미터를 사용하여 캐시 breakpoint 지정:

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are an AI assistant.",
        },
        {
            "type": "text",
            "text": "<전체 문서 내용>",
            "cache_control": {"type": "ephemeral"}  # 여기까지 캐싱
        }
    ],
    messages=[
        {"role": "user", "content": "문서에 대해 질문"}
    ]
)
```

### 2. 캐시 가능 최소 토큰 수

| 모델 | 최소 토큰 |
|------|-----------|
| Claude Opus 4.6, 4.5 | 4096 |
| Claude Sonnet 4.5, 4, Opus 4.1, 4 | 1024 |
| Claude Haiku 4.5 | 4096 |
| Claude Haiku 3.5, 3 | 2048 |

**중요**: 최소 토큰 미만은 `cache_control` 표기해도 캐싱 안 됨

### 3. 캐시 Breakpoint 계층 구조

캐시는 다음 순서로 구성되며, 각 레벨은 상위 레벨을 포함:

```
tools → system → messages
```

**최대 4개 breakpoint** 사용 가능:

```python
client.messages.create(
    model="claude-sonnet-4-5-20250929",
    tools=[
        {"name": "tool1", ...},
        {"name": "tool2", ..., "cache_control": {"type": "ephemeral"}}  # BP1: tools 전체
    ],
    system=[
        {"type": "text", "text": "지침", "cache_control": {"type": "ephemeral"}},  # BP2: 지침
        {"type": "text", "text": "문서", "cache_control": {"type": "ephemeral"}}   # BP3: 문서
    ],
    messages=[
        {"role": "user", "content": "질문"},
        {"role": "assistant", "content": "답변"},
        {"role": "user", "content": [
            {"type": "text", "text": "후속 질문", "cache_control": {"type": "ephemeral"}}  # BP4: 대화
        ]}
    ]
)
```

### 4. 자동 prefix 체크

**핵심**: breakpoint 1개만 설정해도, 시스템이 자동으로 이전 블록들을 역순으로 체크 (최대 20블록)

```
블록 30에 cache_control 설정 → 시스템이 30→29→28...→11 순으로 체크
→ 가장 긴 일치 시퀀스 자동 재사용
```

**20블록 제한**:
- 20블록 이상 떨어진 곳에 변경사항 있으면 캐시 미스
- 해결: 편집 가능한 내용 직전에 명시적 breakpoint 추가

---

## 캐시 무효화 조건

| 변경 사항 | Tools | System | Messages |
|-----------|-------|--------|----------|
| Tool 정의 수정 | ✘ | ✘ | ✘ |
| Web search 토글 | ✓ | ✘ | ✘ |
| Citations 토글 | ✓ | ✘ | ✘ |
| Speed 설정 변경 | ✓ | ✘ | ✘ |
| tool_choice 변경 | ✓ | ✓ | ✘ |
| 이미지 추가/제거 | ✓ | ✓ | ✘ |
| Thinking 설정 변경 | ✓ | ✓ | ✘ |

✘ = 캐시 무효화, ✓ = 캐시 유지

---

## 캐시 성능 추적

API 응답의 `usage` 필드:

```json
{
  "usage": {
    "input_tokens": 50,                      // 마지막 BP 이후 토큰 (캐시 미대상)
    "cache_creation_input_tokens": 100000,   // 새로 캐시에 쓴 토큰
    "cache_read_input_tokens": 0             // 캐시에서 읽은 토큰
  }
}
```

**총 input 토큰 계산**:
```
total_input = cache_read + cache_creation + input_tokens
```

**주의**: `input_tokens`은 전체 입력이 아니라 **마지막 breakpoint 이후** 토큰만 표시!

---

## 1시간 캐시 (옵션)

기본 5분이 짧으면 1시간 TTL 사용 가능 (추가 비용):

```python
"cache_control": {
    "type": "ephemeral",
    "ttl": "1h"  # 또는 "5m" (기본값)
}
```

### 사용 시나리오
- 5분 이상 걸리는 에이전트 작업
- 사용자 응답 대기 시간이 긴 경우
- 레이턴시가 중요한 경우
- Rate limit 활용 최적화 (cache hit은 rate limit에 미포함)

### 혼합 사용 규칙
**1시간 breakpoint는 5분 breakpoint보다 앞에 와야 함**

```python
system=[
    {"text": "지침", "cache_control": {"type": "ephemeral", "ttl": "1h"}},   # 먼저
    {"text": "문서", "cache_control": {"type": "ephemeral", "ttl": "5m"}}    # 나중
]
```

---

## 사용 사례별 최적화

### 1. 대화형 에이전트 (Claude Code)

**전략**: 시스템 프롬프트 + 대화 이력 캐싱

```python
# 매 턴마다 마지막 메시지에 cache_control 추가
messages=[
    {"role": "user", "content": "질문 1"},
    {"role": "assistant", "content": "답변 1"},
    {"role": "user", "content": [
        {"type": "text", "text": "질문 2", "cache_control": {"type": "ephemeral"}}
    ]}
]
```

**효과**: 대화가 길어질수록 누적 캐싱 → 지연 및 비용 대폭 감소

### 2. 코딩 어시스턴트

**전략**: 코드베이스 요약/주요 섹션 캐싱

```python
system=[
    {"type": "text", "text": "프로젝트 구조 및 주요 파일 요약",
     "cache_control": {"type": "ephemeral"}}
]
```

**효과**: 자동완성, Q&A에서 일관된 컨텍스트 유지 + 빠른 응답

### 3. 대용량 문서 처리

**전략**: 전체 문서(이미지 포함) 캐싱

```python
system=[
    {"type": "text", "text": "긴 법률 문서 전문 (50페이지)",
     "cache_control": {"type": "ephemeral"}}
]
```

**효과**: 레이턴시 증가 없이 전체 문서 참조 가능

### 4. Tool Use (Agentic 워크플로우)

**전략**: Tool 정의 캐싱

```python
tools=[
    {"name": "search", ...},
    {"name": "retrieve", ...},
    {"name": "analyze", ..., "cache_control": {"type": "ephemeral"}}  # 마지막 tool에 표기
]
```

**효과**: 여러 tool call 및 반복 코드 변경 시나리오에서 성능 향상

### 5. Few-shot 예제 (20+ 예제)

**전략**: 대량 예제 캐싱

```python
system=[
    {"type": "text", "text": "지침"},
    {"type": "text", "text": "예제 1\n예제 2\n...\n예제 20",
     "cache_control": {"type": "ephemeral"}}
]
```

**효과**: 캐싱 없이는 비용 문제로 2-3개 예제만 가능 → 20+ 예제로 성능 대폭 향상

---

## 베스트 프랙티스

### ✅ 권장 사항

1. **정적 콘텐츠를 프롬프트 앞부분에 배치**
   - System 지침, Tool 정의, 배경 정보 등

2. **변동 빈도별로 breakpoint 분리**
   ```python
   # 자주 변경되는 순서: tools < system < RAG 문서 < 대화
   tools=[..., "cache_control"],           # 거의 불변
   system=[
       {"지침", "cache_control"},           # 가끔 변경
       {"RAG 문서", "cache_control"}        # 자주 변경
   ],
   messages=[..., {"대화", "cache_control"}]  # 매번 변경
   ```

3. **대화 끝과 편집 가능 콘텐츠 직전에 breakpoint 설정**
   - 20블록 이상 대화 시 캐시 히트율 극대화

4. **캐시 히트율 정기 분석**
   - `cache_read_input_tokens` / `total_input_tokens` 비율 모니터링

### ❌ 피해야 할 실수

1. **최소 토큰 미만 캐싱 시도**
   - 모델별 최소 토큰 수 확인 필수

2. **동적 콘텐츠 캐싱**
   - 사용자 입력 같은 가변 데이터는 제외

3. **tool_choice 또는 이미지 변경 후 캐시 기대**
   - 이런 변경은 캐시 무효화됨

4. **JSON 키 순서 불안정**
   - Swift, Go 같은 언어는 JSON 변환 시 키 순서 무작위 → 캐시 깨짐
   - 안정적인 키 순서 보장 필요

---

## Claude Code 특화 팁

### 자동 캐싱 활용

Claude Code는 내부적으로 prompt caching을 자동 활용:

- **CLAUDE.md**: 매 세션 로드 → 자동 캐싱
- **.claude/rules/**: 규칙 파일 → 자동 캐싱
- **대화 이력**: 턴마다 증분 캐싱

### 최적 설정 전략

1. **CLAUDE.md, rules, skills**: 영어 작성
   - 재사용 빈도 높음 → 캐싱 효과 극대화
   - 토큰 효율도 좋음 (영어가 한국어보다 50% 적은 토큰)

2. **긴 시스템 프롬프트는 분리**
   ```python
   # 변경 빈도별 분리
   system=[
       {"핵심 지침 (불변)", "cache_control"},
       {"프로젝트 컨텍스트 (가변)", "cache_control"}
   ]
   ```

3. **Tool 정의는 한 번에**
   - Tool 추가/수정 시 전체 tool cache 무효화
   - 안정된 tool set 확정 후 배포

---

## 문제 해결

### 예상과 다른 캐시 동작

**체크리스트**:
- ✓ 캐시된 섹션이 완전히 동일한가?
- ✓ `cache_control` 위치가 동일한가?
- ✓ 5분 이내에 재호출했는가?
- ✓ `tool_choice`나 이미지 사용이 일관한가?
- ✓ 최소 토큰 수 이상인가?
- ✓ JSON 키 순서가 안정적인가?

### 20블록 넘는 프롬프트에서 캐시 미스

**원인**: 자동 체크는 20블록까지만
**해결**: 편집 가능 영역 직전에 명시적 breakpoint 추가

```python
# 블록 5 수정 예정 → 블록 5 직전에 breakpoint
system=[{"블록 1-4", "cache_control"}, {"블록 5 (편집 가능)"}]
```

---

## 프라이버시 & 보안

1. **조직별 캐시 격리** (2026-02-05부터 workspace별 격리)
   - 다른 조직/workspace는 동일 프롬프트도 캐시 공유 안 됨

2. **암호화 해시 기반 키**
   - 캐시 키 = 프롬프트 내용의 cryptographic hash
   - 100% 동일한 프롬프트만 캐시 접근 가능

3. **출력 토큰에 영향 없음**
   - 캐싱 여부와 무관하게 동일한 응답 생성

---

## 핵심 포인트

- **비용 절감**: Cache Read는 일반 Input의 **10%** → 최대 90% 절감
- **속도 향상**: 긴 프롬프트에서 **최대 85% 지연 감소**
- **캐시 수명**: 기본 5분 (사용 시마다 갱신), 옵션으로 1시간 가능
- **Breakpoint**: 최대 4개, 자동 20블록 역순 체크
- **최소 토큰**: 모델별 1024~4096 토큰 (미만은 캐싱 안 됨)
- **무효화**: Tool/이미지/설정 변경 시 캐시 깨짐
- **Claude Code**: CLAUDE.md, rules 등은 자동 캐싱됨

---

## 메타

- 관련 문서: [[korean-vs-english-prompting.md]] (언어별 토큰 효율)
- 태그: #prompt-caching #optimization #cost-reduction #performance #api

## 출처

- [Prompt caching - Claude API Docs](https://platform.claude.com/docs/en/docs/build-with-claude/prompt-caching) — VERIFIED
- [Prompt caching with Claude - Anthropic](https://claude.com/blog/prompt-caching) — VERIFIED
- [How Prompt Caching Elevates Claude Code Agents - Walturn](https://www.walturn.com/insights/how-prompt-caching-elevates-claude-code-agents) — LIKELY
- [Supercharge development with Claude Code and prompt caching - AWS](https://aws.amazon.com/blogs/machine-learning/supercharge-your-development-with-claude-code-and-amazon-bedrock-prompt-caching/) — LIKELY

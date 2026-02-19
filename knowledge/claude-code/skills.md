# Claude Code Skills 완벽 가이드

> **요약**: Skills는 Claude Code의 기능을 확장하는 지시 파일 시스템이다. SKILL.md에 지시사항을 작성하면 Claude가 자동 또는 수동으로 로드하여 사용한다. 슬래시 커맨드, subagent 실행, 동적 context 주입 등 고급 기능을 지원한다.
> **출처**: https://code.claude.com/docs/en/skills
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-19

---

## 1. Skills란 무엇인가

Skills는 Claude Code의 기능을 확장하는 **지시사항 폴더**다. `SKILL.md` 파일에 지시사항을 작성하면 Claude가 자신의 toolkit에 추가한다. 사용자가 `/skill-name`으로 직접 호출하거나, Claude가 대화 맥락상 관련될 때 자동으로 로드한다.

### 핵심 특징

- **Progressive Disclosure**: 시작 시 이름/설명만 시스템 프롬프트에 로드 → 관련될 때 전체 내용 로드 → 필요 시 참조 파일 로드 (3단계 점진적 공개)
- **Agent Skills 표준**: [agentskills.io](https://agentskills.io) 오픈 표준을 따르며, 다른 AI 도구와도 호환 가능
- **기존 commands 통합**: `.claude/commands/review.md`와 `.claude/skills/review/SKILL.md`는 동일하게 `/review`를 생성. 기존 commands 파일도 계속 동작하지만, skills가 추가 기능(디렉토리 구조, frontmatter, 자동 호출)을 지원

### Skills vs Commands

| 구분 | Commands (레거시) | Skills (권장) |
|------|-------------------|---------------|
| 위치 | `.claude/commands/review.md` | `.claude/skills/review/SKILL.md` |
| 디렉토리 구조 | 단일 파일 | 디렉토리 (보조 파일 포함 가능) |
| Frontmatter | 지원 | 지원 |
| 자동 호출 | 미지원 | 지원 |
| 동일 이름 시 | Skills가 우선 | - |

---

## 2. SKILL.md 파일 구조

모든 skill은 `SKILL.md`를 진입점으로 하는 디렉토리다.

```
my-skill/
├── SKILL.md           # 메인 지시사항 (필수)
├── template.md        # Claude가 채울 템플릿
├── examples/
│   └── sample.md      # 예상 출력 형식
└── scripts/
    └── validate.sh    # Claude가 실행할 스크립트
```

### SKILL.md 기본 구조

```yaml
---
name: explain-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works.
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow
3. **Walk through the code**: Explain step-by-step
4. **Highlight a gotcha**: Common mistakes or misconceptions
```

두 부분으로 구성:
1. **YAML Frontmatter** (`---` 사이): Claude에게 언제 이 skill을 사용할지 알려주는 메타데이터
2. **Markdown Content**: Skill이 호출되었을 때 Claude가 따를 지시사항

---

## 3. Frontmatter 옵션 전체 목록

```yaml
---
name: my-skill
description: What this skill does
argument-hint: [issue-number]
disable-model-invocation: true
user-invocable: false
allowed-tools: Read, Grep, Glob
model: claude-sonnet-4-20250514
context: fork
agent: Explore
hooks:
  # hooks 설정 (Hooks in skills 참고)
---
```

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | 아니오 | Skill 표시 이름. 생략 시 디렉토리 이름 사용. 소문자, 숫자, 하이픈만 허용 (최대 64자) |
| `description` | 권장 | Skill의 용도와 사용 시점. Claude가 자동 호출 판단에 사용. 생략 시 markdown 첫 문단 사용 |
| `argument-hint` | 아니오 | 자동완성 시 표시되는 인자 힌트. 예: `[issue-number]`, `[filename] [format]` |
| `disable-model-invocation` | 아니오 | `true` 시 Claude 자동 호출 차단. `/name`으로만 수동 호출 가능. 기본값: `false` |
| `user-invocable` | 아니오 | `false` 시 `/` 메뉴에서 숨김. 배경 지식용. 기본값: `true` |
| `allowed-tools` | 아니오 | Skill 활성화 시 Claude가 권한 확인 없이 사용 가능한 도구 목록 |
| `model` | 아니오 | Skill 활성화 시 사용할 모델 지정 |
| `context` | 아니오 | `fork`으로 설정하면 격리된 subagent context에서 실행 |
| `agent` | 아니오 | `context: fork` 설정 시 사용할 subagent 타입. `Explore`, `Plan`, `general-purpose` 또는 커스텀 agent |
| `hooks` | 아니오 | Skill 생명주기에 연결되는 hooks 설정 |

---

## 4. Auto-invocation vs Manual invocation

### 동작 방식

| Frontmatter 설정 | 사용자 호출 | Claude 자동 호출 | Context 로드 방식 |
|-------------------|-------------|-------------------|-------------------|
| (기본값) | 가능 | 가능 | 설명은 항상 context에, 전체 내용은 호출 시 |
| `disable-model-invocation: true` | 가능 | 불가 | 설명이 context에 포함 안 됨, 사용자 호출 시만 로드 |
| `user-invocable: false` | 불가 | 가능 | 설명은 항상 context에, 전체 내용은 호출 시 |

### 사용 지침

- **`disable-model-invocation: true`**: 부작용이 있는 작업에 사용. 예: `/commit`, `/deploy`, `/send-slack-message`. Claude가 "코드가 준비된 것 같으니 배포하겠다"고 자동 판단하는 것을 방지
- **`user-invocable: false`**: 사용자 액션이 아닌 배경 지식에 사용. 예: `legacy-system-context`는 Claude가 알아야 할 정보이지만, 사용자가 직접 호출할 이유가 없음

### Claude의 Skill 접근 제어

Permission rules로 세밀하게 제어 가능:

```
# 특정 skill만 허용
Skill(commit)
Skill(review-pr *)

# 특정 skill 차단
Skill(deploy *)

# 모든 skill 차단
Skill
```

문법: `Skill(name)` (정확히 일치), `Skill(name *)` (prefix 일치 + 인자 허용)

---

## 5. $ARGUMENTS 치환 (String Substitution)

### 사용 가능한 변수

| 변수 | 설명 |
|------|------|
| `$ARGUMENTS` | 전달된 모든 인자. 내용에 없으면 자동으로 `ARGUMENTS: <값>` 형태로 끝에 추가됨 |
| `$ARGUMENTS[N]` | 0-based 인덱스로 특정 인자 접근. `$ARGUMENTS[0]`은 첫 번째 인자 |
| `$N` | `$ARGUMENTS[N]`의 단축형. `$0`은 첫 번째, `$1`은 두 번째 |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID. 로깅, 세션별 파일 생성에 유용 |

### 예제: 단일 인자

```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Implement the fix
3. Write tests
4. Create a commit
```

`/fix-issue 123` 실행 시 → "Fix GitHub issue 123 following our coding standards..."

### 예제: 복수 인자

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

`/migrate-component SearchBar React Vue` 실행 시:
- `$0` → `SearchBar`
- `$1` → `React`
- `$2` → `Vue`

### 예제: 세션 ID 활용

```yaml
---
name: session-logger
description: Log activity for this session
---

Log the following to logs/${CLAUDE_SESSION_ID}.log:

$ARGUMENTS
```

---

## 6. Supporting Files로 복잡한 Skill 만들기

`SKILL.md`는 500줄 이내로 유지하고, 상세한 참조 자료는 별도 파일로 분리한다.

### 디렉토리 구조

```
my-skill/
├── SKILL.md           # 개요와 네비게이션 (필수)
├── reference.md       # 상세 API 문서 (필요 시 로드)
├── examples.md        # 사용 예제 (필요 시 로드)
└── scripts/
    └── helper.py      # 유틸리티 스크립트 (실행용, 로드 안 함)
```

### SKILL.md에서 참조하기

```markdown
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

이렇게 참조하면 Claude가 각 파일의 내용과 로드 시점을 인지한다. 큰 참조 문서, API 명세, 예제 컬렉션은 skill이 실행될 때마다 context에 로드되지 않으며, 필요할 때만 접근한다.

### 실전 패턴: 시각적 출력 생성

Python 스크립트를 번들하여 HTML 시각화를 생성하는 패턴:

```
codebase-visualizer/
├── SKILL.md
└── scripts/
    └── visualize.py    # Python으로 인터랙티브 HTML 생성
```

```yaml
---
name: codebase-visualizer
description: Generate an interactive collapsible tree visualization of your codebase.
allowed-tools: Bash(python *)
---

# Codebase Visualizer

Run the visualization script from your project root:

```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```
```

이 패턴은 의존성 그래프, 테스트 커버리지 리포트, API 문서, DB 스키마 시각화 등에 활용 가능하다.

---

## 7. context: fork로 Subagent에서 실행하기

`context: fork`를 설정하면 skill이 격리된 subagent에서 실행된다. SKILL.md의 내용이 subagent의 프롬프트가 된다. 대화 기록에 접근할 수 없다.

### 기본 구조

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

### 실행 흐름

1. 새로운 격리된 context 생성
2. Subagent가 SKILL.md 내용을 프롬프트로 수신
3. `agent` 필드가 실행 환경(모델, 도구, 권한) 결정
4. 결과가 요약되어 메인 대화에 반환

### agent 필드 옵션

- `Explore`: 읽기 전용 도구, 코드베이스 탐색에 최적화
- `Plan`: 읽기 전용, 계획 수립용
- `general-purpose`: 전체 도구 접근 (기본값)
- 커스텀 agent: `.claude/agents/`에 정의된 agent

### Skills vs Subagents 비교

| 접근 방식 | 시스템 프롬프트 | 태스크 | 추가 로드 |
|-----------|----------------|--------|-----------|
| Skill + `context: fork` | agent 타입에서 제공 | SKILL.md 내용 | CLAUDE.md |
| Subagent + `skills` 필드 | Subagent의 markdown 본문 | Claude의 위임 메시지 | 사전 로드된 skills + CLAUDE.md |

### 주의사항

`context: fork`는 명확한 지시사항이 있는 skill에서만 의미가 있다. "이 API 규칙을 사용하라" 같은 가이드라인만 있고 실행할 태스크가 없으면, subagent가 가이드라인만 받고 유의미한 출력 없이 반환된다.

---

## 8. Dynamic Context Injection (`` !`command` ``)

`` !`command` `` 구문은 skill 내용이 Claude에게 전달되기 **전에** 셸 명령을 실행한다. 명령 출력이 placeholder를 대체하므로, Claude는 명령이 아닌 실제 데이터를 받는다.

### 구문

```
!`shell command here`
```

### 예제: PR 요약 skill

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

### 실행 순서

1. 각 `` !`command` ``가 즉시 실행 (Claude가 보기 전에)
2. 출력이 skill 내용의 placeholder를 대체
3. Claude는 실제 PR 데이터가 삽입된 완성된 프롬프트를 수신

이것은 **전처리**다. Claude가 실행하는 것이 아니다. Claude는 최종 결과만 본다.

### 활용 사례

- Git 상태 정보 주입
- API 응답 데이터 삽입
- 시스템 환경 정보 수집
- 파일 목록이나 구조 정보 제공

---

## 9. Skill 파일 위치와 검색 경로

### 저장 위치별 적용 범위

| 위치 | 경로 | 적용 범위 |
|------|------|-----------|
| Enterprise | Managed settings 참고 | 조직 내 모든 사용자 |
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | 모든 프로젝트 |
| Project | `.claude/skills/<skill-name>/SKILL.md` | 해당 프로젝트만 |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | Plugin이 활성화된 곳 |

### 우선순위

동일 이름일 경우: **Enterprise > Personal > Project**

Plugin skills는 `plugin-name:skill-name` 네임스페이스를 사용하므로 충돌하지 않는다.

### 자동 탐색 (Nested Directories)

하위 디렉토리의 파일을 편집할 때, Claude Code는 중첩된 `.claude/skills/` 디렉토리에서도 자동으로 skill을 탐색한다. 예를 들어 `packages/frontend/`의 파일을 편집 중이면, `packages/frontend/.claude/skills/`도 검색한다. 모노레포에서 패키지별 skill을 지원한다.

### 추가 디렉토리

`--add-dir`로 추가된 디렉토리의 `.claude/skills/`도 자동 로드되며, 실시간 변경 감지를 지원한다. 세션 중에 편집해도 재시작 없이 반영된다.

### Context 예산

Skill 설명은 context window의 2%까지 로드된다 (fallback: 16,000자). 많은 skill이 있으면 일부가 제외될 수 있다. `/context`로 경고를 확인할 수 있다.

환경 변수로 제한 조정: `SLASH_COMMAND_TOOL_CHAR_BUDGET`

---

## 10. 실용적인 예제

### 예제 1: 코드 리뷰 Skill

```yaml
---
name: review
description: Review code changes for bugs, style issues, and improvements
context: fork
agent: Explore
---

Review the current changes in the repository:

1. Run !`git diff --stat` to see changed files
2. Analyze each changed file for:
   - Potential bugs or logic errors
   - Style inconsistencies
   - Performance concerns
   - Missing error handling
3. Provide a structured review with severity levels (critical/warning/suggestion)
```

### 예제 2: 커밋 Skill (수동 전용)

```yaml
---
name: commit
description: Create a well-formatted commit with conventional commit messages
disable-model-invocation: true
argument-hint: [type: feat|fix|refactor|docs]
---

Create a git commit:

1. Run `git status` and `git diff --staged` to understand changes
2. If no files are staged, ask which files to stage
3. Write a commit message following conventional commits:
   - Type: $0 (or infer from changes if not provided)
   - Scope: infer from changed files
   - Description: concise summary of what changed and why
4. Create the commit
5. Show the commit summary
```

`/commit feat` → conventional commit 타입이 `feat`로 설정된 커밋 생성

### 예제 3: 안전한 읽기 전용 탐색 Skill

```yaml
---
name: safe-explore
description: Explore and understand code without making any changes
allowed-tools: Read, Grep, Glob
user-invocable: false
---

When exploring code for understanding:
- Use Glob to find relevant files
- Use Grep to search for patterns and references
- Use Read to examine file contents
- Never suggest modifications unless explicitly asked
- Provide clear summaries with file path references
```

이 skill은 `user-invocable: false`이므로 `/` 메뉴에 나타나지 않고, Claude가 코드 탐색이 필요하다고 판단할 때 자동으로 활성화된다.

---

## 트러블슈팅

| 증상 | 해결 방법 |
|------|-----------|
| Skill이 트리거되지 않음 | description에 사용자가 자연스럽게 말할 키워드 포함. `What skills are available?`로 확인 |
| Skill이 너무 자주 트리거됨 | description을 더 구체적으로 수정. `disable-model-invocation: true` 설정 |
| Claude가 일부 skill을 못 봄 | Context 예산 초과. `/context`로 확인. `SLASH_COMMAND_TOOL_CHAR_BUDGET` 환경 변수 조정 |

---

## 팁

- **Extended Thinking 활성화**: Skill 내용 어딘가에 "ultrathink"를 포함하면 extended thinking 모드가 활성화됨
- **SKILL.md는 500줄 이내**: 상세 참조 자료는 별도 파일로 분리
- **description이 핵심**: Claude의 자동 호출 판단은 description에 의존

---

## 핵심 포인트

- Skills는 SKILL.md 파일 하나로 시작하는 간단한 확장 시스템이다
- Progressive Disclosure (이름 → 설명 → 전체 내용 → 참조 파일)로 context를 효율적으로 관리한다
- `disable-model-invocation`과 `user-invocable`로 호출 권한을 세밀하게 제어한다
- `context: fork`로 격리된 subagent에서 실행하여 메인 대화를 오염시키지 않는다
- `` !`command` ``로 실행 전에 동적 데이터를 주입할 수 있다
- Supporting files와 번들된 스크립트로 복잡한 워크플로우를 구축할 수 있다

## 메타

- 관련 문서: [[context-management.md]], [[permissions.md]], [[claude-md-guide.md]]
- 태그: #skills #slash-commands #subagent #automation #progressive-disclosure

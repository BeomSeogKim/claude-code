# CLAUDE.md 완벽 가이드

> **요약**: CLAUDE.md는 Claude Code의 "영구 기억"으로, 매 세션마다 자동 로드되는 프로젝트 지침서
> **출처**: Anthropic 공식 문서, 커뮤니티 베스트 프랙티스
> **신뢰도**: [VERIFIED]
> **최종 갱신**: 2026-02-16

## 핵심 개념

**CLAUDE.md는 Claude가 매 세션 시작 시 자동으로 읽는 특수 파일**입니다.

- 코드에서 추론할 수 없는 정보를 제공
- Bash 명령어, 코드 스타일, 워크플로우 규칙 포함
- 프로젝트의 "헌법"이자 Claude의 영구 기억

---

## `/init` 명령어로 자동 생성

### 사용법

```bash
> /init
```

### 동작 원리

1. **코드베이스 분석**: package.json, README, 설정 파일 등 탐색
2. **패턴 감지**: 빌드 시스템, 테스트 프레임워크, 코드 스타일 파악
3. **CLAUDE.md 생성**: 감지된 정보를 기반으로 초안 작성

### 생성되는 내용

- 빌드/테스트 명령어 (`npm run test`, `cargo build` 등)
- 코드 스타일 규칙 (들여쓰기, import 스타일 등)
- 주요 디렉토리 구조
- 사용 기술 스택
- 아키텍처 개요

**중요**: `/init`은 시작점일 뿐. 프로젝트 특성에 맞게 커스터마이징 필수!

---

## 파일 위치 및 우선순위

### 위치별 용도

| 위치 | 경로 | 용도 | 공유 범위 |
|------|------|------|----------|
| **조직 정책** | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | 회사 전체 표준 | 조직 전체 |
| **프로젝트 공유** | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀 공유 규칙 | Git으로 팀 공유 |
| **프로젝트 모듈** | `./.claude/rules/*.md` | 주제별 세분화 규칙 | Git으로 팀 공유 |
| **개인 전역** | `~/.claude/CLAUDE.md` | 모든 프로젝트 개인 선호 | 본인만 (모든 프로젝트) |
| **개인 프로젝트** | `./CLAUDE.local.md` | 이 프로젝트만 개인 선호 | 본인만 (현재 프로젝트) |
| **자동 메모리** | `~/.claude/projects/<project>/memory/` | Claude 자동 학습 노트 | 본인만 (프로젝트별) |

### 우선순위 규칙

**구체적인 것이 우선**: 조직 정책 < 개인 전역 < 프로젝트 공유 < 개인 프로젝트

**상하 계층**:
- 상위 디렉토리 CLAUDE.md: 프로젝트 루트에서 자동 로드
- 하위 디렉토리 CLAUDE.md: 해당 디렉토리 작업 시 on-demand 로드

---

## 파일 구조 예시

### 기본 구조

```markdown
# 프로젝트명

## 빌드 & 테스트

- 테스트: `npm test`
- 빌드: `npm run build`
- 린트: `npm run lint`

## 코드 스타일

- ES modules 사용 (import/export), CommonJS 금지 (require)
- 2-space 들여쓰기
- import는 destructure 선호: `import { foo } from 'bar'`

## 워크플로우

- 코드 변경 후 반드시 typecheck 실행
- 단일 테스트 실행 선호 (전체 suite는 느림)
- 커밋 전 lint 통과 필수

## 아키텍처

- `src/api/`: REST API 엔드포인트
- `src/components/`: React 컴포넌트
- `src/lib/`: 공통 유틸리티
```

### 모듈화 구조 (큰 프로젝트)

```
your-project/
├── .claude/
│   ├── CLAUDE.md              # 핵심 규칙
│   └── rules/
│       ├── code-style.md      # 코드 스타일
│       ├── testing.md         # 테스트 규칙
│       ├── security.md        # 보안 요구사항
│       └── frontend/
│           ├── react.md       # React 규칙
│           └── styles.md      # CSS 규칙
```

`.claude/rules/*.md` 파일들은 자동으로 로드되며, CLAUDE.md와 동일한 우선순위를 가집니다.

---

## `@` Import 문법

다른 파일을 import하여 재사용 가능:

```markdown
# CLAUDE.md

프로젝트 개요는 @README.md 참조.
npm 명령어는 @package.json 참조.

# 추가 지침
- Git 워크플로우: @docs/git-instructions.md
- 개인 설정: @~/.claude/my-project-instructions.md
```

### Import 규칙

- **상대 경로**: import를 포함한 파일 기준 (working directory 아님)
- **절대 경로**: `~/.claude/` 같은 home directory 경로 사용 가능
- **최대 depth**: 5단계까지 재귀 import 가능
- **코드 블록 무시**: markdown 코드 블록 내 `@` 는 무시됨

**보안**: 외부 import는 첫 사용 시 승인 필요 (프로젝트당 1회)

---

## 경로별 조건부 규칙 (Path-specific rules)

`.claude/rules/` 파일에 frontmatter로 적용 범위 제한:

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "lib/api/**/*.ts"
---

# API 개발 규칙

- 모든 API 엔드포인트는 input validation 필수
- 표준 error response 포맷 사용
- OpenAPI documentation 주석 포함
```

### Glob 패턴

| 패턴 | 매칭 |
|------|------|
| `**/*.ts` | 모든 디렉토리의 TypeScript 파일 |
| `src/**/*` | src/ 하위 모든 파일 |
| `*.md` | 프로젝트 루트의 Markdown |
| `src/**/*.{ts,tsx}` | src/ 하위 .ts 또는 .tsx 파일 |

**paths 없는 규칙**: 모든 파일에 무조건 적용

---

## Auto Memory (자동 메모리)

### 개념

Claude가 **스스로 작성하는 메모**:
- CLAUDE.md: 사용자가 작성한 지침
- Auto memory: Claude가 학습한 패턴/인사이트

### 위치

```
~/.claude/projects/<project>/memory/
├── MEMORY.md           # 핵심 인덱스 (200줄까지 자동 로드)
├── debugging.md        # 디버깅 패턴 상세
├── api-conventions.md  # API 설계 결정사항
└── ...
```

### 동작

- **MEMORY.md 첫 200줄**: 매 세션 자동 로드
- **200줄 초과 내용**: 로드 안 됨 (Claude가 on-demand로 읽음)
- **Topic 파일**: Claude가 필요시 직접 읽음

### 활성화/비활성화

```bash
# 강제 비활성화
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=1

# 강제 활성화
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=0
```

### 수동 관리

- `/memory` 명령어로 파일 편집
- "remember that we use pnpm" → Claude가 자동 저장
- 직접 파일 수정 가능

---

## 작성 베스트 프랙티스

### ✅ 포함해야 할 것

| 항목 | 예시 |
|------|------|
| **추론 불가능한 명령어** | `npm run test:integration` (docs에 없음) |
| **프로젝트 특유 스타일** | "API는 kebab-case, JSON은 camelCase" |
| **테스트 지침** | "단일 테스트 실행 선호, Jest 사용" |
| **Git 규칙** | "브랜치명: feature/*, PR 전 squash" |
| **아키텍처 결정** | "인증은 JWT, 세션 저장은 Redis" |
| **환경 특이사항** | ".env.local 필요, REDIS_URL 설정 필수" |
| **Gotchas** | "빌드 전 migrations 실행 필요" |

### ❌ 제외해야 할 것

| 항목 | 이유 |
|------|------|
| **코드로 알 수 있는 것** | 파일 읽으면 알 수 있음 |
| **언어 기본 규칙** | Claude가 이미 알고 있음 |
| **상세 API 문서** | 링크만 제공 (문서 자체는 X) |
| **자주 변경되는 정보** | Skill이나 별도 파일로 분리 |
| **장황한 설명** | 간결하게, 핵심만 |
| **당연한 것** | "clean code 작성" 같은 자명한 것 |

### 간결함 유지 (Critical!)

**150-200 지침 한계**: 연구에 따르면 LLM은 150-200개 지침까지만 일관되게 따를 수 있음

**각 줄마다 질문하기**:
> "이 줄을 제거하면 Claude가 실수할까?"
> - YES → 유지
> - NO → 삭제

**1페이지 규칙**: CLAUDE.md는 1페이지 (약 50-100줄) 이내 권장

**비대해지면**:
- Claude가 지침을 무시하기 시작
- 중요한 규칙이 "묻힘"
- 실제 사용자 지시가 무시됨

---

## 강조 기법

Claude가 특정 지침을 잘 안 따르면 강조:

```markdown
# 코드 스타일

- IMPORTANT: 절대로 CommonJS (require) 사용 금지
- YOU MUST: 모든 API에 input validation 필수
- CRITICAL: 프로덕션 배포 전 반드시 테스트 통과
```

**주의**: 너무 많이 쓰면 효과 감소. 정말 중요한 것만.

---

## 유지보수 전략

### 주기적 리뷰

- **월 1회**: CLAUDE.md 검토 및 불필요한 내용 제거
- **프로젝트 변경 시**: 새 도구/패턴 추가, 오래된 것 삭제
- **Claude 실수 패턴 분석**: 반복 실수 → CLAUDE.md에 규칙 추가

### Git 관리

```bash
# 팀 공유 규칙
git add .claude/CLAUDE.md
git commit -m "Add API naming conventions"

# 개인 설정은 .gitignore
echo "CLAUDE.local.md" >> .gitignore
```

### 진화적 접근

- **처음부터 완벽 X**: 작게 시작 → 문제 발생 시 추가
- **팀 학습 반영**: 좋은 패턴 발견 시 CLAUDE.md 업데이트
- **도구 변경 추적**: 새 프레임워크/라이브러리 도입 시 문서화

---

## 실전 예시

### 최소 CLAUDE.md (소형 프로젝트)

```markdown
# MyApp

## Commands
- Test: `npm test`
- Build: `npm run build`
- Dev: `npm run dev`

## Code Style
- 2-space indentation
- ES modules only (no require)
- Prettier formatting (auto on save)

## Workflow
- Always run tests after changes
- Lint must pass before commit
```

### 모듈화 CLAUDE.md (대형 프로젝트)

**`.claude/CLAUDE.md`** (핵심만):
```markdown
# Enterprise App

## Quick Start
@README.md

## Commands
@package.json

## Rules
Detailed rules are in `.claude/rules/`:
- Code style → `rules/code-style.md`
- Testing → `rules/testing.md`
- Security → `rules/security.md`
- API design → `rules/api-design.md`
```

**`.claude/rules/api-design.md`**:
```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Design Rules

## Naming
- Endpoints: kebab-case (`/api/user-profile`)
- JSON keys: camelCase (`userId`, `createdAt`)

## Validation
- All inputs validated with Zod
- Return 400 with detailed error messages

## Documentation
- Include OpenAPI/Swagger comments
- Example requests/responses in docstrings
```

---

## 고급 패턴

### 1. Monorepo 구조

```
monorepo/
├── CLAUDE.md                    # 전체 monorepo 규칙
├── packages/
│   ├── frontend/
│   │   └── CLAUDE.md            # Frontend 특화 규칙
│   └── backend/
│       └── CLAUDE.md            # Backend 특화 규칙
```

하위 디렉토리 작업 시 상위 + 하위 CLAUDE.md 모두 로드

### 2. Symlink로 규칙 공유

```bash
# 회사 공통 규칙 symlink
ln -s ~/company-standards/security.md .claude/rules/security.md

# 개인 공통 규칙
ln -s ~/.claude/shared-rules .claude/rules/shared
```

### 3. 개인 설정 분리

**`.claude/CLAUDE.md`** (팀 공유):
```markdown
# Team Rules
...
@~/.claude/my-personal-preferences.md
```

**`~/.claude/my-personal-preferences.md`** (개인):
```markdown
# 내 선호 스타일
- 변수명은 한글 주석 포함 선호
- 디버깅 시 console.log보다 debugger 선호
```

---

## 트러블슈팅

### Claude가 CLAUDE.md 무시하는 것 같아요

**원인**:
1. 파일이 너무 김 (200줄 이상)
2. 모호한 표현
3. 당연한 내용으로 가득 참

**해결**:
- 50-100줄로 줄이기
- 구체적으로 작성 ("좋은 코드" X → "2-space indentation" O)
- 불필요한 것 과감히 삭제

### `/init`이 엉뚱한 내용을 생성했어요

**해결**:
- `/init`은 시작점일 뿐, 수동 편집 필수
- 잘못된 부분 삭제, 누락된 부분 추가
- 프로젝트 이해하며 점진적 개선

### Auto memory가 작동 안 해요

**확인**:
```bash
# 환경변수 확인
echo $CLAUDE_CODE_DISABLE_AUTO_MEMORY

# 강제 활성화
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=0
```

### Import한 파일이 로드 안 돼요

**원인**: 첫 사용 시 승인 필요

**해결**:
- 승인 다이얼로그에서 "Approve" 클릭
- 거부했다면 `.claude/` 설정 초기화 후 재시도

---

## 체크리스트

완료 체크:
- [ ] `/init`으로 CLAUDE.md 자동 생성
- [ ] 프로젝트 특성에 맞게 커스터마이징
- [ ] 50-100줄 이내로 간결하게 유지
- [ ] `@` import 문법으로 모듈화
- [ ] 팀 공유 규칙은 Git에 커밋
- [ ] 개인 설정은 `CLAUDE.local.md` 또는 `~/.claude/CLAUDE.md`
- [ ] 월 1회 정기 리뷰 및 정리

---

## 핵심 포인트

- **CLAUDE.md = Claude의 영구 기억**: 매 세션 자동 로드
- **/init으로 시작**: 자동 생성 후 커스터마이징
- **간결함이 생명**: 50-100줄 권장, 200줄 초과 시 무시됨
- **위치별 우선순위**: 구체적 > 일반적
- **모듈화**: 큰 프로젝트는 `.claude/rules/` 활용
- **@import**: 파일 재사용으로 중복 제거
- **Auto memory**: Claude가 스스로 학습한 내용 (MEMORY.md)
- **진화적 접근**: 완벽부터 시작 X, 점진적 개선

---

## 메타

- 관련 문서: [[learning-checklist.md]] (체크리스트 항목 1-1)
- 태그: #claude-md #project-setup #best-practices #memory

## 출처

- [Manage Claude's memory - Claude Code Docs](https://code.claude.com/docs/en/memory) — VERIFIED
- [Best Practices for Claude Code - Claude Code Docs](https://code.claude.com/docs/en/best-practices) — VERIFIED
- [Using CLAUDE.MD files - Claude Blog](https://claude.com/blog/using-claude-md-files) — VERIFIED
- [Writing a good CLAUDE.md - HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — LIKELY
- [How to Write a Good CLAUDE.md File - Builder.io](https://www.builder.io/blog/claude-md-guide) — LIKELY

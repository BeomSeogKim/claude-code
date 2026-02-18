# Claude Code 고급화 학습 체크리스트

> **용도**: Claude Code 마스터를 위한 학습 로드맵. 각 항목 학습 완료 시 `[ ]` → `[x]` 로 체크
> **최종 갱신**: 2026-02-16

---

## 📊 학습 진행률

- **전체**: 4 / 50 항목
- **기초**: 4 / 6
- **중급**: 0 / 12
- **고급**: 0 / 17
- **전문가**: 0 / 15

---

## 🏗️ 1. 프로젝트 구조 & 설정 (기초)

### CLAUDE.md 마스터하기
- [x] `/init`으로 자동 생성하고 커스터마이징하기
- [x] 간결함 유지 원칙 이해 (1페이지 이내)
- [x] `@` 문법으로 다른 파일 import 활용
- [x] 위치별 우선순위 이해 (home → project → child)

### Permission 설정
- [ ] `/permissions`로 allowlist 설정하기
- [ ] Sandbox 모드 활용법 학습

### Context 관리
- [ ] Context window 중요성 이해
- [ ] `/clear` 사용 타이밍 터득
- [ ] `/compact` 커스터마이징
- [ ] Status line으로 토큰 추적 설정

---

## 🛠️ 2. 핵심 확장 기능 (중급)

### Skills (자동 실행 기능)
- [ ] 첫 번째 `SKILL.md` 작성 및 테스트
- [ ] Frontmatter 옵션 이해 (name, description, disable-model-invocation 등)
- [ ] Auto-invocation vs Manual invocation 차이 구분
- [ ] `$ARGUMENTS`, `$0`, `$1` substitution 활용
- [ ] Supporting files 구조로 복잡한 skill 만들기
- [ ] `context: fork`로 subagent에서 실행하기
- [ ] Dynamic context injection (`` !`command` ``) 사용

### Hooks (이벤트 기반 자동화)
- [ ] 7가지 이벤트 타입 이해
- [ ] Command mode vs Prompt mode 차이
- [ ] `.claude/settings.json`에 첫 hook 설정
- [ ] Auto-formatting hook 구현
- [ ] Validation hook 구현

### Subagents (전문화된 AI 위임)
- [ ] 독립 context window 개념 이해
- [ ] `.claude/agents/*.md` 첫 subagent 작성
- [ ] Preloaded skills 활용하기
- [ ] 병렬 실행 패턴 테스트
- [ ] Context poisoning 방지 효과 체감

### MCP (Model Context Protocol)
- [ ] `claude mcp add`로 첫 MCP 서버 연결
- [ ] MCP tool 제약사항 이해
- [ ] Context 소비 특성 파악

### Plugins
- [ ] `/plugin`으로 마켓플레이스 탐색
- [ ] 첫 plugin 설치 및 사용
- [ ] Code intelligence plugin 활용

---

## 💡 3. 워크플로우 패턴 (고급)

### Plan Mode 활용
- [ ] Explore → Plan → Implement → Commit 4단계 워크플로우 실습
- [ ] `Ctrl+G`로 plan 편집 경험
- [ ] Plan Mode 사용/미사용 판단 기준 내재화

### Multi-session 전략
- [ ] Desktop app으로 parallel sessions 테스트
- [ ] Writer/Reviewer 패턴 실습
- [ ] Agent teams 구성 및 활용

### Headless Mode
- [ ] `claude -p "prompt"` 기본 사용
- [ ] `--output-format json` 출력 파싱
- [ ] `--output-format stream-json` 실시간 처리
- [ ] CI/CD 통합 시나리오 구현
- [ ] Fan-out 패턴으로 대량 파일 처리

### Rewind & Checkpointing
- [ ] `Esc + Esc` 또는 `/rewind` 사용
- [ ] Conversation vs Code 복원 차이 이해
- [ ] Summarize from checkpoint 활용
- [ ] 제약사항 (Claude 변경만 추적) 인지

---

## 🎯 4. 프롬프팅 기법 (중급)

### Rich Context 제공
- [ ] `@` 파일 참조 활용
- [ ] 이미지/스크린샷 paste 활용
- [ ] Pipe 입력 (`cat file | claude`) 사용

### Verification 제공
- [ ] 테스트 케이스 제시 패턴
- [ ] 스크린샷 비교 패턴
- [ ] "Verify your work" 지시 습관화

### 구체적 지시
- [ ] 파일/패턴 명시적 참조 습관
- [ ] 제약사항 명확화 기법
- [ ] Edge case 언급 패턴

### Interview Pattern
- [ ] Claude에게 질문하게 하기
- [ ] `AskUserQuestion` tool 활용
- [ ] Spec 작성 후 새 세션 구현 패턴

---

## 🔧 5. 환경 & 도구 통합 (중급)

### CLI Tools 활용
- [ ] `gh` (GitHub CLI) 설치 및 활용
- [ ] 프로젝트별 CLI tool 연동
- [ ] Claude에게 새 CLI 학습시키기

### Git 워크플로우
- [ ] Safe git operations 패턴 확립
- [ ] Branch 전략 자동화
- [ ] Commit message 규칙 자동 적용

### Testing Integration
- [ ] Test runner 자동 실행 설정
- [ ] TDD 패턴 with Claude

---

## 📊 6. 성능 최적화 (고급)

### Token 효율화
- [ ] Subagent로 investigation 분리 효과 측정
- [ ] Skill 모듈화로 CLAUDE.md 간소화
- [ ] 불필요한 파일 읽기 방지 패턴

### Caching 이해
- [ ] Prompt caching 자동 동작 확인
- [ ] CLAUDE.md, rules 영어 작성 효과 측정
- [ ] Tool 정의 안정화 전략

### Model 선택
- [ ] Opus vs Sonnet vs Haiku 비용/성능 비교
- [ ] Skill/Agent별 적절한 model 지정

---

## 🚫 7. 안티패턴 & 트러블슈팅 (기초)

### 흔한 실수 인지
- [ ] Kitchen sink session 경험 및 해결
- [ ] 반복 수정의 늪 탈출 (2회 규칙)
- [ ] 비대한 CLAUDE.md 리팩토링
- [ ] Verification 부재로 인한 실패 경험

### Course Correction
- [ ] `Esc` 중단 타이밍 익히기
- [ ] 명확한 피드백 제공 패턴

---

## 🎓 8. 고급 패턴 (전문가)

### Visual Output Generation
- [ ] Skills에 script 번들링
- [ ] Interactive HTML 생성 패턴
- [ ] Visualization 자동화

### Agent Teams
- [ ] Task 분배 전략
- [ ] Team coordination 패턴
- [ ] 복잡한 프로젝트에 적용

### Autonomous Workflows
- [ ] Sandbox 환경 구축
- [ ] 안전한 자동화 파이프라인
- [ ] Monitoring & logging 설정

---

## 📝 학습 노트

### 완료한 항목별 메모

**[2026-02-16] ✓ CLAUDE.md 마스터하기 (4개 항목 완료)**
- 문서: `claude-md-guide.md` 작성 완료
- 핵심 인사이트:
  - `/init`은 시작점, 반드시 커스터마이징 필요
  - 50-100줄 유지 (200줄 초과 시 Claude가 무시)
  - 150-200 지침 한계 존재 (연구 결과)
  - 각 줄마다 "제거하면 실수할까?" 질문하기
  - 모듈화: 큰 프로젝트는 `.claude/rules/` 활용
  - Auto memory: Claude가 스스로 학습 (MEMORY.md)
- 다음: Permission 설정 학습

---

## 🎯 다음 학습 목표

**현재 집중**: (여기에 지금 공부 중인 항목 기록)

**이번 주**:
- [ ]

**이번 달**:
- [ ]

---

## 📚 관련 문서

- [[prompt-caching.md]] - Prompt Caching 상세
- [[korean-vs-english-prompting.md]] - 언어별 효율성

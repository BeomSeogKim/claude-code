# 설정 분석 규칙

## 분석 대상

Claude Code 프로젝트의 다음 설정 파일들을 분석한다:

- `CLAUDE.md` — 페르소나, 핵심 규칙, 프로젝트 컨텍스트
- `.claude/rules/*.md` — 행동 규칙
- `.claude/skills/*/SKILL.md` — 스킬 정의
- `.claude/agents/*.md` — 에이전트 정의
- `.claude/settings.json` — 권한 설정
- 기타 프로젝트 설정 파일 (package.json, tsconfig 등)

## 분석 포맷

```markdown
# [프로젝트명] 설정 분석

> **출처**: URL 또는 경로
> **분석일**: YYYY-MM-DD

## 개요

(프로젝트의 목적과 설정 전체 요약)

## 파일별 분석

### [파일명]
- **역할**: 이 파일이 하는 일
- **핵심 설정**: 주요 설정값과 의미
- **특이사항**: 일반적이지 않은 설정

## 교차 관찰

(파일 간 관계, 패턴, 일관성 등)

## 교훈

- 이 프로젝트에서 배울 수 있는 점
- 우리 프로젝트에 적용할 수 있는 점
```

## 파일명 규칙

- `settings/` 하위에 저장
- 파일명: `{프로젝트명}-analysis.md` (kebab-case)
- 예: `cursor-rules-analysis.md`, `typescript-starter-analysis.md`

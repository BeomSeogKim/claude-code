# 파일 규칙

## 파일명

- **kebab-case** 사용: `my-document.md` (O), `MyDocument.md` (X)
- 영어로 작성: `claude-code-overview.md` (O), `클로드-코드-개요.md` (X)
- 의미가 명확하게: `hook-system.md` (O), `doc1.md` (X)

## 마크다운 포맷

- 제목: `#` 한 개만 (문서당 하나)
- 소제목: `##`, `###` 순서대로
- 코드 블록: 언어 명시 (```javascript, ```bash 등)
- 링크: 상대 경로 사용 (`../claude-code/hooks.md`)

## 언어 규칙

- **문서 본문**: 한국어
- **기술 용어**: 영어 유지 (hook, skill, agent, settings, fetch 등)
- **파일명/디렉토리명**: 영어
- **코드/설정**: 영어
- **커밋 메시지**: 한국어 또는 영어 (일관성 유지)

## 디렉토리별 용도

| 디렉토리 | 용도 | 예시 |
|-----------|------|------|
| `knowledge/claude-code/` | Claude Code 기능, 설정, 내부 동작 | `hooks.md`, `skills.md` |
| `knowledge/prompting/` | 프롬프팅 기법, 시스템 프롬프트 | `system-prompt-tips.md` |
| `knowledge/best-practices/` | 검증된 모범 사례 | `project-setup.md` |
| `settings/` | 외부 프로젝트 설정 분석 | `awesome-project-analysis.md` |

# /analyze-setting

## 설명
Claude Code 프로젝트의 설정 파일을 가져와 구조화된 분석을 수행하고 `settings/`에 저장한다.

## 사용법
```
/analyze-setting <URL 또는 로컬 경로>
```

## 인자
- `$ARGUMENTS`: 분석할 프로젝트의 GitHub URL 또는 로컬 파일 경로

## 절차

1. **대상 확인**: 인자로 받은 URL 또는 경로 확인
2. **파일 수집**:
   - URL인 경우: WebFetch로 CLAUDE.md, .claude/ 하위 파일들을 fetch
   - 로컬 경로인 경우: Read로 파일 읽기
3. **분석 수행**: `.claude/rules/settings-analysis.md`의 분석 포맷에 따라 분석
4. **저장**: `settings/{프로젝트명}-analysis.md`로 저장
5. **인덱스 갱신**: `settings/README.md` 업데이트

## 출력 예시
```
✅ 분석 완료: settings/awesome-project-analysis.md
- CLAUDE.md: 페르소나 기반 설정, 15개 규칙
- rules: 3개 파일 (코딩 표준, 테스트, 배포)
- skills: 2개 (/deploy, /test)
```

## 주의사항
- 비공개 저장소는 접근 불가할 수 있음
- raw 파일 URL을 사용하거나 gh CLI 활용

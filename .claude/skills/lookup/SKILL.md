# /lookup

## 설명
기존 `knowledge/` 디렉토리에서 관련 문서를 검색하여 요약을 반환한다. 웹 검색은 하지 않는다.

## 사용법
```
/lookup <질문>
```

## 인자
- `$ARGUMENTS`: 찾고자 하는 내용 (예: "hooks 사용법", "skill 만드는 법")

## 절차

1. **키워드 추출**: 질문에서 핵심 키워드 추출
2. **검색**: `knowledge/` 디렉토리에서 Grep으로 관련 문서 탐색
3. **관련 문서 읽기**: 검색된 문서들을 Read로 읽기
4. **요약 반환**: 관련 내용을 간결하게 요약하여 응답

## 출력 예시
```
📖 검색 결과: "hooks" 관련 2개 문서 발견

1. knowledge/claude-code/hooks.md
   → hook은 이벤트 기반 자동 실행 스크립트. PreToolUse, PostToolUse 등의 이벤트에 반응...

2. knowledge/best-practices/project-setup.md
   → 프로젝트 초기 설정 시 hooks를 활용하면...
```

## 주의사항
- 웹 검색 없이 로컬 knowledge/만 탐색
- 관련 문서가 없으면 `/update-knowledge` 사용을 안내

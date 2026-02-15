# /update-knowledge

## 설명
특정 주제에 대해 기존 지식을 확인하고, 최신 정보를 리서치하여 문서를 생성하거나 갱신한다.

## 사용법
```
/update-knowledge <주제>
```

## 인자
- `$ARGUMENTS`: 조사하고 저장할 주제 (예: "Claude Code hooks", "system prompt 구조")

## 절차

1. **기존 검색**: `knowledge/` 에서 관련 문서 검색 (Grep, Glob)
2. **판단**:
   - 기존 문서 있음 → 갱신 모드
   - 기존 문서 없음 → 생성 모드
3. **리서치**: researcher 에이전트 또는 직접 WebSearch/WebFetch로 공식 문서 탐색
4. **문서 작성**: `.claude/rules/knowledge-management.md`의 포맷에 따라 작성
5. **신뢰도 태그**: `.claude/rules/source-verification.md`에 따라 태그 부여
6. **저장**: 적절한 카테고리 디렉토리에 저장
   - Claude Code 관련 → `knowledge/claude-code/`
   - 프롬프팅 관련 → `knowledge/prompting/`
   - 모범 사례 → `knowledge/best-practices/`
7. **인덱스 갱신**: `knowledge/README.md` 및 카테고리 README 업데이트

## 출력 예시
```
✅ 지식 저장 완료: knowledge/claude-code/hooks.md
- 신뢰도: [VERIFIED]
- 출처: https://docs.anthropic.com/...
- 핵심: hook은 이벤트 기반 자동 실행 스크립트...
```

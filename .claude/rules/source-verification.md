# 출처 검증 규칙

## 1차 출처 목록 (신뢰도 최상)

- Anthropic 공식 문서: https://docs.anthropic.com
- Claude Code GitHub: https://github.com/anthropics/claude-code
- Anthropic 블로그: https://www.anthropic.com/blog
- Claude Code npm: https://www.npmjs.com/package/@anthropic-ai/claude-code

## 2차 출처 (교차 검증 필요)

- 커뮤니티 블로그, 튜토리얼
- Stack Overflow, Reddit 답변
- 비공식 GitHub 리포지토리

## 검증 프로세스

1. **공식 문서 먼저 확인**: 항상 1차 출처에서 정보를 찾는다
2. **교차 검증**: 2차 출처 정보는 1차 출처로 확인한다
3. **날짜 확인**: 정보의 작성일/갱신일을 확인한다
4. **버전 확인**: Claude Code 버전에 따라 동작이 다를 수 있다

## 신뢰도 태그

| 태그 | 의미 | 기준 |
|------|------|------|
| `[VERIFIED]` | 검증됨 | 공식 문서에서 확인, 또는 직접 테스트 완료 |
| `[LIKELY]` | 높은 확률 | 신뢰할 수 있는 출처이나 공식 확인은 안 됨 |
| `[UNVERIFIED]` | 미검증 | 커뮤니티 정보, 추측, 또는 오래된 정보 |

## 규칙

- 신뢰도 태그 없는 지식 문서는 저장하지 않는다
- `[UNVERIFIED]` 태그 문서는 갱신 시 우선적으로 검증한다
- 출처 URL이 깨진 경우 재검색하여 업데이트한다

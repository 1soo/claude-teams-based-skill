# 통합 테스트 시나리오 · 결과 규칙

## 규칙

- 커버리지: 도메인의 모든 요구사항/기능(EARS 인수 기준)을 최소 1개 이상의 테스트 항목으로 다룬다. 정상 경로와 예외(Unwanted) 경로를 모두 포함.
- **빌드 테스트 신뢰 정책**: 각 개발 에이전트(UI/FE/BE/DB)가 이미 통과시킨 개별 빌드 테스트는 다시 실행하지 않는다(`dev-lead`가 전달한 완료 보고를 신뢰). 시나리오의 사전 조건에는 **여러 영역이 이번에 처음 함께 동작하는 통합 빌드/구동 테스트 1회**만 포함한다(예: 프런트+백엔드+DB를 함께 기동해 정상 구동되는지 확인).
- 추적성: 각 항목은 `@docs/01_analyze/...` 로 근거 요구사항/기능을 링크한다.
- 시나리오에 명시된 내용만 수행한다.

## playwright 사용 규칙 (토큰 최적화)

- **격리**: 매 항목마다 새 창(새 context)에서 수행, storage 초기화.
- 요소를 찾을 때는 전체 페이지를 담는 `browser_snapshot` 대신 **`browser_find`**를 우선 사용한다(스니펫만 반환되어 훨씬 저렴하다).
- `browser_snapshot`이 필요하면 **`target`으로 확인할 영역만 좁히거나 `depth`로 트리 깊이를 제한**한다. 전체 페이지 스냅샷은 레이아웃 전반을 봐야 할 때만 사용한다.
- **`browser_take_screenshot`은 시각적 확인이 꼭 필요할 때만** 사용한다. 스크린샷으로는 액션을 수행할 수 없고 이미지라 텍스트 스냅샷보다 비싸므로 기본 검증 수단으로 쓰지 않는다.
- 증적(스냅샷/콘솔 로그/네트워크 요청)은 대화에 그대로 담지 않고 **`filename` 파라미터로 `result/{domain}/` 하위 파일에 저장**한 뒤, 결과 문서에는 경로만 남긴다.
- 콘솔 로그(`browser_console_messages`)는 기본 `level: error`로 필요한 심각도만 조회하고, 세션 전체 이력(`all: true`)은 꼭 필요할 때만 사용한다.
- 네트워크 요청(`browser_network_requests`)은 `filter`로 검증 대상 API만 좁혀 조회하고, 정적 리소스(`static: true`)는 검증에 필요할 때만 포함한다.
- 대기는 임의 반복 조회 대신 **`browser_wait_for`의 `text`/`textGone`** 으로 조건부 대기한다.
- 테스트 케이스가 끝나면 **`browser_close`(또는 `browser_tabs` action: close)로 즉시 컨텍스트를 정리**한다.

## 시나리오 양식 — `docs/04_test/{yyyyMMdd-HHmmss}/{domain}/scenario.md`

```markdown
# 통합 테스트 시나리오 — {도메인명}

## 사전 조건
- 통합 빌드/구동 테스트 통과 (개별 영역 빌드는 각 개발 에이전트 완료 보고로 갈음, 재실행하지 않음)
- {필요한 데이터/계정/역할}

## 시나리오

### TC-{약어}-001 · {테스트명}
- 근거: @docs/01_analyze/feature/{domain}.md (FEAT-...), @docs/01_analyze/prd/{domain}.md (REQ-...)
- 전제: {상태}
- 절차:
  1. {단계}
  2. {단계}
- 기대 결과: {EARS 인수 기준에 대응하는 기대 동작}
```

## 결과 양식 — `docs/04_test/{yyyyMMdd-HHmmss}/{domain}/result/{domain}.md`

```markdown
---
date: {yyyyMMdd-HHmmss}
domain: {domain}
result: pass | fail | partial
keywords: [{핵심개념1}, {핵심개념2}]
---

# 통합 테스트 결과 — {도메인명} ({yyyyMMdd-HHmmss})

## 요약
- 총 {N}건 · 성공 {N} · 실패 {N}

## 상세

| TC ID | 결과 | 실제 동작 | 비고(스크린샷/로그) |
|-------|------|-----------|----------------------|
| TC-{약어}-001 | PASS / FAIL | {관찰된 결과} | {증적} |

## 실패 항목 분석
- TC-... : {원인 / 재현 조건 / 관련 요구사항}
```

- result: 상세 표의 실패 건수가 0이면 pass, 전부 실패면 fail, 일부 실패면 partial.
- keywords: 이 테스트가 다루는 핵심 도메인 개념 2~5개.
- 프런트매터는 `result/{domain}.md`에만 채운다(`scenario.md`는 대상 아님).

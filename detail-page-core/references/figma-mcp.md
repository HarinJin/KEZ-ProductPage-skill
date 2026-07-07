# Figma MCP 조작 규칙 (use_figma API — HOW-TO)

> use_figma API 자체의 규칙만. "무엇을/왜"(디자인 결정)는 `design-dna/*`, 제작 순서·코드 패턴은 `build-recipes.md`.
> use_figma 호출 전 반드시 figma-use 스킬(또는 `skill://figma/figma-use/SKILL.md` 리소스)을 로드한다. 로드 시 `skillNames`에 `resource:figma-use` 명시.

## 읽기: 2단계로만

1. `get_metadata(fileKey, nodeId?)` — 구조 파악, 저토큰. nodeId 생략 시 페이지 목록.
2. 필요한 nodeId만 `get_design_context(fileKey, nodeId)` — 코드+스크린샷+메타데이터.
3. **파일 전체를 한 번에 덤프하지 않는다.** 섹션 단위로 nodeId 스코핑.
4. 시각 확인은 `get_screenshot(fileKey, nodeId, maxDimension?)` — 기본 URL 방식(토큰 절약). 미리보기가 저해상도라 겹침/잘림이 실제보다 과장돼 보일 수 있음 → 의심되면 해당 노드만 고해상도 재조회 또는 `get_metadata`로 좌표 확인.

## 쓰기: use_figma 5규칙

1. **폰트 로딩**: `await figma.loadFontAsync(fontName)` → 완료 후에만 텍스트/`appendChild`/`insertChild`. 스킵 시 `"Cannot write to node with unloaded font"`. 기존 텍스트 편집 시 `getStyledTextSegments(['fontName'])`로 현재 폰트를 읽어 로드(하드코딩 금지). **착수(스켈레톤) 단계에서 브랜드 프로파일의 모든 폰트를 `listAvailableFontsAsync()`로 1회 전수 확인**하고 결과(가용/미보유+대체 규칙)를 이후 전 작업·모든 빌더 프롬프트에 전파 — 작업자마다 재확인시키지 않는다(→ `troubleshooting.md` T7). 미보유 시 브랜드 프로파일의 대체 규칙을 따르거나 유저에게 업로드 요청 — 임의 유사 폰트 대체 금지.
2. **오토레이아웃 순서**: `parent.appendChild(child)` **먼저** → 그 다음 `child.layoutSizingVertical/Horizontal='FILL'`. 순서 바뀌면 실패. 텍스트 래핑: `textAutoResize='HEIGHT'` + 명시적 width(`FIXED`+`resize()`). `FILL`만 주면 collapse. 자식=`layoutSizing*`(FIXED|HUG|FILL), 프레임=`primaryAxis/counterAxisSizingMode`(FIXED|AUTO) — 섞지 말 것.
3. **필수 async await**: `setCurrentPageAsync`(페이지 전환, 동기 setter는 throw), `loadFontAsync`, `node.screenshot()`. await 누락=silent failure. 한 스크립트당 `setCurrentPageAsync` 1회 — 멀티페이지는 병렬 fan-out.
4. **반환값**: 영향받은 노드ID 전부 return(`return {createdNodeIds:[...], mutatedNodeIds:[...]}`). 에이전트는 return 값만 봄.
5. **색상 0–1 범위**(0–255 아님). `{r:1,g:0,b:0}`=빨강. Paint: `{type:'SOLID', color:{r,g,b}, opacity}`. fills/strokes는 read-only 배열 → 새 배열로 재할당.

## 성능/분할

- **한 호출당 논리 연산 ≤ 10개**(생성+속성+parenting=1세트). 20노드면 2~3회로 분할.
- 빌드 패턴: 스켈레톤(`placeholder=true`) → 섹션별 증분 채우기(`placeholder=false`) → `get_metadata`+`screenshot` 검증.
- 여러 독립 섹션 동시 제작은 루프 대신 병렬 fan-out(섹션당 use_figma 호출 하나).
- **원자적 실행**: 스크립트 에러 시 아무 변경도 반영 안 됨(고아 노드 없음) → 에러 읽고 고쳐 재시도. 즉시 재시도 금지.
- use_figma는 베타 — 결과는 항상 스크린샷 재확인.

## 자주 나오는 함정

에러·이상 렌더의 증상별 색인은 → **`troubleshooting.md`** (textAutoResize 순서, 중첩 줄바꿈 미계산, hug 재계산, query 한글 셀렉터, 소수점 높이, 크로스파일 자산, 폰트, 흰배경 래퍼, union 리셋, resize 왜곡, 긴 프레임 검수, 렌더 아티팩트, 크롭 검수). 새로 해결한 함정도 그 문서에 축적한다.

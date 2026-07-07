# 공개 전 체크리스트

이 리포는 현재 **비공개**. 아래를 완료하면 바로 공개 전환 가능하도록 정리해 둠. (2026-07-08 기준)

## 1. 스크린샷 보완 — `docs/assets/`에 넣고 README의 `<!-- TODO -->` 위치에 삽입

README에 주석으로 자리를 표시해 둠 (`<!-- TODO(공개 전): ... -->` 검색):

- [ ] **hero.png** — 완성된 상세페이지 캔버스 전경 1장 (README 최상단). 공용판이므로 실브랜드 노출 없는 샘플/데모 브랜드 권장
- [ ] **온보딩 대화** 1장 — `_template.md` 복사 → 슬롯 확인 질문 장면
- [ ] **제작 과정** 2~3장 — 스켈레톤(placeholder) 단계 → 섹션 완성 단계 대비
- [ ] (선택) 트러블슈팅 before/after 1장 — T13 렌더 아티팩트나 T2 줄바꿈 사례

## 2. 데모 영상

- [ ] 30초~1분: "리뷰 섹션 만들어줘" → 인터뷰 질문 → 캔버스에 프레임 생성 → 스크린샷 검증까지 한 사이클
- [ ] GitHub README는 mp4 직접 재생 불가 → GIF 변환 또는 유튜브 링크. GIF는 10MB 이하 권장

## 3. 콘텐츠 최종 점검

- [ ] **누출 재검증** — 스크린샷·영상 추가 후 반드시 재실행 (이미지 파일명·영상 속 캔버스에 실브랜드 노출 주의):

```bash
cd <리포 루트>
grep -rniE '바비야기|에피랜드|브뤼|진하린|kezilac|jinharin' . --exclude-dir=.git
grep -rnE 'wRtk0YLDHqM3FpDZ9qMzIh|O22tnzWjSFqBgvjSCHshnE' . --exclude-dir=.git
grep -rniE '#37865a|#61cd75|#2062AE|#7c7b6f|#f7f2df|#4c251e|#ff522c' . --exclude-dir=.git
grep -rniE 'Paperlogy|AggroOTF|GyeonggiBatang|SUITE' . --exclude-dir=.git
```

전부 0건이어야 함. (텍스트 13파일은 2026-07-08 추출 시 3중 검증 통과 완료 — 이후 변경분만 주의)

- [ ] README의 `<!-- TODO -->` 주석 전부 제거
- [ ] 라이선스 결정 → `LICENSE` 파일 추가 + README 라이선스 섹션 갱신
  - 참고: 코드가 아닌 디자인 시스템 문서 → MIT(제한 없음) vs CC BY-NC 4.0(상업 이용 제한) 중 선택
- [ ] Figma MCP 등록 명령어가 여전히 유효한지 확인 (`https://mcp.figma.com/mcp`)

## 4. 공개 전환

```bash
gh repo edit HarinJin/KEZ-ProductPage-skill --visibility public --accept-visibility-change-consequences
```

- [ ] 리포 Description·Topics 설정 (예: `claude-code`, `figma`, `skill`, `design-system`, `ecommerce`)
- [ ] (선택) 릴리스 태그 v1.0.0

## 메모 — 개인 환경 관련 (공개와 무관)

- 개인 머신에는 `detail-page-builder`(브랜드 값 포함 개인판)와 `detail-page-core`(이 공용판)가 **같은 트리거("상세페이지")를 공유** — 개인 환경에서는 core를 비활성화하거나 트리거를 좁히는 정리가 필요 (개발일지 P09_S02 "남긴 문제의식")
- 이 리포의 정본은 `~/.claude/skills/detail-page-core/`. 스킬을 수정하면 리포에도 복사 → 커밋 → 누출 grep 재실행

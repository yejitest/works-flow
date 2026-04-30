# works-flow — Claude Code 프로젝트 컨텍스트

NAVER WORKS 메일 + Flow AI 패널 데모 (`demo.html` 단일 파일 SPA). Figma 시안과 픽셀 단위 매칭이 목표.

## 작업 원칙

- **텍스트 콘텐츠 보존**: 폴더명·메일 본문 등 데모 데이터(2분기 OKR, Aurora 프로젝트, Tokyo Office, 글로벌 파트너십, BTS, Zeplin 등)는 변경 금지. 디자인(아이콘·컬러·간격·크기)만 Figma에 맞춤.
- **Figma 원본 사용**: 손그린 SVG·근사값 금지. 항상 Figma API로 SVG export하고 `absoluteBoundingBox` 좌표를 절대값으로 추출해 그리드·여백 계산.
- **컬러는 Figma `fills[].color`** 그대로 hex 변환.
- **사용자 거부 사례**: "아이콘이랑 간격 등이 너무 달라", "디자인이 여전히 너무 달라 ㅠㅠ 아예 똑같이좀 맞춰줘" → 처음부터 정확히.

## 기술 노트

### SVG data URL 인코딩 (필수)
CSS `mask-image: url("data:image/svg+xml;utf8,<svg ... fill='#3F4247'>")` 형식에서 `#`을 그대로 두면 fragment identifier로 해석되어 아이콘이 깨짐. 반드시 `%23`으로 인코딩.

```python
import re
s = re.sub(r"#([0-9A-Fa-f]{3,8})", r"%23\1", svg_string)
s = s.replace('"', "'")  # outer url("...") 충돌 방지
```

### 좌표 변환
Figma 절대 좌표 → 메일리스트 행 기준: `relative_x = abs_x - 274`. (참고 frame 35:4621 기준 offset OFFX=6584, OFFY=1584)

## Figma 레퍼런스

### Active design file: `yQUb8nHvyLUWj0dxGpbdS5` (🚫memo🚫)

| 노드 | 의미 | 크기 |
|---|---|---|
| 35:4621 | 메일_목록 (전체 화면) | 1280x1024 |
| 35:5159 / 35:5160 | LNB 사이드바 | 250x966 |
| 35:4622 | BTN 그룹 (메일 리스트) | 982x649 |
| 35:5067 | BTN_Top (툴바) | - |
| 35:5808 | SaaS_GNB (상단바) | 1280x58 |
| 35:5130 | Pagination | - |

**LNB 아이콘 노드**:
- 35:5172 allmail / 35:5181 inbox / 35:5190 etcmail / 35:5208 memo
- 35:5221 mymail (folder) / 35:5240 sendmail / 35:5249 receipt / 35:5259 underdepth
- 35:5218 arrow_open / 35:5286 ic8_arrow_down
- 35:5332 bookmark (중요) / 35:5336 remind / 35:5340 to (받는사람)
- 35:5349 trash / 35:5355 spam

**메일리스트 아이콘 노드**:
- 35:4643 mail_unread / 35:4811 mail_read / 35:4878 mail_reply / 35:4913 mail_forward
- 35:4644 bookmark / 35:4645 favorit_line / 35:4664 uncheck
- 35:5072 viewtype / 35:5077 recent / 35:5073 arrow_down / 35:5129 more2

### 이전 파일: `9igBIJX4JQezt5SIpQba5f` (PC Icon set, node `1:17450`)
초기 LNB 아이콘 라이브러리. 현재 위 yQUb8nH... 가 메인.

### Figma API 호출
```bash
# 토큰은 로컬 메모리에 저장됨 (공개 repo에 커밋 금지)
TOKEN="<personal-access-token>"

# 노드 구조 조회
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/files/yQUb8nHvyLUWj0dxGpbdS5/nodes?ids=35:4622&depth=6"

# SVG export
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/yQUb8nHvyLUWj0dxGpbdS5?ids=35:4622&format=svg"
```

토큰은 사용자에게 다시 받거나 로컬 `~/.claude/projects/.../memory/figma_refs.md` 참조.

## 디자인 토큰 (Figma 추출)

```css
--nw-corp:#0CC759;       /* 메일쓰기 버튼 그린 */
--nw-blue:#118DFF;       /* 선택/카운트 메인 블루 */
--nw-blue-cnt:#3591FF;   /* 카운트 숫자 */
--nw-blue10:rgba(17,141,255,.10); /* 선택된 행 배경 */
--nw-bg:#F9FAFC;         /* 사이드바 배경 */
--nw-gray500:#71767A;    /* 라벨 텍스트 */
--nw-gray800:#3F4247;    /* 메뉴 기본 텍스트 */
--nw-gray900:#1D1F23;
```

**메일리스트**:
- 행: 40px 높이, 폰트 14px, 구분선 `#F1F3F9`
- 그리드: `32px 32px 32px 136px 1fr 76px 80px` (chk/star/mail/from/subject/date/size)
- 안읽음 텍스트: `#202124` / 읽음: `#3F4247`
- 봉투: 안읽음 채움 `#93B4EE`, 읽음 윤곽 `#C6CBCE`
- 별: ON `#FFB032`, OFF `#C6CBCE`

## 진행 상황

**완료** (2026-04-29):
- LNB 사이드바 — 14종 아이콘 Figma SVG, 토큰 적용, 카운트 인라인 배치
- 메일리스트 (BTN 그룹) — 7컬럼 그리드, 봉투/별 Figma SVG, 용량 컬럼

**미완료**:
- GNB 상단바 — 간격·아이콘 정렬 (사용자 피드백: "GNB 간격 그대로 반영 안됐고")
- 툴바 — 버튼 재구성 (모두 읽음 / 삭제 / 스팸신고 / 답장 / 전체 답장 / 전달 / 이동▾ / 리마인드▾ / ⋯ + 우측 filter/clock/list 드롭다운)

## 커밋 관행

- 커밋 메시지 한국어 (`style(demo): ...`, `feat(demo): ...`)
- 사용자가 "깃허브에 업데이트" 요청 시: `git add demo.html && git commit && git push` (main 브랜치 직접)

# 10_21 프로젝트 (AX 강의 · API 화면 실습)

키가 필요 없는 공개 API를 불러와 보여 주는 HTML 화면 모음이다. 작업 파일은 `git-practice/d6-api/`에 있다.
`D06_1021.html`은 수업 워크북이라 작업 대상이 아니다. 건드리지 않는다.

API 화면의 필수 규칙은 `.claude/rules/api.md`에 있고 자동으로 적용된다. 여기서는 그 규칙을 반복하지 않고, 이 프로젝트에서 정해진 것만 적는다.

## 화면 목록 (`git-practice/d6-api/`)

| 파일 | 내용 | 쓰는 API |
| --- | --- | --- |
| `weather.html` | 서울 날씨 (지금·12시간·7일) | Open-Meteo forecast |
| `rate.html` | 외화 → 원화 계산 + 최근 30일 추이 미니 차트(SVG, 짚으면 그날 값) | Frankfurter (latest, 시계열 `/시작일..`) |
| `city.html` | 도시 검색 → 기온 + 미세먼지 등급 | Open-Meteo 지오코딩 → forecast → air-quality |
| `morning.html` | 아침 대시보드 (날씨·미세먼지, 환율, 공휴일 D-day, 강아지, 오늘의 역사) | Open-Meteo, Frankfurter, Nager.Date, Dog CEO, Wikipedia(onthisday + langlinks) |
| `color.html` | 색 코드 → 이름·보색·유사색 팔레트 | The Color API |
| `index.html` | 위 weather·city·rate·morning을 한 파일로 합친 허브. 탭 3개: 날씨(weather+city) / 돈(rate) / 오늘(morning). `color.html`은 아직 안 합침 | 위 화면들의 API 전부 |

`index.html`은 원본 파일을 지우지 않고 합친 사본이다. 탭은 처음 열 때만 API를 부르고, 고른 탭은 URL 해시(`#weather` `#money` `#today`)에 저장한다. 화면별로 `const API = { weather:{}, city:{}, rate:{}, morning:{} }`처럼 이름을 묶고, 요소 id는 `w-` `c-` `r-` `m-` 접두어, JS는 화면마다 IIFE로 나눠 서로 겹치지 않게 했다. 한쪽(원본 또는 `index.html`)을 고치면 다른 쪽도 같이 고쳐야 한다. 자동으로 맞춰지지 않는다.

모든 화면은 파일 하나에 CSS·JS를 담고 더블클릭으로 열린다. 새 화면을 만들 때도 같은 형태로 만든다.

## 모든 화면에 공통으로 들어가는 것

### 다크/라이트 스위치
새 화면에도 반드시 넣는다. 기준은 `weather.html`이고, 만들 때는 아래 세 덩어리를 그대로 복사한다. 모양을 새로 만들지 않는다.
- CSS: `main { position:relative; }`부터 `.theme` 규칙들과 `prefers-reduced-motion` 줄까지
- HTML: `<main>` 바로 안 첫 줄의 `<button class="theme" role="switch" …>`
- JS: `// 라이트/다크 토글` IIFE (`localStorage`의 `theme`에 저장, 없으면 기기 설정을 따름)

팔레트는 변수로 정의한다. 하나를 반드시 지켜야 한다.
- 라이트 `:root`에 `--edge:var(--line)` 같은 토글용 변수와 `--ease`를 둔다.
- 다크는 `:root[data-theme=dark]`와 `@media (prefers-color-scheme:dark){ :root:not([data-theme=light]) }` 두 곳에 같은 값을 넣는다. `data-theme`로 고른 값이 시스템 설정보다 우선한다.
- 다크 배경 `#0b0f17`, 카드 `#161b26`, 글자 `#f5f5f7` 계열을 쓴다. 버튼·강조색은 흰 글자가 읽히는 `#3b6fe0`.
- `h1`에 `padding-right:5rem`을 줘서 스위치와 제목이 겹치지 않게 한다.

### 카드 구성
카드마다 로딩 중 문구와 실패 화면을 따로 둔다. 카드 하나(또는 한 카드 안의 API 하나)가 실패해도 나머지는 보여야 한다. 한 카드가 API를 둘 쓰면 `Promise.allSettled`로 따로 처리한다.
맨 아래에는 "🔌 이 화면이 쓰는 API" 상자를 둔다.

## 문구(UX 라이팅) 규칙
- 말투는 해요체로 통일한다. 번역투와 명사 나열("~한 모델 추정값이라")을 피하고, 한 문장에 정보 하나만 담는다.
- 실패 문구는 "무슨 일이 생겼는지 → 사용자가 할 일" 순서로 쓴다. 기본형은 "불러오지 못했어요. 잠시 뒤 다시 시도해 주세요." 화면별로 동사만 바꾼다(다시 검색해/골라/찾아 주세요).
- 영어 오류 문구(`Failed to fetch`)나 HTTP 번호를 그대로 보여 주지 않는다. 각 파일의 `whyFailed(e)`가 네트워크 오류는 "인터넷 연결을 확인해 주세요.", HTTP 오류는 "(서버 오류 404)"로 바꿔 붙인다. 새 화면에도 같은 함수를 넣는다.
- 사용자가 고칠 수 있는 실패(검색 결과 없음 등)는 괄호 속 코드 없이 해결 방법만 말한다(`err.friendly = true`로 던지면 메시지만 보인다).
- 앞 단계가 실패해서 못 보여 주는 카드는 "건너뛰었어요"가 아니라 이유를 말한다("도시를 먼저 찾아야 볼 수 있어요.").
- 미세먼지 안내 문구는 한 곳의 문장을 그대로 쓴다: "환경부 등급은 하루 평균 기준이에요. 여기서는 지금 값(모델 추정)에 맞춰 본 거라 에어코리아 발표와 다를 수 있어요."
- 접근성 라벨(`aria-label`)도 소리 내어 읽어 어색하지 않게 쓴다("다시 찾기: Cerulean #24B1E0").

## 모션·성능 규칙
- `transform`과 `opacity`만 애니메이션한다. `transition: all`은 쓰지 않는다.
- 등장 효과는 `translateY(8px)` + `opacity`, 곡선은 `cubic-bezier(.23,1,.32,1)`(`--ease`), 350~500ms 이내.
- 눌림 효과는 `scale(.96~.97)`, 120ms, `ease-out`.
- `prefers-reduced-motion`일 때는 이동을 빼고 투명도 변화만 남긴다.
- 블러는 `backdrop-filter: blur(14px)` 이하로 쓴다. 배경 그라디언트는 `background-attachment:fixed` 대신 `body::before { position:fixed; inset:0; z-index:-1 }` 레이어에 둔다(스크롤 때마다 다시 그리지 않게). 불투명한 어두운 카드에는 블러를 쓰지 않는다.
- 터치 대상은 44px 이상으로 한다.

## API별로 알아 둔 것
- **Open-Meteo 지오코딩:** "서울", "부산" 같은 짧은 한글 이름은 못 찾거나 엉뚱한 곳(경북의 작은 마을 Pusan)을 준다. "Seoul", "Busan", "서울특별시"는 정확하다. 기본 검색어는 영어로 두고, 결과가 없으면 영어·정식 이름으로 검색하라는 안내를 보여 준다. `count=10`으로 받아 인구(`population`)가 가장 많은 곳을 고른다.
- **미세먼지 등급(환경부):** PM10 좋음 ≤30 · 보통 ≤80 · 나쁨 ≤150 · 그 위 매우 나쁨. PM2.5는 ≤15 · ≤35 · ≤75 · 그 위. 이 구간은 하루 평균용이고 API는 지금 시각의 모델 추정값이라, 화면에 "에어코리아 발표와 다를 수 있다"고 적는다. 등급은 화면에 보이는 반올림한 값으로 매긴다(30.4가 "30"으로 보이는데 "보통"이 되지 않게).
- **Frankfurter 환율:** 유럽중앙은행이 하루 한 번 발표하는 기준값이다. "실시간"이라고 쓰지 않고 기준 날짜를 함께 보여 준다. `amount=100`으로 100엔 환산이 한 번에 된다. 시계열은 `/YYYY-MM-DD..?base=USD&symbols=KRW`(끝 날짜 생략 시 최신까지)이고, 주말·휴일은 빠져서 영업일 값만 온다(30일 ≈ 22개). 그래프는 날짜가 아니라 값 순서로 x축을 잡는다.
- **Nager.Date 공휴일:** 대체공휴일·임시공휴일이 빠질 수 있다. D-day의 "오늘"은 한국 시간(`Asia/Seoul`)으로 계산한다.
- **Wikipedia 오늘의 역사:** 키 없음, CORS `*`. 한국어판(`/wikipedia/ko/onthisday/...`)은 "language not yet supported"(404)라서 영어판 `https://api.wikimedia.org/feed/v1/wikipedia/en/onthisday/selected/MM/DD`를 쓴다(선정 사건 13건 안팎, 본문은 영어). 한국어 문서 제목은 `https://en.wikipedia.org/w/api.php?action=query&prop=langlinks&lllang=ko&redirects=1&origin=*&titles=A|B`로 한 번에 조회하고, 이 호출이 실패해도 영어 목록은 보여 준다. 날짜 MM/DD는 한국 시간 기준. "다른 사건" 버튼은 받아 둔 목록을 넘겨 볼 뿐 API를 다시 부르지 않는다. 출처에 Wikipedia(CC BY-SA)를 적는다.
- **Dog CEO:** 이미지 주소만 주므로 `Image`로 파일을 다 받은 뒤 보여 준다. 새로고침은 이 카드만 다시 부른다.
- **The Color API:** 키 없음, CORS 허용. 엉터리 색 코드(`ZZZZZZ`)를 넣어도 200과 `NaN`을 주므로 부르기 전에 화면에서 6자리 hex인지 먼저 검사한다. 색 이름은 영어로만 온다. `scheme`의 `colors[0]`이 보색이고 `contrast.value`가 그 색 위에 읽기 좋은 글자색이다.
- **REST Countries는 쓰지 않는다:** v3.1(`restcountries.com/v3.1/...`)은 폐지돼 에러 JSON으로 리다이렉트되고, 새 v5는 모든 엔드포인트가 API 키를 요구하며 브라우저에서는 허용 도메인 등록까지 필요하다. "이 API는 키가 필요해서 D8 방식이 필요해요"로 안내한다(데모 키 `rc_live_demo`는 샘플만 준다).
- 시간은 모두 한국 시간 기준으로 표시한다.

## 작업 방식
- 화면을 만들거나 고친 뒤에는 헤드리스 Chrome으로 직접 열어 확인한다(`chrome.exe --headless=new --virtual-time-budget=15000 --screenshot=… file:///…`). 라이트는 `<html data-theme="light">`, 다크는 `data-theme="dark"`를 임시 사본에 붙여 찍는다. 임시 파일은 프로젝트 안에 두지 않는다.
- 카드 하나의 주소를 일부러 틀리게 해서 나머지 카드가 보이는지도 확인한다.
- 실제 기기에서 느낌을 봐야 하는 것(스크롤 부드러움, 스위치 손맛)은 확인하지 못했다고 솔직히 적는다.
- 커밋·푸시는 하지 않고 커밋 메시지만 제안한다.

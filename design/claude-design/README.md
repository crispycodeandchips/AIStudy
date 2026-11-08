# Claude Design 시안 — 들꽃 웨딩페어 랜딩 & 신청자 관리

가져온 날: 2026-11-08
원본 캔버스: https://claude.ai/artifact/JJL7gfDxS2QAGy3aRxFDeu (버전 1794122751-c81c, 2026-11-08 두 번째 시안으로 갱신)
근거 문서: [../design-brief.md](../design-brief.md), [../claude-design-prompt.md](../claude-design-prompt.md)

이 폴더는 Claude Design 캔버스의 화면 원본을 그대로 내려받은 사본이다. 화면 구현 SPEC은 이 파일을 시각 기준으로 삼는다. 캔버스를 고치면 이 폴더도 다시 내려받아 맞춘다.

## 화면 목록

| 파일 | 화면 | 크기 |
|------|------|------|
| [Main.dc.html](project/Main.dc.html) | 1a. 기본 랜딩 (`/`) · PC | 1440 |
| [LandingMobile.dc.html](project/LandingMobile.dc.html) | 1a. 기본 랜딩 · 모바일 | 390 |
| [FairLanding.dc.html](project/FairLanding.dc.html) | 1b. 회차 랜딩 (`/fairs/[회차]`) · PC | 1440 |
| [FairMobile.dc.html](project/FairMobile.dc.html) | 1b. 회차 랜딩 · 모바일 (전체: 히어로~신청 폼·푸터) | 390 |
| [FormStates.dc.html](project/FormStates.dc.html) | 2. 신청 폼 상태: 입력 오류 / 제출 중 / 서버 오류 | 1440 |
| [Complete.dc.html](project/Complete.dc.html) | 3. 신청 완료 · PC (제출 직후, 요약 있음) | 1440 |
| [CompleteMobile.dc.html](project/CompleteMobile.dc.html) | 3. 신청 완료 · 모바일 (새로고침·직접 접근, 요약 없음) | 390 |
| [Privacy.dc.html](project/Privacy.dc.html) | 4. 개인정보 처리방침 · PC | 1440 |
| [PrivacyMobile.dc.html](project/PrivacyMobile.dc.html) | 4. 개인정보 처리방침 · 모바일 | 390 |
| [EmptyStates.dc.html](project/EmptyStates.dc.html) | 예외 상태: 신청 가능 회차 없음 / 회차 종료 / 비공개(404) | 1440 |
| [Dashboard.dc.html](project/Dashboard.dc.html) | 5. 관리자 신청 현황판 (Payload 안) | 1440 |
| [DashboardEmpty.dc.html](project/DashboardEmpty.dc.html) | 5. 관리자 현황판 · 데이터 없음 | 1440 |
| [Tokens.dc.html](project/Tokens.dc.html) | 디자인 토큰 | — |
| [Components.dc.html](project/Components.dc.html) | 기본 컴포넌트 · 상태별 | — |
| [canvas.json](project/canvas.json) | 캔버스 배치·제목·메모 | — |

## 캔버스 메모 요약

- `[예시]` 표시: 회차 이름·날짜·장소·혜택·시간대처럼 실제 값이 아직 없는 자리다.
- `[대괄호]`만 있는 값(상호·주소·연락처 등)은 운영자가 Payload에 넣는다.
- 수량·금액·할인율·누적 신청자 수 같은 숫자는 넣지 않았다.
- 관리자 현황판의 숫자는 모두 0이고, 이름·시각은 `[예시]`다.
- 동의문 전문은 `[법률 검토 중인 동의문 자리]`, 처리방침 본문은 `[운영자 입력·법률 검토 필요]`로만 두었다.
- 사진 자리는 회색 상자 + 설명 문구다.
- 로고 옆에 `[가칭]` 표시가 붙는다.
- 신청 완료(PC) 요약에는 회차·방문 날짜·방문 시간대만 보여 준다. 이름·휴대폰 번호·인원은 표시하지 않는다.
- 모바일 하단 고정 신청 버튼: 높이 56px, `#apply` 신청 폼 섹션이 화면에 들어오면 숨긴다. 페이지 맨 아래에 같은 높이(56px + 안전 영역)의 여백을 둔다.

## 구현할 때 주의할 점

- `.dc.html`은 Claude Design 전용 형식이다. `./support.js`와 `/_blob/...` 글꼴 주소는 캔버스 안에서만 동작하므로, 이 파일을 브라우저로 바로 열면 모양이 깨질 수 있다. 화면을 눈으로 볼 때는 원본 캔버스 링크를 쓴다.
- 코드로 옮길 때는 인라인 스타일을 그대로 복사하지 않는다. `tech.md`의 결정대로 색·간격·모서리 값은 전역 CSS 변수 토큰(`Tokens.dc.html` 기준)으로, 화면별 스타일은 CSS Modules로 옮긴다.
- 글꼴은 캔버스의 업로드 파일 대신 `next/font`로 자체 호스팅한다(Pretendard: `next/font/local`, Noto Serif KR: `next/font/google`).
- 관리자 현황판은 Payload 관리자 기본 스타일을 따른다. 방문자 토큰을 관리자 화면에 섞지 않는다.

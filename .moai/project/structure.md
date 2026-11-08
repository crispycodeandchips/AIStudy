# 프로젝트 구조 (제안)

> 이 문서는 소스 코드가 없는 상태에서 작성한 **제안안**이다. 실제 디렉터리 구성과 필드 정의는 `/moai plan` 단계에서 확정한다.

## 구성 방식

Next.js App Router 기반의 단일 애플리케이션이며, Payload CMS 3.x가 같은 Next.js 앱 안에서 동작한다. 랜딩페이지와 관리자 화면·API가 하나의 프로젝트에 함께 있고, 데이터베이스는 Docker로 실행하는 PostgreSQL을 쓴다.

## 디렉터리 배치

```
.
├── src/
│   ├── app/
│   │   ├── (frontend)/          # 방문자용 랜딩 라우트
│   │   │   ├── layout.tsx       # 방문자 레이아웃 (tokens.css, base.css를 여기서만 불러옴)
│   │   │   ├── page.tsx         # 기본 랜딩 (/) — 회차 카드 목록 + #apply 신청 폼
│   │   │   ├── fairs/[slug]/    # 회차 랜딩 + #apply 신청 폼 (회차 미리 선택)
│   │   │   ├── apply/complete/  # 신청 완료 화면 (URL에 개인정보 없음)
│   │   │   ├── privacy/         # 개인정보 처리방침 (경로명은 제안)
│   │   │   └── styles/          # tokens.css (CSS 변수 토큰), base.css
│   │   └── (payload)/           # Payload 관리자 화면과 API 라우트
│   │       ├── admin/
│   │       └── api/
│   ├── collections/
│   │   ├── Fairs.ts             # 회차
│   │   ├── Applications.ts      # 신청
│   │   ├── Media.ts             # 배너 등 이미지
│   │   └── Users.ts             # 관리자 계정
│   ├── globals/
│   │   └── LandingPage.ts       # 랜딩 공통 기본값 (헤드라인, 소개, 배너, 혜택, 참여 분야 소개, FAQ, 푸터)
│   ├── lib/
│   │   ├── validation/          # 신청 입력값 검증
│   │   ├── spam-guard/          # honeypot 검사, 신청 횟수 제한
│   │   ├── csv-export/          # 신청자 CSV 생성
│   │   └── utm/                 # UTM 파라미터 수집
│   └── payload.config.ts        # Payload 설정 (DB 어댑터, 컬렉션 등록)
├── tests/
│   ├── unit/                    # Vitest 단위 테스트
│   └── e2e/                     # Playwright 브라우저 테스트
├── docker-compose.yml           # 로컬 PostgreSQL
└── .env.example                 # 환경 변수 견본 (실제 값은 .env, 저장소 제외)
```

## 디렉터리별 역할

| 경로 | 역할 |
|------|------|
| `src/app/(frontend)` | 방문자가 보는 기본 랜딩(`/`), 회차 랜딩(`/fairs/[slug]`), 완료 화면(`/apply/complete`), 개인정보 처리방침. 라우트 그룹으로 관리자 화면과 레이아웃을 분리한다. 완료 화면의 신청 요약은 제출 직후 sessionStorage로만 전달하고 URL에 개인정보를 남기지 않는다. |
| `src/app/(frontend)/styles` | `tokens.css`(CSS 변수 토큰)와 `base.css`. `(frontend)` 레이아웃에서만 불러와 관리자 화면 스타일과 섞이지 않게 한다. 컴포넌트는 각자 `*.module.css`를 쓴다. 관리자 현황판은 별도 CSS Module로 Payload 관리자 화면 스타일을 따른다. |
| `src/app/(payload)` | Payload가 생성하는 관리자 화면(`/admin`)과 REST/GraphQL API 라우트. |
| `src/collections` | Payload 컬렉션 정의. 데이터 모델, 접근 권한, 관리자 화면 표시 방식을 함께 둔다. |
| `src/globals` | 단일 문서형 콘텐츠. 랜딩 공통 기본값(문구·배너·혜택·FAQ 등)을 관리자가 수정한다. |
| `src/lib/validation` | 신청 폼 입력 검증. 클라이언트와 서버가 같은 규칙을 공유한다. |
| `src/lib/spam-guard` | 숨김 입력칸 검사와 신청 횟수 제한. 외부 서비스에 의존하지 않는다. |
| `src/lib/csv-export` | 신청자 목록을 CSV(UTF-8 BOM 포함, Excel 한글 호환)로 변환. 인증된 관리자만 호출할 수 있다. |
| `src/lib/utm` | 첫 방문 시 UTM 파라미터를 보존했다가 신청 시 함께 저장한다. |
| `tests/unit` | 입력값 검증, 중복 판정, 스팸 방지, CSV 변환 로직 테스트. |
| `tests/e2e` | "랜딩페이지에서 신청 → 관리자 화면에서 확인" 흐름 테스트. |
| `docker-compose.yml` | 로컬 개발용 PostgreSQL 컨테이너 정의. |
| `.env.example` | 필요한 환경 변수 목록. 비밀 값은 넣지 않는다. |

## 데이터 모델 (제안)

### Fairs (회차)
| 필드 | 설명 |
|------|------|
| name | 회차 이름 |
| slug | 랜딩 URL 식별자 |
| startDate / endDate | 개최 시작일·종료일 (하루짜리 회차는 같은 날) |
| venueName | 장소명 |
| address | 주소 |
| directions | 오시는 길 안내 문구 (네이버 지도·카카오맵 링크와 함께 표시, 지도 임베드 없음) |
| timeSlots | 시간대 배열(각 항목 label). 새 회차 기본값 4개: 11:00–13:00, 13:00–15:00, 15:00–17:00, 17:00–19:00. 회차마다 수정 가능 |
| status | 공개 상태. 노출 조건: status=open 이고 endDate가 오늘(KST) 이후 |
| heroHeadline | 선택. 비어 있으면 LandingPage 공통 값 사용 |
| heroImage | 선택(Media 관계). 비어 있으면 공통 값 사용 |
| benefits | 선택. 값이 있으면 공통 혜택 목록 전체를 대체(합치지 않음) |

#### 혜택(benefits) 항목 형태 (LandingPage·Fairs 공통)
| 필드 | 형식 |
|------|------|
| icon | 선택: gift / ring / calendar / sparkle / leaf / ticket |
| title | 20자 이내, 필수 |
| description | 60자 이내, 필수 |
| condition | 40자 이내, 선택 |

### Applications (신청)
| 필드 | 형식 | 비고 |
|------|------|------|
| name | 텍스트 | 필수 |
| phone | 텍스트 | 필수, 형식 검증. 중복 판정 전 숫자만 남기도록 정규화 (제안) |
| weddingDate | 날짜 | 선택. null = 미정 |
| weddingRegion | 선택 | 선택. 시·도 17개 + 미정 코드. CSV에는 한글 라벨로 출력 |
| fair | 관계 (Fairs) | 필수 |
| preferredVisitDate | 날짜 | 필수. 서버에서 회차 시작일~종료일 범위인지 검증 |
| preferredTimeSlot | 텍스트 | 선택. 제출 시점의 회차 timeSlots label을 복사해 저장, 서버에서 회차 timeSlots와 대조. null = 미정. 정원 없음 |
| visitorCount | 숫자 | 선택. 방문 인원(본인 포함) 1~4 (4 = 4명 이상), 기본값 2 |
| role | 선택: 신랑 / 신부 | 선택 |
| interests | 복수 선택: 웨딩홀, 스드메, 혼수·가전, 예물, 허니문, 기타 | 선택(고정 목록) |
| privacyConsent | 체크 | 필수 (미동의 시 저장 불가) |
| consentedAt | 일시 | 동의 시각, 서버에서 기록 |
| consentVersion | 텍스트 | 동의한 동의문 버전, 서버에서 기록 |

선택 항목은 모두 null을 허용한다.
| status | 선택: 신규 / 연락완료 / 방문확정 / 방문완료 / 취소 | 기본값 신규 |
| memo | 긴 텍스트 | 관리자 상담 메모 |
| utmSource / utmMedium / utmCampaign | 텍스트 | 유입 경로 (URL 파라미터 `utm_source` 등에서 수집) |
| isDuplicate | 체크 | 같은 회차 + 같은 휴대폰 번호 재신청 시 표시 |
| createdAt | 일시 | 자동 기록 |

방문자는 신청 생성만 가능하고, 조회·수정·삭제는 관리자만 가능하도록 접근 권한을 둔다.

`status`, `memo`, `isDuplicate`, `consentedAt`, `consentVersion`은 방문자 입력을 받지 않고 서버에서만 설정한다(필드 단위 접근 제한). 입력값 검증, 동의 확인, 스팸 방지는 폼뿐 아니라 Payload REST/GraphQL API를 포함한 모든 생성 경로에서 서버 측에 적용한다. 구현 방식(훅, 서버 액션 등)은 `/moai plan`에서 정한다.

### Media
배너 등 랜딩 이미지. 저장소는 배포처 결정 전까지 로컬 저장을 기본으로 한다.

### Users
Payload 인증을 쓰는 관리자 계정. 첫 버전에서는 1개 계정, 역할 구분 없음.

### LandingPage (global)
전 회차 공통 기본값: 헤드라인, 소개 문구, 배너 이미지(Media 관계), 혜택 목록(위 benefits 형태), 참여 분야 소개, FAQ(질문·답변 목록), 푸터 정보.

회차(Fairs)의 heroHeadline·heroImage·benefits에 값이 있으면 회차 값을, 비어 있으면 공통 값을 쓴다. FAQ는 덮어쓰지 않는다(2026-11-08 디자인 브리프 결정).

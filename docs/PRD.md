# PRD: 모임 이벤트 관리 웹 MVP

## 1. 제품 개요

### 배경
수영, 헬스, 친구 모임 등 소규모 정기 모임을 운영하는 주최자는 공지, 참여자 확인, 정산 등 여러 관리 업무를 메신저나 수기로 처리하고 있다. 이를 하나의 웹에서 효율적으로 처리할 수 있는 도구가 필요하다.

### 목표
- 주최자가 이벤트를 생성하고 링크 하나로 참여자를 관리할 수 있도록 한다.
- 참여자는 별도 회원가입 없이 링크 접속만으로 참석 의사를 표현할 수 있도록 한다.
- MVP는 공지/일정 관리, 참여자 관리, 간단 정산에 집중한다.

---

## 2. 사용자 정의

| 구분 | 설명 | 인증 |
|---|---|---|
| 주최자 | 이벤트를 생성하고 관리하는 사람 | 이메일 OTP 로그인 필요 |
| 참여자 | 링크로 접근해 참석 여부를 응답하는 사람 | 비로그인 (이름만 입력) |

**주요 사용 시나리오**: 10~30명 규모의 정기 소모임 (주 1~2회 반복 활동)

---

## 3. MVP 기능 범위

### 주최자 기능
- 이벤트 생성/수정 (제목, 설명, 날짜, 장소, 최대 인원)
- 참여 링크 생성 및 복사
- 참여자 목록 확인 (참석/불참)
- 공지사항 작성
- 정산: 총액 입력 → 참석자 수로 1/N 자동 계산

### 참여자 기능
- 이벤트 정보 확인
- 이름 입력 + 참석/불참 선택
- 현재 참여자 목록 보기 (이름 + 상태)
- 공지사항 읽기

### MVP 제외 항목
- 카풀 매칭 (향후 확장 예정)
- 외부 결제 연동
- 알림 (이메일/푸시)
- 반복 이벤트 자동 생성

---

## 4. 화면 목록

### 주최자 화면 (로그인 필요)

| 경로 | 화면명 | 주요 내용 |
|---|---|---|
| `/protected/events` | 이벤트 목록 | 내 이벤트 카드, 상태 배지(open/closed), 새 이벤트 버튼 |
| `/protected/events/new` | 이벤트 생성 | 제목/설명/날짜/장소/최대인원 입력 폼 |
| `/protected/events/[id]` | 이벤트 관리 | 탭: 개요 / 참여자 / 공지 / 정산 |

**이벤트 관리 탭 상세**
- **개요 탭**: 이벤트 정보 표시, 공유 링크 복사 버튼, 이벤트 상태 변경(open/closed)
- **참여자 탭**: 참석/불참 목록, 인원 카운트
- **공지 탭**: 공지 작성 폼, 공지 목록 (최신순)
- **정산 탭**: 총액 입력, 참석 인원 수 자동 반영, 1인당 금액 표시

### 참여자 공개 화면 (비로그인)

| 경로 | 화면명 | 주요 내용 |
|---|---|---|
| `/e/[code]` | 이벤트 공개 페이지 | 이벤트 정보, 참여 폼, 참여자 목록, 공지사항 |

---

## 5. 데이터 모델

### events
| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | PK |
| host_id | uuid | auth.users FK |
| title | text | 이벤트 제목 |
| description | text | 설명 |
| location | text | 장소 |
| event_date | timestamptz | 일시 |
| max_participants | int | 최대 인원 (nullable) |
| status | text | open / closed |
| share_code | text | 공개 링크용 고유 코드 (UNIQUE) |
| created_at | timestamptz | 생성일시 |

### announcements
| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | PK |
| event_id | uuid | events FK (CASCADE) |
| content | text | 공지 내용 |
| created_at | timestamptz | 작성일시 |

### participants
| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | PK |
| event_id | uuid | events FK (CASCADE) |
| name | text | 참여자 이름 |
| status | text | attending / absent |
| created_at | timestamptz | 응답일시 |

**제약**: `UNIQUE(event_id, name)` — 동일 이벤트 내 같은 이름 중복 등록 방지
**참여 응답 처리**: INSERT 대신 UPSERT(`ON CONFLICT (event_id, name) DO UPDATE SET status = EXCLUDED.status`) 사용

### settlements
| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | PK |
| event_id | uuid | events FK (CASCADE) |
| total_amount | int | 총 비용 (원) |
| description | text | 비용 설명 (nullable) |
| created_at | timestamptz | 입력일시 |

### RLS 정책 요약
- `events` SELECT: 공개 (share_code 조회용)
- `events` INSERT/UPDATE/DELETE: host_id = auth.uid()
- `participants` INSERT: 공개 (비로그인 참여자)
- `participants` SELECT: 공개
- `announcements`, `settlements`: host만 쓰기, 공개 읽기

---

## 6. 구현 순서

### Phase 1 — DB 마이그레이션
1. events, announcements, participants, settlements 테이블 생성
   - `participants` 테이블에 `UNIQUE(event_id, name)` 복합 제약 포함
2. RLS 정책 설정
3. TypeScript 타입 자동 생성 (`supabase gen types`)
4. 미들웨어 인증 예외 경로 설정: `proxy.ts`에 `/e/` 경로 추가 (`/`, `/auth/*` 외 `/e/*`도 비인증 접근 허용)

### Phase 2 — 주최자 화면
1. `/protected/events` — 이벤트 목록 페이지
2. `/protected/events/new` — 이벤트 생성 폼 (React Hook Form + Zod)
3. `/protected/events/[id]` — 탭 기반 이벤트 관리 페이지
   - 개요: 이벤트 정보 + 공유 링크 복사
   - 참여자: 참석/불참 목록
   - 공지: 작성 폼 + 목록
   - 정산: 총액 입력 → 1/N 계산 결과

### Phase 3 — 공개 참여자 화면
1. `/e/[code]` — 이벤트 공개 페이지
   - 이벤트 정보 표시
   - 참여 응답 폼 (Server Action)
     - INSERT 대신 UPSERT 처리 (`ON CONFLICT (event_id, name) DO UPDATE`)
     - IP 기반 Rate Limiting 적용 (비로그인 참여자의 스팸 등록 방지)
   - 참여자 목록, 공지사항

### Phase 4 — 마무리
1. `npm run check-all` 통과
2. `npm run build` 확인

---

## 7. 향후 확장 계획

| 기능 | 설명 |
|---|---|
| 카풀 매칭 | 드라이버/동승자 등록 및 매칭. DB 스키마 여지 확보 (`carpool_seats` 테이블) |
| 반복 이벤트 | 정기 모임용 주간/월간 반복 생성 |
| 알림 | 이벤트 전날 카카오톡/이메일 리마인더 |
| 정산 내역 관리 | 항목별 비용 입력, 특정 인원만 분담 |
| 외부 송금 연동 | 토스/카카오페이 송금 링크 자동 생성 |

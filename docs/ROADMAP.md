# 모임 이벤트 관리 웹 MVP - 개발 로드맵

> 주최자가 이벤트를 생성하고 링크 하나로 참여자를 관리할 수 있는 소규모 모임 관리 웹 애플리케이션

**최종 수정일**: 2026-03-17

## 개요

모임 이벤트 관리 웹 MVP는 소규모 정기 모임(수영, 헬스, 친구 모임 등) 주최자를 위한 통합 관리 도구로 다음 기능을 제공합니다:

- **이벤트 생성/관리**: 제목, 설명, 날짜, 장소, 최대 인원 설정 및 공유 링크 생성
- **참여자 관리**: 비로그인 참여자의 참석/불참 응답 수집 및 목록 관리
- **공지사항**: 이벤트별 공지 작성 및 조회
- **간편 정산**: 총액 입력 후 참석 인원 기준 1/N 자동 계산

### 기술 스택

- **프론트엔드**: Next.js (App Router, Turbopack) + TypeScript
- **백엔드/DB**: Supabase (PostgreSQL + RLS + Auth)
- **UI**: shadcn/ui (new-york 스타일) + TailwindCSS v4
- **폼 처리**: React Hook Form + Zod
- **테스트**: Playwright MCP

### 사용자 구분

| 구분 | 설명 | 인증 |
|---|---|---|
| 주최자 | 이벤트를 생성하고 관리하는 사람 | 이메일 OTP 로그인 필요 |
| 참여자 | 링크로 접근해 참석 여부를 응답하는 사람 | 비로그인 (이름만 입력) |

---

## 개발 워크플로우

1. **작업 계획**
   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 구현**
   - 각 Task의 구현 사항을 순서대로 진행
   - API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수
   - 각 단계 완료 후 Task 내 체크박스 업데이트
   - 구현 완료 후 `npm run check-all` 및 `npm run build` 통과 확인

3. **로드맵 업데이트**
   - 완료된 Task를 ✅로 표시
   - Phase 내 모든 Task 완료 시 Phase도 ✅로 표시

---

## 개발 단계

### Phase 1: 타입 정의 및 공통 기반

> 화면 구현에 필요한 타입, 더미 데이터, 공통 컴포넌트, 미들웨어 설정 등 골격을 구축합니다. DB 없이 진행 가능합니다.

- ⬜ **TASK-001: TypeScript 타입 정의 및 더미 데이터 준비** `[순서 고정]` - 우선순위
  - 의존: 없음
  - PRD 데이터 모델 기반으로 TypeScript 인터페이스 수동 정의 (`@/types/` 하위)
    - `Event`, `Announcement`, `Participant`, `Settlement` 타입 정의
  - 화면 개발용 더미 데이터 파일 생성 (`@/lib/mock-data.ts`)
    - 이벤트 2~3개, 참여자 5~10명, 공지 2~3개, 정산 1건의 하드코딩 데이터
  - API 응답 타입 및 폼 입력 스키마(Zod) 정의
  - `npm run type-check` 통과 확인

- ⬜ **TASK-002: 미들웨어 설정 및 라우트 골격 생성** `[순서 고정]`
  - 의존: TASK-001
  - `proxy.ts`에 `/e/` 경로를 인증 예외 경로로 추가 (`/`, `/auth/*` 외 `/e/*`도 비인증 접근 허용)
  - 모든 주요 페이지의 빈 껍데기 파일 생성
    - `/protected/events/page.tsx`
    - `/protected/events/new/page.tsx`
    - `/protected/events/[id]/page.tsx`
    - `/e/[code]/page.tsx`
  - 공통 레이아웃 컴포넌트 골격 구현 (헤더, 네비게이션)
  - `npm run build` 통과 확인

---

### Phase 2: MVP 핵심 화면 구현 (더미 데이터)

> DB 없이 하드코딩된 더미 데이터로 모든 화면 UI를 먼저 완성합니다. 공개 참여자 화면이 MVP에서 가장 중요한 화면이므로 우선 구현합니다.

- ⬜ **TASK-003: 공개 이벤트 페이지 UI 구현** `[순서 고정]` - 우선순위
  - 의존: TASK-002
  - `/e/[code]/page.tsx` 컴포넌트 구현 (더미 데이터 사용)
  - 이벤트 정보 영역: 제목, 설명, 날짜, 장소, 현재 참석 인원 / 최대 인원
  - 참여 응답 폼 UI (이름 입력 + 참석/불참 선택 버튼) - React Hook Form + Zod
  - 현재 참여자 목록 표시 (이름 + 참석/불참 상태)
  - 공지사항 목록 표시 (최신순, 읽기 전용)
  - 이벤트 상태가 `closed`인 경우 참여 불가 안내 UI
  - 이벤트가 존재하지 않는 경우 404 UI
  - 반응형 레이아웃 적용 (모바일 우선)
  - Playwright MCP로 UI 렌더링 테스트

- ⬜ **TASK-004: 이벤트 목록 페이지 UI 구현** `[순서 유연]`
  - 의존: TASK-002
  - `/protected/events/page.tsx` 컴포넌트 구현 (더미 데이터 사용)
  - 이벤트 카드 UI 구현 (제목, 날짜, 장소, 상태 배지 open/closed)
  - "새 이벤트 만들기" 버튼 배치 (`/protected/events/new`로 이동)
  - 이벤트 없을 때 빈 상태(Empty State) UI 구현
  - Playwright MCP로 목록 페이지 렌더링 테스트

- ⬜ **TASK-005: 이벤트 생성 폼 UI 구현** `[순서 유연]`
  - 의존: TASK-002
  - `/protected/events/new/page.tsx` 컴포넌트 구현
  - React Hook Form + Zod 스키마로 폼 구현
    - 필드: 제목(필수), 설명, 날짜(필수), 장소, 최대 인원
  - 폼 유효성 검사 에러 메시지 표시
  - 제출 시 콘솔 출력 (Server Action은 DB 연동 시 구현)
  - Playwright MCP로 폼 입력, 유효성 검사 테스트

- ⬜ **TASK-006: 이벤트 관리 페이지 UI 구현 (개요/참여자/공지 탭)** `[순서 유연]`
  - 의존: TASK-002
  - `/protected/events/[id]/page.tsx` 탭 기반 레이아웃 구현 (더미 데이터 사용)
  - 탭 네비게이션 UI 구현 (개요 / 참여자 / 공지 / 정산)
  - **개요 탭**: 이벤트 정보 표시, 공유 링크 복사 버튼, 이벤트 상태 변경 토글 UI
  - **참여자 탭**: 참석/불참 목록 및 인원 카운트 표시
  - **공지 탭**: 공지 작성 폼 UI + 공지 목록(최신순) 표시
  - **정산 탭**: 총액 입력 폼 UI + 1인당 금액 계산 결과 표시 (더미 계산)
  - Playwright MCP로 탭 전환 및 각 탭 렌더링 테스트

---

### Phase 3: DB 마이그레이션 및 데이터 연동

> 데이터베이스 스키마를 구축하고, 더미 데이터를 실제 Supabase API 호출로 교체합니다.

- ⬜ **TASK-007: 데이터베이스 테이블 생성** `[순서 고정]` - 우선순위
  - 의존: 없음 (Phase 2와 병렬 진행 가능하나, 연동은 Phase 2 완료 후)
  - Supabase MCP를 사용하여 마이그레이션 파일 생성 및 적용
  - `events` 테이블 생성 (id, host_id, title, description, location, event_date, max_participants, status, share_code, created_at)
  - `announcements` 테이블 생성 (id, event_id FK CASCADE, content, created_at)
  - `participants` 테이블 생성 (id, event_id FK CASCADE, name, status, created_at)
  - `participants` 테이블에 `UNIQUE(event_id, name)` 복합 유니크 제약 추가
  - `settlements` 테이블 생성 (id, event_id FK CASCADE, total_amount, description, created_at)
  - `events.share_code`에 UNIQUE 제약 추가

- ⬜ **TASK-008: RLS 정책 설정** `[순서 고정]`
  - 의존: TASK-007
  - `events` SELECT: 공개 허용 (share_code 기반 조회)
  - `events` INSERT/UPDATE/DELETE: `host_id = auth.uid()` 조건
  - `participants` SELECT/INSERT: 공개 허용 (비로그인 참여자 지원)
  - `announcements` SELECT: 공개 허용 / INSERT/UPDATE/DELETE: host만 허용
  - `settlements` SELECT: 공개 허용 / INSERT/UPDATE/DELETE: host만 허용
  - RLS 정책 적용 후 Supabase MCP로 SQL 실행하여 정책 검증

- ⬜ **TASK-009: TypeScript 타입 자동 생성 및 클라이언트 연결** `[순서 고정]`
  - 의존: TASK-008
  - `supabase gen types`로 TypeScript 타입 자동 생성
  - 생성된 타입을 `@/lib/supabase/` 하위에 배치
  - TASK-001에서 수동 정의한 타입을 자동 생성 타입으로 교체
  - `npm run type-check` 통과 확인

- ⬜ **TASK-010: 공개 참여자 화면 DB 연동** `[순서 고정]`
  - 의존: TASK-009, TASK-003
  - `/e/[code]/page.tsx`의 더미 데이터를 Supabase 쿼리로 교체
  - `share_code`로 이벤트 정보 조회 (서버 컴포넌트)
  - 참여 응답 Server Action 구현 (UPSERT 처리: `ON CONFLICT (event_id, name) DO UPDATE SET status = EXCLUDED.status`)
  - 동일 이름 재응답 시 기존 응답 업데이트 안내 메시지 표시
  - 최대 인원 초과 시 참석 등록 제한 처리
  - 참여자 목록 및 공지사항 실시간 조회
  - Playwright MCP로 참여 응답 플로우 E2E 테스트 (신규 등록, 재응답, 최대 인원 초과 시나리오)

- ⬜ **TASK-011: 주최자 화면 DB 연동 (이벤트 CRUD)** `[순서 고정]`
  - 의존: TASK-009, TASK-004, TASK-005, TASK-006
  - 이벤트 목록 페이지: Supabase에서 `host_id = auth.uid()` 조건으로 조회
  - 이벤트 생성 폼: Server Action으로 Supabase INSERT 처리
  - `share_code` 자동 생성 (nanoid 또는 crypto.randomUUID 활용)
  - 생성 성공 시 `/protected/events/[id]`로 리다이렉트
  - 개요 탭: 이벤트 상태 변경 Server Action (open <-> closed 토글)
  - 공유 링크 복사 기능 (`/e/[share_code]` URL 클립보드 복사)
  - Playwright MCP로 이벤트 생성 및 목록 조회 E2E 테스트

- ⬜ **TASK-012: 주최자 화면 DB 연동 (참여자/공지)** `[순서 고정]`
  - 의존: TASK-011
  - 참여자 탭: event_id 기준 참여자 목록 조회 (참석/불참 구분, 인원 카운트)
  - 공지 탭: Server Action으로 공지 INSERT 처리 + 공지 목록 조회 (최신순)
  - Playwright MCP로 참여자 목록 및 공지 CRUD 테스트

---

### Phase 4: 부가 기능 및 마무리

> 정산 기능 DB 연동, 보안 강화, 통합 테스트, 코드 품질 검사를 수행합니다.

- ⬜ **TASK-013: 정산 탭 DB 연동** `[순서 유연]`
  - 의존: TASK-012
  - 총액 입력 폼: Server Action으로 Supabase UPSERT 처리
  - 참석 인원 수 자동 반영 (participants 테이블에서 attending 카운트)
  - 1인당 금액 자동 계산 및 표시 (총액 / 참석 인원)
  - 정산 내역 목록 표시
  - Playwright MCP로 정산 입력 및 계산 결과 테스트

- ⬜ **TASK-014: 보안 강화 (Rate Limiting)** `[순서 유연]`
  - 의존: TASK-010
  - 공개 참여 응답 폼에 IP 기반 Rate Limiting 적용 (비로그인 참여자의 스팸 등록 방지)
  - Rate Limiting 초과 시 사용자 안내 메시지 표시

- ⬜ **TASK-015: 전체 사용자 플로우 통합 테스트** `[순서 유연]`
  - 의존: TASK-012
  - Playwright MCP로 주최자 플로우 E2E 테스트 (로그인 -> 이벤트 생성 -> 관리)
  - Playwright MCP로 참여자 플로우 E2E 테스트 (링크 접속 -> 참여 응답 -> 목록 확인)
  - 에러 핸들링 및 엣지 케이스 테스트 (잘못된 URL, 만료된 이벤트, 네트워크 오류 등)
  - 반응형 디자인 테스트 (모바일/태블릿/데스크톱)

- ⬜ **TASK-016: 코드 품질 검사 및 빌드 확인** `[순서 고정]`
  - 의존: TASK-015
  - `npm run check-all` 통과 (ESLint + Prettier + TypeScript 타입 체크)
  - `npm run build` 프로덕션 빌드 성공 확인
  - 불필요한 콘솔 로그, TODO 주석 정리
  - 접근성(a11y) 기본 점검

---

## 향후 확장 계획 (MVP 이후)

| 기능 | 설명 |
|---|---|
| 카풀 매칭 | 드라이버/동승자 등록 및 매칭 (`carpool_seats` 테이블) |
| 반복 이벤트 | 정기 모임용 주간/월간 반복 자동 생성 |
| 알림 | 이벤트 전날 카카오톡/이메일 리마인더 |
| 정산 내역 관리 | 항목별 비용 입력, 특정 인원만 분담 |
| 외부 송금 연동 | 토스/카카오페이 송금 링크 자동 생성 |

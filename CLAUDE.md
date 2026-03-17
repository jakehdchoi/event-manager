# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 개발 명령어

```bash
# 개발 서버 실행 (Turbopack 사용)
npm run dev

# 프로덕션 빌드
npm run build

# 프로덕션 서버 실행
npm start

# 코드 검사 및 포맷팅
npm run lint           # ESLint 검사
npm run lint:fix       # ESLint 자동 수정
npm run format         # Prettier 포맷팅
npm run format:check   # Prettier 검사만
npm run type-check     # TypeScript 타입 체크
npm run check-all      # 모든 검사 통합 실행 (권장)
```

## ⚡ 자주 사용하는 명령어

```bash
# 개발
npm run dev         # 개발 서버 실행 (Turbopack)
npm run build       # 프로덕션 빌드
npm run check-all   # 모든 검사 통합 실행 (권장)

# UI 컴포넌트
npx shadcn@latest add button    # 새 컴포넌트 추가
```

## ✅ 작업 완료 체크리스트

```bash
npm run check-all   # 모든 검사 통과 확인
npm run build       # 빌드 성공 확인
```

## 환경변수 설정

`.env.local` 파일에 아래 두 변수가 필요합니다:

```env
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=...
```

## 아키텍처 개요

**Next.js App Router + Supabase** 기반의 이벤트 관리 애플리케이션입니다.

### Supabase 클라이언트 패턴

두 가지 클라이언트를 상황에 맞게 사용합니다:

- `lib/supabase/client.ts` — 클라이언트 컴포넌트용 (`createBrowserClient`)
- `lib/supabase/server.ts` — 서버 컴포넌트 / Route Handler / Server Action용 (`createServerClient`)
  - **중요**: Fluid compute 호환을 위해 전역 변수에 저장하지 말고 함수 내부에서 매번 생성할 것

### 인증 플로우

- `proxy.ts` + `lib/supabase/proxy.ts` — Next.js Proxy에서 세션 갱신 처리. 미인증 사용자를 `/auth/login`으로 리다이렉트 (`/`와 `/auth/*` 경로는 제외)
- `app/auth/confirm/route.ts` — 이메일 OTP 인증 콜백 처리
- 보호된 페이지는 `app/protected/` 아래에 위치

### UI 컴포넌트

- shadcn/ui (`new-york` 스타일, `neutral` base color, CSS variables 사용)
- 컴포넌트 추가: `npx shadcn@latest add <component>`
- `components/ui/`에 shadcn 기본 컴포넌트, `components/`에 앱 전용 컴포넌트

### 경로 별칭

`@/` → 프로젝트 루트 (`components`, `lib`, `hooks` 등)

## MCP 서버

이 프로젝트에서 활용 가능한 MCP 서버 목록:

- **supabase** — DB 테이블 조회, SQL 실행, 마이그레이션, 브랜치 관리 등 Supabase 작업
- **playwright** — 브라우저 자동화, UI 테스트, 스크린샷 캡처
- **context7** — 라이브러리 최신 공식 문서 및 코드 예제 조회
- **sequential-thinking** — 복잡한 문제를 단계별로 분해해 사고
- **shadcn** — shadcn/ui 컴포넌트 검색, 예제 조회, 추가 명령어 확인
- **shrimp-task-manager** — 작업 계획, 분할, 추적, 검증 관리

## 개발 가이드 문서

`docs/guides/` 아래 개발 규칙 가이드가 있습니다. 관련 작업 전에 참조하세요:

- `docs/guides/nextjs.md` — App Router, Server Components, Turbopack, 캐싱 패턴 등 Next.js 규칙
- `docs/guides/component-patterns.md` — 컴포넌트 설계 패턴, SRP, CVA, 안티패턴
- `docs/guides/forms-react-hook-form.md` — React Hook Form + Zod + Server Actions 폼 처리
- `docs/guides/styling-guide.md` — TailwindCSS v4 + shadcn/ui 스타일링 규칙
- `docs/guides/project-structure.md` — 폴더 구조, 네이밍 컨벤션, 경로 별칭

# docs/SECURITY.md — 보안 기준

## 개요
보안 항목은 기능 구현보다 우선순위가 높다.
작업 시작 전 반드시 이 문서를 숙지할 것.

## 인증/인가

### JWT
- Secret Key는 반드시 환경변수로 관리 (`.env`)
- Secret Key 길이: 256bit 이상
- Access Token 유효기간: 30분
- Refresh Token: HttpOnly Cookie로만 전달, 유효기간 7일
- 토큰 검증 실패 시 즉시 401 반환

### Spring Security
- 모든 `/api/v1/admin/**` 엔드포인트는 `ROLE_ADMIN` 필수
- 인증 필요 API는 `SecurityConfig`에 명시적으로 선언
- CORS 허용 Origin은 환경변수로 관리 (하드코딩 금지)

## 입력값 검증

### 백엔드
- 모든 입력값은 `@Valid` + DTO 레벨에서 검증
- HTML 입력(게시글 본문)은 서버사이드 sanitize 필수
  - 라이브러리: `com.googlecode.owasp-java-html-sanitizer:owasp-java-html-sanitizer`
  - 허용 태그: `p`, `br`, `strong`, `em`, `u`, `a`(href만), `img`(src만), `ul`, `ol`, `li`, `blockquote`, `h1`~`h4`, `code`, `pre`
  - `script`, `iframe`, `on*` 이벤트 속성 전면 차단
- 파일 업로드 시 MIME 타입 검증: Apache Tika(`org.apache.tika:tika-core`)로 magic bytes를 읽어 실제 MIME 타입을 확인한다. Content-Type 헤더와 확장자만으로 신뢰하지 않는다.
- SQL은 JPA 파라미터 바인딩만 사용 (네이티브 쿼리 직접 작성 금지)

### 프론트엔드
- 게시글 본문 렌더링 시 DOMPurify로 XSS 방지
- `dangerouslySetInnerHTML` 사용 시 반드시 sanitize 후 사용

## 민감 정보 보호

### 서버
- 비밀번호는 BCrypt 해싱 (rounds: 10 이상)
- 응답에 비밀번호, 토큰 등 민감 정보 절대 포함 금지
- 에러 응답에 스택 트레이스 노출 금지 (프로덕션)
- 로그에 비밀번호, 개인정보 기록 금지

### 환경변수
- DB 비밀번호, JWT Secret, 이메일 계정 등은 `.env` 파일로 관리
- `.env` 파일은 절대 Git에 커밋하지 말 것
- `.env.example` 파일에 키 목록만 제공 (값은 비워둠)

## 요청 제한 (Rate Limiting)

구현 방법: **Bucket4j** 라이브러리 + Spring MVC 인터셉터.
Redis 없이 로컬 인메모리로 동작 가능 (로컬 개발 환경 기준).

| 엔드포인트 | 제한 | 기준 |
|-----------|------|------|
| `POST /auth/login` | 분당 10회 | IP |
| `POST /auth/signup` | 시간당 5회 | IP |
| `POST /auth/resend-verification` | 5분에 1회 | 계정 |
| `POST /reports` | 하루 20회 | 계정 |
| `POST /attachments/*` | 분당 10회 | 계정 |

## 관리자 페이지 보안
- URL: `/admin/*` → 일반 사용자 접근 시 **404** 반환 (403 아님 — 존재 여부 노출 금지)
- 관리자 계정은 회원가입으로 취득 불가 (Flyway seed로만 생성)
- 모든 관리자 액션은 로그 기록 (누가, 언제, 무엇을)

## 파일 업로드 보안
- 실행 파일 (`.exe`, `.sh`, `.bat`, `.php` 등) 업로드 전면 차단
- 파일명은 UUID로 저장 (원본 파일명 노출 금지)
- 업로드 디렉토리는 웹 루트 외부에 위치
- 이미지 파일은 재인코딩 후 저장: `net.coobird:thumbnailator`로 원본을 읽어 새 이미지로 재출력한다. 이미지에 삽입된 악성 페이로드를 제거하는 효과가 있다.

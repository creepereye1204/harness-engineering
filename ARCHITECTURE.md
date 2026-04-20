# ARCHITECTURE.md

## 시스템 구성도

```
[React SPA]
    ↕ HTTP/REST (JSON)
[Spring Boot API Server]
    ↕ JPA/Hibernate
[MySQL 8.x]

모든 환경은 Docker Compose로 로컬 실행
Nginx가 앞단에서 정적 파일 서빙 + API 프록시
```

## 폴더 구조

```
/
├── frontend/                 # React (Vite)
│   ├── src/
│   │   ├── components/       # 공통 UI 컴포넌트
│   │   ├── pages/            # 라우팅 단위 페이지
│   │   ├── hooks/            # 커스텀 훅
│   │   ├── api/              # API 호출 함수 모음
│   │   ├── store/            # 전역 상태 (Zustand)
│   │   └── utils/            # 공통 유틸
│   └── ...
│
├── backend/
│   └── src/main/java/
│       ├── domain/
│       │   ├── user/
│       │   │   ├── controller/
│       │   │   ├── service/
│       │   │   ├── repository/
│       │   │   └── entity/
│       │   ├── post/
│       │   ├── comment/
│       │   ├── board/
│       │   ├── vote/
│       │   ├── attachment/
│       │   ├── notification/
│       │   ├── report/
│       │   └── admin/
│       ├── global/
│       │   ├── config/       # Security, CORS 등
│       │   ├── exception/    # 전역 예외 처리
│       │   ├── jwt/          # JWT 유틸
│       │   └── settings/     # ServiceSettingsService
│       └── ...
│
├── docker/
│   └── docker-compose.yml
└── docs/
```

## 인증 방식 (JWT)
- 로그인 성공 시 Access Token + Refresh Token 발급
- Access Token: 유효기간 30분, Authorization 헤더로 전달
- Refresh Token: 유효기간 7일, HttpOnly Cookie로 저장
- 모든 인증 필요 API는 Spring Security Filter에서 토큰 검증

## API 설계 원칙

### URL 규칙 (★ 중요)
- 게시판 식별자: slug 사용 → /api/v1/boards/{slug}
- 그 외 모든 리소스: id 사용 → /api/v1/posts/{postId}
- Base URL: /api/v1/

### 페이지네이션 파라미터 통일
- 페이지 번호 방식 (게시글 목록):
  ?page=0&size=20&sort=createdAt,desc
- 더보기 방식 (댓글, 알림):
  ?lastId={마지막_id}&size=20

### 응답 포맷 통일
```json
{
  "success": true,
  "data": { ... },
  "message": "ok"
}
```

### 에러 응답 포맷 통일
```json
{
  "success": false,
  "code": "USER_NOT_FOUND",
  "message": "존재하지 않는 사용자입니다"
}
```

### 에러 코드 네이밍 규칙
- 형식: {DOMAIN}_{ERROR_TYPE}
- 예: USER_NOT_FOUND, POST_FORBIDDEN, COMMENT_DUPLICATE_VOTE
- 모든 예외는 GlobalExceptionHandler에서 위 형식으로 통일

### HTTP 메서드 원칙
- GET: 조회
- POST: 생성
- PUT: 전체 수정
- PATCH: 부분 수정
- DELETE: 삭제

## 주요 도메인
- User: 회원가입, 로그인, 프로필
- Post: 게시글 CRUD, 목록/검색
- Comment: 댓글 CRUD (parent_id로 대댓글 관리)
- Board: 게시판 카테고리 관리
- Vote: 추천/비추천 (게시글), 추천만 (댓글)
- Attachment: 파일/이미지 첨부
- Notification: 알림 (SSE)
- Report: 신고

## DB 원칙
- 모든 Entity는 createdAt, updatedAt 공통 필드 포함
- soft delete 방식 사용 (deletedAt 컬럼, 실제 삭제 X)
- 모든 목록 조회는 deletedAt IS NULL 기본 필터 적용
- N+1 문제 방지: fetch join 또는 @EntityGraph 사용
- 스키마 변경은 반드시 Flyway 마이그레이션으로 관리
- 카운터 컬럼(upvote_count 등) 증감은 벌크 업데이트 쿼리 사용

## 게시판 권한 체크 원칙 (★ 중요)
게시판 운영자 여부는 아래 두 조건 중 하나를 만족하면 인정:
1. boards.owner_id = 현재 사용자 id
2. board_moderators 테이블에 (board_id, user_id) 레코드 존재

단, 게시판 개설 시 owner는 board_moderators에도 자동 삽입하여
권한 체크를 board_moderators 단일 테이블로 통일할 수 있도록 한다.

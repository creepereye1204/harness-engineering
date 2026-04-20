# CLAUDE.md

Reddit/DC인사이드 스타일 커뮤니티 게시판. React 18 + Spring Boot 3 + MySQL 8.

코딩 원칙과 워크플로우는 `CONSTITUTION.md`를 따른다.
보안 요구사항은 기능 구현보다 우선순위가 높다 — 작업 전 `docs/SECURITY.md` 숙지 필수.

## 작업 전 필독 가이드

| 작업 유형 | 읽어야 할 문서 |
|---|---|
| 기능 구현 전 스펙 확인 | `docs/product-specs/{도메인}.md` |
| API 설계 / 응답 포맷 / URL 규칙 | `ARCHITECTURE.md` |
| DB 스키마 확인 | `docs/generated/db-schema.md` |
| 디자인 / 컴포넌트 / 색상 | `docs/design-docs/index.md` |
| 프론트엔드 개발 규칙 | `docs/FRONTEND.md` |
| 보안 요구사항 | `docs/SECURITY.md` |
| 안정성 기준 | `docs/RELIABILITY.md` |
| 작업 완료 체크리스트 | `docs/QUALITY_SCORE.md` |
| 현재 진행할 백엔드 작업 | `docs/exec-plans/active/phase-1-backend.md` |
| 현재 진행할 프론트엔드 작업 | `docs/exec-plans/active/phase-2-frontend.md` |

새 기능 구현 전 반드시 해당 product-spec을 먼저 확인하라.

## 이 프로젝트의 불변 원칙

아래는 코딩 스타일이 아니라 이 프로젝트의 데이터 모델과 비즈니스 규칙에서 오는 제약이다.
어기면 데이터 정합성이 깨진다.

**Soft Delete** — 모든 목록 조회는 `deleted_at IS NULL` 필터 필수. 예외가 필요한 경우 CONSTITUTION.md 주석 규칙에 따라 한 줄 이유를 기술한다.

**카운터 동기화** — `upvote_count`, `downvote_count`, `comment_count`, `subscriber_count` 변경은 항상 JPA 벌크 업데이트 쿼리로. 엔티티 로드 후 증감 금지.
```sql
UPDATE posts SET upvote_count = upvote_count + 1 WHERE id = :id
```

**운영 정책값 하드코딩 금지** — 블라인드 기준, 정지 기준 등은 `ServiceSettingsService.getValue(key)` 로만.

**이메일 중복 체크** — `deleted_at` 무관 전체 레코드 대상. 탈퇴 계정 이메일 영구 재사용 불가.

**게시판 권한** — `board_moderators` 테이블 단일 기준. 설계 근거는 `ARCHITECTURE.md` 참조.

**Flyway** — DB 스키마 변경은 반드시 마이그레이션 파일로. 자동 실행 금지, 실행 전 사용자 확인 필수.

**N+1 방지** — fetch join 또는 `@EntityGraph` 사용. 목록 API 작성 후 쿼리 로그 확인 필수.
쿼리 로그 확인: `application-local.yml`에 `logging.level.org.hibernate.SQL: DEBUG` 설정. 목록 조회 API 호출 시 SQL이 1개만 출력되는지 확인한다.

## API 규칙

- Base URL: `/api/v1/`
- 게시판 식별자: slug (`/boards/{slug}`), 나머지: id
- 페이지네이션: 목록 `?page=0&size=20&sort=createdAt,desc` / 댓글·알림 `?lastId=&size=20`
- 성공 응답: `{ "success": true, "data": {...}, "message": "ok" }`
- 에러 응답: `{ "success": false, "code": "DOMAIN_ERROR_TYPE", "message": "..." }`
- 모든 예외는 `GlobalExceptionHandler`에서 위 형식으로 통일 처리

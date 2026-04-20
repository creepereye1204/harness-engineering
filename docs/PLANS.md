# docs/PLANS.md — 전체 계획 요약

## 프로젝트 개요
React + Spring Boot 기반 일반 커뮤니티 게시판.
AI 에이전트(Claude Code) 기반 하네스 엔지니어링으로 개발.

## 개발 전략
백엔드 API 전체 완성 → 프론트엔드 구현 순서.

## 전체 로드맵

### 백엔드 (phase-1-backend.md)
| Phase | 내용 | 상태 |
|-------|------|------|
| 1 | 프로젝트 기반 세팅 | ⬜ 대기 |
| 2 | 인증/회원 API | ⬜ 대기 |
| 3 | 게시판 API | ⬜ 대기 |
| 4 | 게시글 API | ⬜ 대기 |
| 5 | 댓글 + 추천 API | ⬜ 대기 |
| 6 | 파일 첨부 API | ⬜ 대기 |
| 7 | 알림 + 신고 API | ⬜ 대기 |
| 8 | 관리자 API | ⬜ 대기 |

### 프론트엔드 (phase-2-frontend.md)
| Phase | 내용 | 선행 조건 | 상태 |
|-------|------|-----------|------|
| 1 | 기반 세팅 | 백엔드 Phase 1 | ⬜ 대기 |
| 2 | 인증 화면 | 백엔드 Phase 2 | ⬜ 대기 |
| 3 | 게시판 + 게시글 화면 | 백엔드 Phase 3, 4 | ⬜ 대기 |
| 4 | 댓글 + 추천 화면 | 백엔드 Phase 5 | ⬜ 대기 |
| 5 | 알림 + 신고 화면 | 백엔드 Phase 7 | ⬜ 대기 |
| 6 | 관리자 페이지 | 백엔드 Phase 8 | ⬜ 대기 |

## 문서 맵

| 궁금한 것 | 참조 문서 |
|-----------|-----------|
| 작업 규칙·진입점 | `CLAUDE.md` |
| 코딩 원칙·워크플로우 | `CONSTITUTION.md` |
| 시스템 구조·API 설계 | `ARCHITECTURE.md` |
| 기능 명세 | `docs/product-specs/` |
| UI/UX 규칙 | `docs/design-docs/` |
| 프론트엔드 개발 규칙 | `docs/FRONTEND.md` |
| 작업 순서 | `docs/exec-plans/active/` |
| DB 구조 | `docs/generated/db-schema.md` |
| 보안 요구사항 | `docs/SECURITY.md` |
| 안정성 기준 | `docs/RELIABILITY.md` |
| 완료 체크리스트 | `docs/QUALITY_SCORE.md` |
| 기술 부채 | `docs/exec-plans/tech-debt-tracker.md` |

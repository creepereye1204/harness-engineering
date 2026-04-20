# docs/exec-plans/active/phase-1-backend.md — 백엔드 API 구축

## 목표
프론트엔드 개발 전 모든 백엔드 API를 완성한다.
각 Phase 완료 기준: API 엔드포인트 동작 확인 + 기본 테스트 통과.

## Phase 의존성
```
Phase 1 (기반) → Phase 2 (인증) → Phase 3 (게시판) → Phase 4 (게시글)
                                                     → Phase 5 (댓글·추천)
                                                     → Phase 6 (파일 첨부)
Phase 4 + Phase 5 → Phase 7 (알림·신고)
Phase 1~7 완료 → Phase 8 (관리자)
```
이전 Phase 완료 기준을 통과하지 않은 상태에서 다음 Phase를 시작하지 않는다.

## Phase 1 — 프로젝트 기반 세팅
**목표**: 개발 환경 구축 및 공통 구조 완성

### 작업 목록
- [ ] Docker Compose 세팅 (Spring Boot + MySQL + Nginx)
- [ ] Spring Boot 프로젝트 초기화
  - [ ] 의존성 설정 (Spring Security, JPA, JWT, Flyway 등)
  - [ ] 도메인형 패키지 구조 생성
  - [ ] application.yml 환경별 분리 (local / prod)
- [ ] Flyway 마이그레이션 설정
- [ ] 공통 응답 포맷 구현 (ApiResponse)
- [ ] 전역 예외 처리기 구현 (GlobalExceptionHandler)
- [ ] JWT 유틸 구현 (발급 / 검증 / 재발급)
- [ ] Spring Security 기본 설정
- [ ] ServiceSettingsService 구현 (service_settings 테이블 조회)

### 완료 기준
- Docker Compose up 시 전체 서비스 정상 실행
- /api/v1/health 엔드포인트 200 응답

---

## Phase 2 — 인증/회원 API
**목표**: user.md 기반 전체 구현

### 작업 목록
- [ ] users, email_verifications, refresh_tokens 테이블 Flyway 마이그레이션
- [ ] 회원가입 API
  - [ ] 이메일 중복 체크 (deleted_at 무관 전체 레코드 대상)
  - [ ] 닉네임 중복 체크 (deleted_at IS NULL 활성 계정만)
  - [ ] 이메일 인증 토큰 발급
- [ ] 이메일 발송 설정 (JavaMailSender, 트랜잭션 밖에서 비동기 처리)
- [ ] 이메일 인증 확인 API
- [ ] 인증 이메일 재발송 API
- [ ] 로그인 API (JWT 발급, 정지 계정 차단)
- [ ] 로그아웃 API
- [ ] 토큰 재발급 API
- [ ] 내 정보 조회/수정 API
- [ ] 비밀번호 변경 API
- [ ] 회원 탈퇴 API (soft delete, 즉시 로그아웃)
- [ ] 타인 프로필 조회 API

### 완료 기준
- 회원가입 → 이메일 인증 → 로그인 → 토큰 재발급 플로우 정상 동작
- 탈퇴 계정 이메일로 재가입 시 409 반환 확인

---

## Phase 3 — 게시판 API
**목표**: board.md 기반 전체 구현

### 작업 목록
- [ ] boards, board_moderators, board_subscriptions 테이블 Flyway 마이그레이션
- [ ] 게시판 개설 API
  - [ ] 개설자를 board_moderators에 자동 삽입
- [ ] 게시판 목록/상세 조회 API
- [ ] 게시판 설정 수정 API (운영자: board_moderators 기준 체크)
- [ ] 게시판 폐쇄 API (is_system=true이면 차단)
- [ ] 구독/구독취소 API (subscriber_count 벌크 업데이트)
- [ ] 운영자 추가/제거 API
- [ ] 시스템 기본 게시판 초기 데이터 (Flyway seed)

### 완료 기준
- 게시판 개설 → 구독 → 운영자 추가 플로우 정상 동작
- board_moderators 단일 기준 권한 체크 확인

---

## Phase 4 — 게시글 API
**목표**: post.md 기반 전체 구현

### 작업 목록
- [ ] posts, post_drafts, post_view_logs 테이블 Flyway 마이그레이션
- [ ] 게시글 작성 API (board_id 필수)
- [ ] 게시글 상세 조회 API
  - [ ] 조회수 중복 방지 (post_view_logs DB 기반)
- [ ] 게시글 목록 API (최신순/인기순, ?page=0&size=20&sort=createdAt,desc)
- [ ] 게시글 수정 API
- [ ] 게시글 삭제 API (soft delete + 연관 첨부 파일 soft delete 동시 처리)
- [ ] 인기글 목록 API (점수 계산: 추천×0.7 + 조회×0.3)
- [ ] 인기글 점수 배치 갱신 (10분 간격, 조회수 반영)
- [ ] 게시글 검색 API (LIKE 쿼리, 추후 Full-text로 전환)
- [ ] 임시저장 CRUD API
- [ ] 고정글 처리 API

### 완료 기준
- 게시글 작성 → 조회 → 수정 → 삭제 플로우 정상 동작
- 게시글 삭제 시 attachments.deleted_at 동시 처리 확인
- 인기글 점수 계산 로직 단위 테스트 통과

---

## Phase 5 — 댓글 + 추천 API
**목표**: comment.md + vote.md 기반 전체 구현

### 작업 목록
- [ ] comments, votes 테이블 Flyway 마이그레이션
- [ ] 댓글 작성 API (posts.comment_count +1 벌크 업데이트)
- [ ] 댓글 수정 API
- [ ] 댓글 삭제 API (soft delete, posts.comment_count -1)
- [ ] 대댓글 작성 API
  - [ ] 부모 댓글 deleted_at IS NULL 체크
  - [ ] 부모 댓글의 parent_id IS NULL 체크 (대댓글의 대댓글 방지)
- [ ] 대댓글 수정/삭제 API
- [ ] 댓글 목록 조회 API (최신순/추천순, ?lastId=&size=20)
- [ ] 게시글 추천/비추천 API (upvote_count, downvote_count 벌크 업데이트)
- [ ] 게시글 투표 취소 API
- [ ] 댓글 추천 API (upvote_count 벌크 업데이트)
- [ ] 댓글 추천 취소 API
- [ ] 댓글 비추천 시도 시 VOTE_DOWN_NOT_ALLOWED 에러 반환
- [ ] 비추천 누적 블라인드 처리 로직 (service_settings 참조)
- [ ] 추천 시 popularity_score 즉시 재계산

### 완료 기준
- 댓글 → 대댓글 → 추천 → 취소 플로우 정상 동작
- 댓글 비추천 시도 시 400 에러 반환 확인
- 본인 글 추천 불가 예외 처리 확인

---

## Phase 6 — 파일 첨부 API
**목표**: attachment.md 기반 전체 구현

### 작업 목록
- [ ] attachments 테이블 Flyway 마이그레이션
- [ ] Docker 볼륨 마운트 설정
- [ ] 이미지 임시 업로드 API
- [ ] 이미지 리사이징 처리 (thumbnails + resized)
- [ ] 파일 임시 업로드 API
- [ ] 임시 파일 → 정식 파일 전환 로직 (게시글 발행 시)
- [ ] 파일 다운로드 API
- [ ] 첨부 파일 삭제 API
- [ ] 임시 파일 24시간 자동 삭제 배치
- [ ] Nginx 정적 파일 서빙 설정

### 완료 기준
- 이미지 업로드 → 리사이징 → URL 반환 정상 동작
- 파일 다운로드 정상 동작
- 24시간 후 임시 파일 자동 삭제 확인

---

## Phase 7 — 알림 + 신고 API
**목표**: notification.md + report.md 기반 전체 구현

### 작업 목록
- [ ] notifications, notification_settings, reports 테이블 Flyway 마이그레이션
- [ ] SSE 연결 API (하트비트 15초)
- [ ] 알림 목록 조회 API (?lastId=&size=20)
- [ ] 알림 읽음 처리 API
- [ ] 전체 읽음 처리 API
- [ ] 알림 설정 조회/수정 API
- [ ] 알림 자동 생성 로직 (댓글/추천/신고결과 발생 시)
- [ ] 추천 알림 묶음 처리 로직 (1시간 내 동일 대상)
- [ ] 신고 접수 API
- [ ] 신고 자동 블라인드 처리 (service_settings.auto_blind_threshold 참조)
- [ ] 경고 누적 자동 정지 처리 (service_settings 참조, 즉시 로그아웃)
- [ ] 오래된 알림 자동 삭제 배치 (created_at 기준, 읽음 30일, 미읽음 90일)

### 완료 기준
- SSE 연결 → 댓글 작성 → 실시간 알림 수신 정상 동작
- 신고 5회 누적 시 자동 블라인드 정상 동작
- 경고 3회 누적 시 자동 정지 + 즉시 로그아웃 확인

---

## Phase 8 — 관리자 API
**목표**: admin.md 기반 전체 구현

### 작업 목록
- [ ] 관리자 계정 초기 생성 (Flyway seed로 관리)
  - seed 파일에서 BCrypt 해싱된 비밀번호로 삽입
  - .env에서 초기 관리자 이메일/비밀번호 환경변수로 주입
  - 프로덕션 배포 후 즉시 비밀번호 변경 필수
- [ ] service_settings 초기 데이터 seed
- [ ] 대시보드 통계 API
- [ ] 회원 목록/상세 조회 API
- [ ] 회원 제재 API (경고/정지/영구정지/해제)
  - [ ] 경고 시 warning_count +1 및 자동 정지 체크 (service_settings 참조)
  - [ ] 정지/영구정지 시 즉시 로그아웃 처리
- [ ] 관리자 권한 부여/회수 API (본인 제외)
- [ ] 게시판 상태 변경 API (is_system=true 보호)
- [ ] 게시글/댓글 강제 삭제 API
- [ ] 블라인드 처리/해제 API
- [ ] 신고 목록/상세/처리 API
- [ ] 서비스 설정 조회/수정 API

### 완료 기준
- 관리자 로그인 → 신고 처리 → 회원 제재 플로우 정상 동작
- 일반 사용자 /admin/* 접근 시 404 반환 확인
- 본인 계정 영구정지/권한 회수 시도 시 403 반환 확인

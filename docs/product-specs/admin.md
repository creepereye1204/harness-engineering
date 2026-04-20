# admin.md — 관리자 페이지

## 기능 설명
서비스 전반을 관리하는 관리자 전용 페이지.
회원 관리, 게시판 관리, 신고 처리, 통계 대시보드 포함.
관리자 계정은 Flyway seed로 초기 생성 (회원가입으로 취득 불가).

## 접근 제어
- URL: /admin/*
- Spring Security에서 ROLE_ADMIN 권한 체크
- 일반 사용자 접근 시 404 반환 (관리자 페이지 존재 자체를 노출 X)

## 주요 기능

### 1. 대시보드
- 오늘 가입자 수 / 누적 가입자 수
- 오늘 게시글 수 / 누적 게시글 수
- 오늘 댓글 수 / 누적 댓글 수
- 처리 대기 중인 신고 수
- 최근 7일 가입자/게시글 추이 차트

### 2. 회원 관리
- 전체 회원 목록 조회 (이메일, 닉네임, 가입일, 상태)
- 회원 검색 (이메일, 닉네임)
- 회원 상세 조회:
  - 기본 정보
  - 작성 글/댓글 목록
  - 신고 내역
  - 제재 내역
- 회원 제재 처리:
  - 경고 부여 (warning_count +1, 자동 정지 체크)
  - 일시정지 (기간 설정)
  - 영구정지
  - 제재 해제
- 관리자 권한 부여/회수

### 3. 게시판 관리
- 전체 게시판 목록 조회
- 게시판 상태 변경 (ACTIVE / INACTIVE / BANNED)
- 게시판 강제 폐쇄 (BANNED 처리)
- 시스템 기본 게시판 관리:
  - 공지사항, 자유게시판 등 기본 게시판 생성
  - 기본 게시판은 삭제/폐쇄 불가 (is_system=true)

### 4. 게시글/댓글 관리
- 전체 게시글 목록 조회 (게시판, 작성자, 날짜 필터)
- 블라인드 처리된 게시글 목록
- 게시글 강제 삭제
- 댓글 강제 삭제
- 블라인드 해제

### 5. 신고 관리
- 신고 목록 조회 (처리 대기 / 처리 완료 탭)
- 신고 상세 조회 (신고 대상 내용 + 신고 사유)
- 신고 처리:
  - 콘텐츠 삭제
  - 작성자 경고/정지
  - 신고 기각
- 처리 시 신고자에게 자동 알림 발송

### 6. 서비스 설정
- 자동 블라인드 신고 누적 기준 수 설정 (auto_blind_threshold, 기본값: 5)
- 경고 누적 자동 정지 기준 수 설정 (auto_suspend_warning_count, 기본값: 3)
- 자동 정지 기간 설정 (auto_suspend_days, 기본값: 7일)
- 회원가입 허용 여부 (signup_enabled)
- 공지 배너 ON/OFF 및 내용 설정

## API 목록

| 메서드 | 엔드포인트 | 설명 |
|--------|-----------|------|
| GET | /api/v1/admin/dashboard | 대시보드 통계 |
| GET | /api/v1/admin/users | 회원 목록 조회 |
| GET | /api/v1/admin/users/{userId} | 회원 상세 조회 |
| POST | /api/v1/admin/users/{userId}/sanction | 회원 제재 |
| DELETE | /api/v1/admin/users/{userId}/sanction | 제재 해제 |
| PATCH | /api/v1/admin/users/{userId}/role | 관리자 권한 부여/회수 |
| GET | /api/v1/admin/boards | 게시판 목록 조회 |
| PATCH | /api/v1/admin/boards/{slug}/status | 게시판 상태 변경 |
| GET | /api/v1/admin/posts | 게시글 목록 조회 |
| DELETE | /api/v1/admin/posts/{postId} | 게시글 강제 삭제 |
| PATCH | /api/v1/admin/posts/{postId}/blind | 블라인드 처리/해제 |
| DELETE | /api/v1/admin/comments/{commentId} | 댓글 강제 삭제 |
| GET | /api/v1/admin/reports | 신고 목록 조회 |
| GET | /api/v1/admin/reports/{reportId} | 신고 상세 조회 |
| PATCH | /api/v1/admin/reports/{reportId}/process | 신고 처리 |
| GET | /api/v1/admin/settings | 서비스 설정 조회 |
| PATCH | /api/v1/admin/settings | 서비스 설정 수정 |

## 예외 처리

| 상황 | HTTP 상태 | 에러 코드 | 메시지 |
|------|-----------|-----------|--------|
| 일반 사용자 접근 | 404 | - | (노출 없음) |
| 본인 계정 영구정지 시도 | 403 | ADMIN_SELF_BAN_FORBIDDEN | "본인 계정은 정지할 수 없습니다" |
| 본인 권한 회수 시도 | 403 | ADMIN_SELF_ROLE_FORBIDDEN | "본인 권한은 회수할 수 없습니다" |
| 시스템 게시판 폐쇄 시도 | 403 | BOARD_SYSTEM_PROTECTED | "시스템 기본 게시판은 폐쇄할 수 없습니다" |

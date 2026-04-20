# board.md — 게시판 카테고리 분류

## 기능 설명
누구나 게시판을 개설하고 운영할 수 있는 서브레딧 방식의 게시판 시스템.
단일 depth 구조로, 각 게시판은 고유한 슬러그(URL)를 가짐.

## 사용자 시나리오

### 게시판 개설
1. 로그인한 사용자가 게시판 이름, 슬러그, 설명 입력
2. 슬러그 중복 확인
3. 개설 완료
4. 개설자(owner)는 board_moderators 테이블에도 자동 삽입
   → 이후 모든 운영자 권한 체크는 board_moderators 단일 테이블 기준
5. 게시판 메인 페이지로 이동

### 게시판 목록 조회
1. 전체 게시판 목록 조회 (구독자 수 순 정렬 기본)
2. 게시판 검색 가능 (이름/설명 기준)
3. 인기 게시판 상단 노출

### 게시판 구독
1. 로그인 사용자는 게시판 구독/구독취소 가능
2. 구독/구독취소 시 boards.subscriber_count 즉시 업데이트
   (동시성 방지: UPDATE boards SET subscriber_count = subscriber_count ± 1)
3. 구독한 게시판 글은 메인 피드에 노출
4. 비로그인 사용자는 구독 없이 모든 게시판 열람 가능

### 게시판 설정 (운영자)
1. 게시판 설명, 배너 이미지, 아이콘 수정 가능
2. 공지사항 등록 가능
3. 운영자 추가/제거 가능 (owner도 board_moderators 기준으로 관리)
4. 게시판 폐쇄 가능 (soft delete)

## 상세 요구사항

### 게시판 개설
- 이름: 2~50자
- 슬러그: 영문+숫자+하이픈만 허용, 3~30자, 중복 불가
  - 예: /b/free, /b/humor, /b/programming
- 설명: 최대 500자
- 1인당 게시판 개설 최대 5개 (어뷰징 방지)

### 게시판 역할
운영자 여부는 board_moderators 테이블 단일 기준으로 체크.
개설자(owner)는 게시판 생성 시 board_moderators에 자동 삽입.

- **일반 회원**: 글 읽기, 구독
- **운영자(moderator)**: 게시판 설정 수정, 공지 등록, 게시글 고정/삭제
- **관리자(admin)**: 모든 게시판에 대한 전체 권한

### 게시판 상태
- ACTIVE: 정상 운영 중
- INACTIVE: 운영자에 의해 비공개 처리
- BANNED: 관리자에 의해 강제 폐쇄

### 메인 피드
- 비로그인: 전체 게시판의 인기글 노출
- 로그인: 구독한 게시판의 최신글 노출

## API 목록

| 메서드 | 엔드포인트 | 설명 | 인증 |
|--------|-----------|------|------|
| GET | /api/v1/boards | 게시판 목록 조회 | 불필요 |
| POST | /api/v1/boards | 게시판 개설 | 필요 |
| GET | /api/v1/boards/{slug} | 게시판 상세 조회 | 불필요 |
| PATCH | /api/v1/boards/{slug} | 게시판 설정 수정 | 운영자 |
| DELETE | /api/v1/boards/{slug} | 게시판 폐쇄 | 운영자 |
| POST | /api/v1/boards/{slug}/subscribe | 구독 | 필요 |
| DELETE | /api/v1/boards/{slug}/subscribe | 구독 취소 | 필요 |
| GET | /api/v1/boards/{slug}/moderators | 운영자 목록 | 불필요 |
| POST | /api/v1/boards/{slug}/moderators | 운영자 추가 | 운영자 |
| DELETE | /api/v1/boards/{slug}/moderators/{userId} | 운영자 제거 | 운영자 |

## 예외 처리

| 상황 | HTTP 상태 | 에러 코드 | 메시지 |
|------|-----------|-----------|--------|
| 슬러그 중복 | 409 | BOARD_SLUG_DUPLICATE | "이미 사용중인 게시판 주소입니다" |
| 개설 한도 초과 | 403 | BOARD_CREATE_LIMIT | "게시판은 최대 5개까지 개설할 수 있습니다" |
| 권한 없음 | 403 | BOARD_FORBIDDEN | "게시판 운영자만 접근할 수 있습니다" |
| 존재하지 않는 게시판 | 404 | BOARD_NOT_FOUND | "존재하지 않는 게시판입니다" |
| BANNED 게시판 접근 | 403 | BOARD_BANNED | "관리자에 의해 폐쇄된 게시판입니다" |
| 시스템 게시판 폐쇄 시도 | 403 | BOARD_SYSTEM_PROTECTED | "시스템 기본 게시판은 폐쇄할 수 없습니다" |

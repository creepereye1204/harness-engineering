# docs/generated/db-schema.md — DB 스키마

> 이 파일은 자동 생성이 아닌 수동 관리 문서다. 스키마 변경 시 Flyway 마이그레이션 파일과 함께 이 파일도 반드시 업데이트한다.

## 공통 원칙
- 모든 테이블은 `created_at`, `updated_at` 포함
- soft delete 테이블은 `deleted_at` 포함
- 모든 목록 조회는 `deleted_at IS NULL` 기본 필터 적용
- 카운터 증감은 벌크 업데이트 쿼리 사용

---

## users
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| email | VARCHAR(255) | UNIQUE, NOT NULL | 탈퇴 계정 포함 전체 UNIQUE |
| password | VARCHAR(255) | NOT NULL | BCrypt 해싱 |
| nickname | VARCHAR(20) | NOT NULL | 활성 계정만 UNIQUE 체크 (앱 레벨) |
| profile_image_url | VARCHAR(500) | NULL | |
| role | ENUM('USER','ADMIN') | NOT NULL DEFAULT 'USER' | |
| status | ENUM('ACTIVE','SUSPENDED','BANNED') | NOT NULL DEFAULT 'ACTIVE' | |
| is_email_verified | BOOLEAN | NOT NULL DEFAULT FALSE | |
| suspended_until | DATETIME | NULL | 정지 해제일 |
| warning_count | INT | NOT NULL DEFAULT 0 | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| deleted_at | DATETIME | NULL | 탈퇴/영구정지 처리일 |

※ email UNIQUE 제약은 DB 레벨에서 전체 레코드 대상 (탈퇴 포함)
※ nickname은 DB UNIQUE 제약 없음, 앱 레벨에서 deleted_at IS NULL 조건으로 체크

---

## email_verifications
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| token | VARCHAR(255) | UNIQUE, NOT NULL | |
| expires_at | DATETIME | NOT NULL | |
| is_used | BOOLEAN | NOT NULL DEFAULT FALSE | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |

---

## refresh_tokens
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| token | VARCHAR(500) | UNIQUE, NOT NULL | |
| expires_at | DATETIME | NOT NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |

---

## boards
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| slug | VARCHAR(30) | UNIQUE, NOT NULL | |
| name | VARCHAR(50) | NOT NULL | |
| description | VARCHAR(500) | NULL | |
| icon_url | VARCHAR(500) | NULL | |
| banner_url | VARCHAR(500) | NULL | |
| owner_id | BIGINT | FK(users.id), NOT NULL | 개설자 (참조용, 권한 체크는 board_moderators) |
| status | ENUM('ACTIVE','INACTIVE','BANNED') | NOT NULL DEFAULT 'ACTIVE' | |
| is_system | BOOLEAN | NOT NULL DEFAULT FALSE | |
| subscriber_count | INT | NOT NULL DEFAULT 0 | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| deleted_at | DATETIME | NULL | |

---

## board_moderators
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| board_id | BIGINT | FK(boards.id), NOT NULL | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| UNIQUE | | (board_id, user_id) | |

※ 게시판 개설 시 owner도 이 테이블에 자동 삽입
※ 운영자 권한 체크는 이 테이블 단일 기준으로 통일

---

## board_subscriptions
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| board_id | BIGINT | FK(boards.id), NOT NULL | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| UNIQUE | | (board_id, user_id) | |

---

## posts
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| board_id | BIGINT | FK(boards.id), NOT NULL | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| title | VARCHAR(100) | NOT NULL | |
| content | LONGTEXT | NOT NULL | sanitize된 HTML |
| view_count | INT | NOT NULL DEFAULT 0 | |
| upvote_count | INT | NOT NULL DEFAULT 0 | |
| downvote_count | INT | NOT NULL DEFAULT 0 | |
| comment_count | INT | NOT NULL DEFAULT 0 | |
| popularity_score | DOUBLE | NOT NULL DEFAULT 0 | 추천 시 즉시 갱신, 조회수는 배치 갱신 |
| is_pinned | BOOLEAN | NOT NULL DEFAULT FALSE | |
| is_blinded | BOOLEAN | NOT NULL DEFAULT FALSE | |
| modified_at | DATETIME | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| deleted_at | DATETIME | NULL | |

---

## post_drafts
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| board_id | BIGINT | FK(boards.id), NOT NULL | 게시판 선택 필수 |
| title | VARCHAR(100) | NULL | |
| content | LONGTEXT | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |

---

## post_view_logs
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| post_id | BIGINT | FK(posts.id), NOT NULL | |
| user_id | BIGINT | NULL | 비로그인은 NULL |
| ip_address | VARCHAR(45) | NULL | 비로그인용 IP |
| viewed_at | DATETIME | NOT NULL | |

※ 초기 구현: DB 기반 중복 방지
※ 추후 Redis로 전환 예정 (tech-debt-tracker.md 참조)

---

## comments
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| post_id | BIGINT | FK(posts.id), NOT NULL | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| parent_id | BIGINT | FK(comments.id), NULL | NULL=댓글, NOT NULL=대댓글 |
| content | VARCHAR(1000) | NOT NULL | plain text |
| upvote_count | INT | NOT NULL DEFAULT 0 | |
| is_blinded | BOOLEAN | NOT NULL DEFAULT FALSE | |
| modified_at | DATETIME | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| deleted_at | DATETIME | NULL | |

※ parent_id는 반드시 parent_id IS NULL인 댓글만 참조 가능 (앱 레벨 체크)
※ 대댓글 작성 전 parent 댓글의 deleted_at IS NULL 확인 필수

---

## votes
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| target_type | ENUM('POST','COMMENT') | NOT NULL | |
| target_id | BIGINT | NOT NULL | |
| vote_type | ENUM('UP','DOWN') | NOT NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| UNIQUE | | (user_id, target_type, target_id) | |

※ target_type=COMMENT일 때 vote_type=DOWN은 서비스 레이어에서 차단

---

## attachments
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| post_id | BIGINT | FK(posts.id), NULL | NULL=임시 파일 |
| user_id | BIGINT | FK(users.id), NOT NULL | |
| original_name | VARCHAR(255) | NOT NULL | |
| stored_name | VARCHAR(255) | NOT NULL | UUID 파일명 |
| file_type | ENUM('IMAGE','FILE') | NOT NULL | |
| mime_type | VARCHAR(100) | NOT NULL | |
| file_size | BIGINT | NOT NULL | |
| original_url | VARCHAR(500) | NULL | |
| resized_url | VARCHAR(500) | NULL | |
| thumbnail_url | VARCHAR(500) | NULL | |
| is_temp | BOOLEAN | NOT NULL DEFAULT TRUE | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| deleted_at | DATETIME | NULL | |

---

## notifications
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), NOT NULL | 수신자 |
| type | ENUM('COMMENT','REPLY_ON_POST','REPLY_ON_COMMENT','VOTE_POST','VOTE_COMMENT','REPORT_RESULT') | NOT NULL | |
| message | VARCHAR(255) | NOT NULL | |
| target_url | VARCHAR(500) | NOT NULL | |
| is_read | BOOLEAN | NOT NULL DEFAULT FALSE | |
| read_at | DATETIME | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | 자동 삭제 배치 기준 |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |

---

## notification_settings
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK(users.id), UNIQUE, NOT NULL | |
| comment_enabled | BOOLEAN | NOT NULL DEFAULT TRUE | |
| reply_enabled | BOOLEAN | NOT NULL DEFAULT TRUE | |
| vote_enabled | BOOLEAN | NOT NULL DEFAULT TRUE | |
| report_result_enabled | BOOLEAN | NOT NULL DEFAULT TRUE | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |

---

## reports
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| reporter_id | BIGINT | FK(users.id), NOT NULL | |
| target_type | ENUM('POST','COMMENT','USER') | NOT NULL | |
| target_id | BIGINT | NOT NULL | |
| reason | ENUM('SPAM','OBSCENE','ABUSE','ILLEGAL','PRIVACY','COPYRIGHT','OTHER') | NOT NULL | |
| description | VARCHAR(500) | NULL | |
| status | ENUM('PENDING','PROCESSED','DISMISSED') | NOT NULL DEFAULT 'PENDING' | |
| action | ENUM('DELETE_CONTENT','WARN_USER','SUSPEND_USER','BAN_USER','DISMISSED') | NULL | |
| admin_memo | VARCHAR(500) | NULL | |
| processed_by | BIGINT | FK(users.id), NULL | |
| processed_at | DATETIME | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (reporter_id, target_type, target_id) | |

---

## service_settings
| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| setting_key | VARCHAR(100) | UNIQUE, NOT NULL | |
| setting_value | VARCHAR(500) | NOT NULL | |
| description | VARCHAR(255) | NULL | |
| created_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | |

### 초기 데이터 (Flyway seed)
| setting_key | setting_value | description |
|-------------|---------------|-------------|
| auto_blind_threshold | 5 | 자동 블라인드 신고 누적 기준 |
| auto_suspend_warning_count | 3 | 자동 정지 경고 누적 기준 |
| auto_suspend_days | 7 | 자동 정지 기간 (일) |
| signup_enabled | true | 회원가입 허용 여부 |
| notice_banner_enabled | false | 공지 배너 표시 여부 |
| notice_banner_content | | 공지 배너 내용 |

---

## 권장 인덱스

성능에 직접 영향을 주는 쿼리 패턴 기준. Flyway 마이그레이션에 포함할 것.

| 테이블 | 인덱스 컬럼 | 이유 |
|--------|------------|------|
| posts | `(board_id, deleted_at, created_at)` | 게시판별 목록 조회 (정렬 포함) |
| posts | `(popularity_score DESC)` | 인기글 정렬 |
| posts | `(user_id, deleted_at)` | 내 게시글 조회 |
| comments | `(post_id, deleted_at, created_at)` | 댓글 목록 조회 |
| comments | `(parent_id)` | 대댓글 조회 |
| votes | `(target_type, target_id)` | 특정 게시글/댓글 투표 수 집계 |
| notifications | `(user_id, is_read, created_at)` | 사용자 알림 목록 조회 |
| post_view_logs | `(post_id, user_id, viewed_at)` | 조회수 중복 방지 |
| board_subscriptions | `(user_id)` | 구독 게시판 목록 |

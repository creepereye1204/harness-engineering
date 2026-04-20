# comment.md — 댓글 + 대댓글

## 기능 설명
게시글에 댓글과 대댓글(1단계)을 작성하는 기능.
최신순/추천순 정렬 지원.
댓글/대댓글 추천 기능 포함 (비추천 없음).

## 사용자 시나리오

### 댓글 작성
1. 게시글 하단 댓글 입력창에 내용 작성
2. 비로그인 사용자는 댓글 작성 불가 → 로그인 유도
3. 작성 완료 → posts.comment_count 즉시 +1 업데이트
4. 목록 최하단에 즉시 반영

### 대댓글 작성
1. 댓글의 "답글" 버튼 클릭
2. 부모 댓글이 삭제된 경우 답글 버튼 비활성화
3. 해당 댓글 하단에 입력창 노출
4. 작성 완료 → 부모 댓글 바로 아래에 들여쓰기로 표시
5. 대댓글의 대댓글 불가 (parent_id가 있는 댓글에 답글 금지)

### 댓글 수정
1. 본인 댓글만 수정 가능
2. 수정 시 "수정됨" 표시 및 수정 시각 기록
3. 인라인 수정 (페이지 이동 없이)

### 댓글 삭제
1. 본인 댓글 또는 게시판 운영자/관리자 삭제 가능
2. soft delete 처리 (deleted_at 기록)
3. 삭제 시 posts.comment_count 즉시 -1 업데이트
4. 삭제된 댓글 처리:
   - 대댓글이 없는 경우: 목록에서 완전히 숨김
   - 대댓글이 있는 경우: "삭제된 댓글입니다" 표시, 대댓글 유지

### 댓글 목록 조회
1. 기본 정렬: 최신순
2. 추천순으로 변경 가능
3. 더보기 방식: ?lastId={마지막_id}&size=20
4. 댓글 > 대댓글 순으로 트리 구조 렌더링

## 상세 요구사항

### 댓글
- 내용: 1~1,000자
- 줄바꿈 허용, HTML 태그 불가 (plain text)

### 대댓글
- 내용: 1~1,000자
- 부모 댓글(parent_id 대상)이 deleted_at IS NOT NULL이면 대댓글 작성 불가
- 대댓글의 대댓글 작성 불가 (parent_id가 NULL인 댓글만 부모 허용)

### 댓글 수
- posts.comment_count는 댓글+대댓글 합산
- 삭제된 댓글/대댓글은 카운트에서 제외
- 카운터 증감은 벌크 업데이트 쿼리 사용

## API 목록

| 메서드 | 엔드포인트 | 설명 | 인증 |
|--------|-----------|------|------|
| GET | /api/v1/posts/{postId}/comments | 댓글 목록 조회 | 불필요 |
| POST | /api/v1/posts/{postId}/comments | 댓글 작성 | 필요 |
| PATCH | /api/v1/posts/{postId}/comments/{commentId} | 댓글 수정 | 필요 |
| DELETE | /api/v1/posts/{postId}/comments/{commentId} | 댓글 삭제 | 필요 |
| POST | /api/v1/posts/{postId}/comments/{commentId}/replies | 대댓글 작성 | 필요 |
| PATCH | /api/v1/posts/{postId}/comments/{commentId}/replies/{replyId} | 대댓글 수정 | 필요 |
| DELETE | /api/v1/posts/{postId}/comments/{commentId}/replies/{replyId} | 대댓글 삭제 | 필요 |

## 예외 처리

| 상황 | HTTP 상태 | 에러 코드 | 메시지 |
|------|-----------|-----------|--------|
| 존재하지 않는 댓글 | 404 | COMMENT_NOT_FOUND | "존재하지 않는 댓글입니다" |
| 삭제된 댓글에 대댓글 | 400 | COMMENT_PARENT_DELETED | "삭제된 댓글에는 답글을 달 수 없습니다" |
| 대댓글에 대댓글 시도 | 400 | COMMENT_REPLY_NOT_ALLOWED | "대댓글에는 답글을 달 수 없습니다" |
| 수정 권한 없음 | 403 | COMMENT_FORBIDDEN | "본인 댓글만 수정할 수 있습니다" |
| 삭제 권한 없음 | 403 | COMMENT_DELETE_FORBIDDEN | "삭제 권한이 없습니다" |
| 글자 수 초과 | 400 | COMMENT_TOO_LONG | "댓글은 1,000자 이내로 작성해주세요" |

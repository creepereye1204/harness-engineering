# design-docs/components.md — 공통 컴포넌트

## 컴포넌트 목록
Claude Code는 아래 컴포넌트를 재사용하고
새로 만들기 전에 반드시 이 목록 먼저 확인할 것.

| 컴포넌트 | 위치 | 용도 |
|----------|------|------|
| Button | components/common/Button | 모든 버튼 |
| Input | components/common/Input | 텍스트 입력 |
| Toast | components/common/Toast | 알림 메시지 |
| Modal | components/common/Modal | 팝업 |
| Skeleton | components/common/Skeleton | 로딩 상태 |
| Avatar | components/common/Avatar | 프로필 이미지 |
| Badge | components/common/Badge | 뱃지/태그 |
| Dropdown | components/common/Dropdown | 드롭다운 메뉴 |
| Pagination | components/common/Pagination | 페이지네이션 |
| PostCard | components/post/PostCard | 게시글 목록 카드 |
| CommentItem | components/comment/CommentItem | 댓글 단위 |
| VoteButton | components/vote/VoteButton | 추천/비추천 버튼 |
| BoardCard | components/board/BoardCard | 게시판 카드 |
| NotificationItem | components/notification/NotificationItem | 알림 단위 |

## Button 변형
- variant: primary / secondary / ghost / danger
- size: sm / md / lg
- 로딩 상태: isLoading prop → 스피너 표시

## Toast 규칙
- 성공: 초록 / 2초 후 자동 닫힘
- 에러: 빨강 / 4초 후 자동 닫힘
- 정보: 파랑 / 3초 후 자동 닫힘
- 화면 우측 하단 고정

## 스켈레톤 규칙
- 모든 데이터 로딩 구간에 Skeleton 사용
- 실제 컴포넌트와 동일한 크기/형태로 제작

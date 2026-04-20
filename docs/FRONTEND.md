# docs/FRONTEND.md — 프론트엔드 가이드

## 개요
React 프론트엔드 개발 시 Claude Code가 따라야 할 규칙과 패턴 모음.

## 기술 스택
- React 18 + Vite
- styled-components
- React Router v6
- Zustand (전역 상태)
- Axios (HTTP 클라이언트)
- TipTap (리치 에디터)
- React Query (서버 상태 관리)

## 폴더 구조
```
src/
├── components/
│   ├── common/       # Button, Input, Toast, Modal 등
│   ├── post/         # PostCard, PostList 등
│   ├── comment/      # CommentItem, CommentList 등
│   ├── board/        # BoardCard 등
│   ├── vote/         # VoteButton 등
│   └── notification/ # NotificationItem 등
├── pages/
│   ├── Home/
│   ├── Board/
│   ├── Post/
│   ├── Auth/
│   ├── Profile/
│   ├── Search/
│   └── Admin/
├── hooks/            # 커스텀 훅
├── api/              # API 호출 함수
├── store/            # Zustand 스토어
├── styles/           # 테마, 글로벌 스타일
└── utils/            # 공통 유틸
```

## 규칙

### 컴포넌트
- 함수형 컴포넌트 + hooks만 사용 (클래스 컴포넌트 금지)
- 컴포넌트당 파일 1개 원칙
- props 타입은 PropTypes로 명시 (TypeScript 미도입 프로젝트 기준)
- 200줄을 넘으면 분리한다. 단, 단일 폼처럼 UI 흐름이 하나인 경우 250줄까지 허용한다.

### 스타일
- 하드코딩 금지 — 반드시 theme.* 사용
- 인라인 스타일 금지
- 전역 스타일은 styles/GlobalStyle.js에만 작성

### 상태 관리
- 서버 데이터: React Query
- 전역 클라이언트 상태: Zustand (인증, 테마, 알림)
- 로컬 UI 상태: useState

### API 호출
- 모든 API 호출은 api/ 폴더에 함수로 분리
- 컴포넌트 내 직접 axios 호출 금지
- Access Token은 Axios 인터셉터로 자동 주입
- 401 응답 시 Refresh Token으로 자동 재발급 시도

### 낙관적 업데이트
- 추천/비추천: 즉시 UI 반영 후 API 호출
- 롤백 구현: `useMutation`의 `onMutate`에서 `queryClient.getQueryData`로 이전 값을 저장하고, `onError`에서 `queryClient.setQueryData`로 복원한다.

### 에러 처리
- API 에러는 Toast로 표시
- 404: 전용 404 페이지로 이동
- 500: "일시적인 오류가 발생했습니다" Toast 표시

### 로딩 처리
- 데이터 로딩 중 반드시 Skeleton UI 표시
- 버튼 클릭 후 응답 대기 중 isLoading 상태로 비활성화

## SSE 연결 관리
- 로그인 시 SSE 자동 연결
- 연결 끊김 시 3초 후 자동 재연결 (최대 5회 시도 후 중단). 중단 후 헤더 알림 아이콘을 회색으로 표시하고 툴팁으로 "알림 연결 끊김 — 새로고침하면 재연결됩니다"를 노출한다.
- 로그아웃 시 연결 종료
- useNotification 커스텀 훅으로 관리

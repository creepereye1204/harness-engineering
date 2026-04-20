# docs/RELIABILITY.md — 안정성 기준

## 개요
서비스 안정성을 위해 Claude Code가 반드시 지켜야 할 기준.

## 백엔드 안정성

### 트랜잭션 관리
- 데이터 변경이 있는 모든 Service 메서드에 `@Transactional` 적용
- 읽기 전용 메서드는 `@Transactional(readOnly = true)`
- 트랜잭션 내에서 외부 API 호출 금지 (이메일 발송 등은 트랜잭션 밖에서 `@Async` 처리)

### 예외 처리
- 모든 예외는 `GlobalExceptionHandler`에서 통합 처리
- `RuntimeException`을 최상위에서 catch해 삼키지 말 것. 해당 예외를 처리할 수 있는 레이어에서만 catch한다. 처리할 수 없으면 상위로 던진다. `GlobalExceptionHandler`가 최종 방어선이다.
- 예외 발생 시 반드시 로그 기록: 예상된 비즈니스 예외(잘못된 입력, 권한 없음 등)는 `WARN`, 예상치 못한 시스템 예외는 `ERROR`

### 배치 작업
- 임시 파일 24시간 후 자동 삭제
- 오래된 알림 자동 삭제 (읽음 30일, 미읽음 90일, created_at 기준)
- 인기글 점수 조회수 반영 (10분 간격)
- 배치 실패 시 로그 기록 후 다음 주기에 재시도 (예외를 삼키지 말 것)

### DB 안정성
- **커넥션 풀 (HikariCP)**
  - 로컬: `maximum-pool-size: 10` (기본값)
  - 운영: `minimum-idle: 5`, `maximum-pool-size: 20`
- **쿼리 타임아웃**
  - 읽기 쿼리: 5초
  - 쓰기 쿼리: 30초
- Flyway 마이그레이션 실패 시 서버 시작 중단 (기본 동작, 변경 금지)

## 프론트엔드 안정성

### API 통신
- React Query(`useQuery` / `useMutation`)를 통한 호출은 `onError` 콜백으로 에러를 처리한다. try-catch를 추가하면 `isError` 상태가 동작하지 않는다.
- try-catch는 React Query 밖에서 직접 호출하는 경우(`useEffect` 내 수동 호출 등)에만 사용한다.
- 네트워크 오류 시 사용자에게 Toast로 안내
- 토큰 만료(401) 시 Refresh Token으로 자동 재발급 후 재시도

### SSE 연결
- 연결 끊김 감지 시 3초 후 자동 재연결
- 재연결 실패 시 최대 5회 시도 후 중단
- 재연결 중 발생한 알림은 알림 목록 API로 보완 (유실 없음)

## Docker 안정성

### 컨테이너 설정
- 모든 컨테이너에 `restart: unless-stopped` 적용
- 헬스체크: Spring Boot Actuator `/actuator/health`
- 볼륨 마운트로 데이터 영속성 보장 (MySQL 데이터, 업로드 파일)

### 로컬 개발 환경
- `docker compose up` 한 번으로 전체 환경 실행
- 환경변수는 `.env` 파일로 관리 (`.env.example` 키 목록 제공, 값 비워둠)
- 컨테이너 재시작 후에도 데이터 유지

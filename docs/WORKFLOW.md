## 5. 워크플로우

### 작업 시작 전
1. `docs/exec-plans/active/`에서 현재 Phase와 작업 항목 확인
2. 해당 도메인의 `docs/product-specs/{도메인}.md` 정독
3. `docs/generated/db-schema.md`에서 관련 테이블 확인
4. 스펙에 없는 동작이 필요하면 구현 전 사용자에게 확인

### 백엔드 구현 순서
Entity → Repository → Service → Controller → Test
Flyway 마이그레이션은 작성 후 실행 전 반드시 사용자 확인을 받는다.

### 프론트엔드 구현 순서
`docs/design-docs/`에서 컴포넌트/색상/타이포그래피 확인
→ 공통 컴포넌트 재사용 우선
→ API 연동
→ 주요 플로우 동작 확인

### 작업 완료 기준
exec-plan의 해당 Phase **완료 기준** 항목이 모두 통과해야 완료로 간주한다.
완료 후 exec-plan의 체크박스를 업데이트한다.

### 의사결정 규칙
- 스펙이 모호하면 → 사용자에게 질문
- 라이브러리 추가가 필요하면 → 사용자 승인 후 진행
- DB 스키마 변경이 필요하면 → Flyway 파일 작성 후 실행 전 승인 요청
- 테스트가 실패하면 → 테스트를 수정하지 말고 코드를 수정
# attachment.md — 파일/이미지 첨부

## 기능 설명
게시글 작성 시 이미지 및 파일을 첨부하는 기능.
이미지는 리치 에디터 내 인라인 삽입 지원.
파일은 게시글 하단에 첨부 목록으로 표시.
저장소: 로컬 서버 (Docker 볼륨)

## 사용자 시나리오

### 이미지 인라인 삽입
1. 리치 에디터에서 이미지 업로드 버튼 클릭
2. 파일 선택 또는 드래그 앤 드롭
3. 업로드 완료 → 에디터 커서 위치에 이미지 즉시 삽입
4. 이미지 클릭 시 크기 조절 가능

### 파일 첨부
1. 게시글 작성 화면 하단 파일 첨부 영역에 파일 추가
2. 여러 파일 동시 첨부 가능
3. 게시글 발행 시 첨부 파일 목록이 본문 하단에 표시
4. 독자는 파일명 클릭 시 다운로드

### 임시 업로드
1. 게시글 발행 전 이미지 업로드 시 임시 저장
2. 게시글 발행 완료 시 임시 → 정식 파일로 전환
3. 임시 파일은 24시간 후 자동 삭제 (발행 안 된 경우)

### 파일 삭제
1. 게시글 수정 시 첨부 파일 개별 삭제 가능
2. 게시글 삭제 시 연관 파일 전체 soft delete
3. 실제 파일은 배치로 주기적 정리 (hard delete)

## 상세 요구사항

### 이미지
- 허용 형식: jpg, jpeg, png, gif, webp
- 최대 크기: 파일당 10MB
- 게시글당 최대 20장
- 업로드 시 서버에서 리사이징:
  - 원본 보관
  - 썸네일 생성 (300×300)
  - 본문용 최대 너비 1200px로 압축

### 일반 파일
- 허용 형식: pdf, zip, txt, doc, docx, xls, xlsx, ppt, pptx
- 최대 크기: 파일당 50MB
- 게시글당 최대 5개

### 저장 구조 (Docker 볼륨)
```
/uploads/
├── temp/
│   └── {uuid}.{ext}
├── images/
│   ├── original/{yyyy}/{mm}/{dd}/{uuid}.{ext}
│   ├── thumbnail/{yyyy}/{mm}/{dd}/{uuid}.{ext}
│   └── resized/{yyyy}/{mm}/{dd}/{uuid}.{ext}
└── files/
    └── {yyyy}/{mm}/{dd}/{uuid}.{ext}
```

### 보안
- 파일명은 UUID로 저장 (원본 파일명 노출 X)
- 업로드 시 MIME 타입 검증 (확장자만으로 신뢰 X)
- 실행 파일 (.exe, .sh, .bat 등) 업로드 차단

## API 목록

| 메서드 | 엔드포인트 | 설명 | 인증 |
|--------|-----------|------|------|
| POST | /api/v1/attachments/image | 이미지 임시 업로드 | 필요 |
| POST | /api/v1/attachments/file | 파일 임시 업로드 | 필요 |
| DELETE | /api/v1/attachments/{attachmentId} | 첨부 파일 삭제 | 필요 |
| GET | /api/v1/attachments/{attachmentId}/download | 파일 다운로드 | 불필요 |

### 이미지 업로드 응답 예시
```json
{
  "success": true,
  "data": {
    "attachmentId": "uuid",
    "originalUrl": "/uploads/images/original/2024/01/01/uuid.jpg",
    "resizedUrl": "/uploads/images/resized/2024/01/01/uuid.jpg",
    "thumbnailUrl": "/uploads/images/thumbnail/2024/01/01/uuid.jpg"
  }
}
```

## 예외 처리

| 상황 | HTTP 상태 | 에러 코드 | 메시지 |
|------|-----------|-----------|--------|
| 허용되지 않는 파일 형식 | 400 | ATTACHMENT_INVALID_TYPE | "업로드할 수 없는 파일 형식입니다" |
| 파일 크기 초과 | 400 | ATTACHMENT_SIZE_EXCEEDED | "파일 크기는 최대 10MB입니다" |
| 첨부 한도 초과 | 400 | ATTACHMENT_LIMIT_EXCEEDED | "이미지는 최대 20장까지 첨부할 수 있습니다" |
| 실행 파일 업로드 | 400 | ATTACHMENT_FORBIDDEN_TYPE | "보안상 업로드할 수 없는 파일입니다" |
| 임시 파일 만료 | 404 | ATTACHMENT_NOT_FOUND | "만료되었거나 존재하지 않는 파일입니다" |

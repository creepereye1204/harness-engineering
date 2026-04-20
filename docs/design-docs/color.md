# design-docs/color.md — 컬러 시스템

## 브랜드 컬러
- Primary: #FF6B35 (활기찬 오렌지)
- Primary Hover: #E55A25
- Secondary: #4ECDC4 (청록)
- Accent: #FFE66D (노랑)

## 라이트 모드
- Background: #F8F9FA
- Surface: #FFFFFF
- Border: #DEE2E6
- Text Primary: #212529
- Text Secondary: #6C757D
- Text Disabled: #ADB5BD

## 다크 모드
- Background: #1A1A1A
- Surface: #242424
- Border: #3A3A3A
- Text Primary: #F8F9FA
- Text Secondary: #ADB5BD
- Text Disabled: #6C757D

## 시맨틱 컬러
- Success: #51CF66
- Warning: #FFD43B
- Error: #FF6B6B
- Info: #339AF0

## styled-components 테마 적용
모든 색상은 ThemeProvider로 주입.
하드코딩 금지 — 반드시 props.theme.colors.* 사용.

예시:
const Button = styled.button`
  background: ${({ theme }) => theme.colors.primary};
  color: ${({ theme }) => theme.colors.textPrimary};
`;

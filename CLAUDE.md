# 서비스 컨텍스트

## 서비스 개요
- **서비스명**: [서비스명 입력]
- **플랫폼**: [iOS / Android / Web / 전체]
- **대상 사용자**: [주요 사용자층 설명]
- **서비스 카테고리**: [e.g. 쇼핑, 핀테크, 헬스케어, 생산성 등]

## Figma 연동
- **검토 단위**: 스프린트별 분리된 Figma 파일 전체
- **파일 키 전달 방식**: 명령 실행 시 인자로 전달 — `/project:review-ux [FIGMA_FILE_KEY]`
- **파일 키는 CLAUDE.md에 고정하지 않음** — 스프린트마다 파일이 바뀌므로

---

# UX 라이팅 원칙 (요약)

> 상세 규칙은 `.claude/rules/ux-writing.md` 참조

1. 사용자 중심의 명확한 언어 사용
2. 버튼/CTA는 동사로 시작, 2단어 이내
3. 에러 메시지는 원인 + 해결책 구조로 작성
4. 금지어 및 브랜드 톤앤매너 준수

---

# 에이전트 구성

이 프로젝트는 Figma 디자인 화면을 자동 검토하는 멀티 에이전트 시스템입니다.

## 활성 에이전트
- `ux-writing-checker` — UX 라이팅 규칙 검토

## 예정 에이전트
- `accessibility-checker` — 터치 타겟, 색상 대비율 등 접근성 검사
- `design-system-checker` — 디자인 시스템/토큰 준수 여부
- `interaction-checker` — 인터랙티브 요소 상태 누락 여부
- `responsive-checker` — 반응형 레이아웃 및 브레이크포인트 검사

---

# 규칙

- 검토 결과는 항상 `./review-reports/` 폴더에 마크다운으로 저장
- Severity는 CRITICAL / WARNING / INFO 3단계로 분류
- Figma 노드 데이터는 Figma MCP 서버를 통해 직접 추출 (JSON 수동 추출 금지)

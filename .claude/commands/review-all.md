아래 Figma 파일을 대상으로 두 가지 검토를 순서대로 실행해줘.

$ARGUMENTS

첫 번째 인자는 Figma 파일 키 또는 URL, 두 번째 인자는 PRD 파일 경로야.

## 실행 순서

### 1단계 — PRD 대조 검토 (prd-checker 에이전트)

prd-checker 에이전트를 실행해서 PRD 요구사항이 Figma에 반영됐는지 검토해줘.
- Figma 입력: 첫 번째 인자 (파일 키 또는 URL 전체)
- PRD 입력: 두 번째 인자 (로컬 .md 또는 .pdf 파일 경로)
- PDF 파일이면 `pdftotext` 로 텍스트 추출 (Bash 사용)
- Figma MCP가 403이면 `curl -H "X-Figma-Token: $FIGMA_API_KEY"` 로 직접 호출
- 결과를 `/Users/gimhyeon-yeong/Desktop/developer/클로드 코드/my-agent/review-reports/YYYYMMDD_PRD review.md` 에 저장

1단계가 완전히 끝난 후 2단계를 시작해.

### 2단계 — UX 라이팅 검토 (ux-writing-checker 에이전트)

ux-writing-checker 에이전트를 실행해서 Figma 텍스트 노드의 UX 라이팅 규칙 준수 여부를 검토해줘.
- Figma 입력: 첫 번째 인자와 동일한 파일
- 결과를 `/Users/gimhyeon-yeong/Desktop/developer/클로드 코드/my-agent/review-reports/YYYYMMDD_ux-writing review.md` 에 저장

### 완료 후

두 리포트 파일 경로와 각 검토의 핵심 이슈(CRITICAL·WARNING 건수)를 요약해서 보여줘.

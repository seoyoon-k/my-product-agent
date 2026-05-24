prd-checker 에이전트를 사용해서 아래 PRD 문서와 Figma 파일을 대조 검토해줘.

$ARGUMENTS

첫 번째 인자는 Figma 파일 키 또는 Figma URL 전체, 두 번째 인자는 PRD 파일 경로야. (마크다운 .md 또는 PDF .pdf 모두 가능)

Figma 입력 형식은 세 가지 모두 지원해:
- 파일 키만: `bgwwSDXFTSx3LXZ8An64TA`
- Figma URL 전체: `https://www.figma.com/design/bgwwSDXFTSx3LXZ8An64TA/파일명?node-id=3622-52078`
  → URL에서 파일 키(`/design/` 다음 세그먼트)와 node-id 쿼리 파라미터를 자동으로 추출해줘.
- 파일 키 + node-id 별도: `bgwwSDXFTSx3LXZ8An64TA 3622-52078 /path/to/prd.pdf`

Figma 파일은 항상 Figma 클라우드(figma.com)에 있으므로 로컬 파일 경로가 아닌 파일 키 또는 URL로 접근한다.

검토 절차:
1. PRD 파일을 읽어서 디자인 요구사항을 추출해줘.
   - .pdf 파일이면 `pdftotext "{파일경로}" -` 명령어로 텍스트 추출 (Bash 사용)
   - .md 파일이면 Read 도구로 직접 읽기
2. Figma 데이터를 가져와줘.
   - 먼저 mcp__figma__get_figma_data 시도
   - 403 오류가 나면 `curl -s -H "X-Figma-Token: $FIGMA_API_KEY" "https://api.figma.com/v1/files/{파일키}/nodes?ids={노드ID}"` 로 직접 호출 (Bash 사용)
   - 노드 ID가 없으면 `https://api.figma.com/v1/files/{파일키}` 전체 파일 호출
3. PRD 요구사항과 Figma 디자인을 텍스트/구조/시각 세 가지 레벨로 대조 검토해줘.
4. 검토 결과를 `/Users/gimhyeon-yeong/Desktop/developer/클로드 코드/my-agent/review-reports/YYYYMMDD_PRD review.md` 파일로 저장해줘.

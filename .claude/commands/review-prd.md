prd-checker 에이전트를 사용해서 아래 PRD 문서와 Figma 파일을 대조 검토해줘.

$ARGUMENTS

첫 번째 인자는 Figma 파일 키 또는 Figma URL 전체, 두 번째 인자는 PRD 파일 경로야. (마크다운 .md 또는 PDF .pdf 모두 가능)

## 0단계 — 입력값 검증 (검토 시작 전 반드시 실행)

아래 bash 스크립트를 실행해서 입력값을 검증해줘.
검증에 실패하면 **즉시 중단**하고 안내 메시지만 출력해줘. 검토는 진행하지 마.

```bash
FIGMA_INPUT="첫 번째 인자"
PRD_PATH="두 번째 인자"

# URL이면 파일 키 추출, 아니면 그대로 사용
if echo "$FIGMA_INPUT" | grep -q "figma.com"; then
  FIGMA_KEY=$(echo "$FIGMA_INPUT" | sed 's|.*figma\.com/[^/]*/\([^/?]*\).*|\1|')
else
  FIGMA_KEY=$(echo "$FIGMA_INPUT" | awk '{print $1}')
fi

ERRORS=""

# 검사 1: Figma 파일 키 형식 (영문+숫자 22자)
if ! echo "$FIGMA_KEY" | grep -qE '^[a-zA-Z0-9]{22}$'; then
  ERRORS="$ERRORS\n❌ Figma 파일 키 형식이 올바르지 않습니다.\n   입력값: $FIGMA_KEY\n   올바른 형식: 영문+숫자 22자 (예: bgwwSDXFTSx3LXZ8An64TA)\n   Figma URL에서 확인: https://www.figma.com/design/[FILE_KEY]/..."
fi

# 검사 2: PRD 파일 존재 여부
if [ ! -f "$PRD_PATH" ]; then
  ERRORS="$ERRORS\n❌ PRD 파일을 찾을 수 없습니다.\n   입력 경로: $PRD_PATH\n   경로를 다시 확인해주세요."
fi

# 오류가 있으면 출력 후 중단
if [ -n "$ERRORS" ]; then
  echo -e "$ERRORS"
  exit 1
fi

echo "✅ 입력값 검증 통과 (파일 키: $FIGMA_KEY / PRD: $PRD_PATH)"
```

검증을 통과한 경우에만 아래 검토 절차를 진행해줘.

---

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

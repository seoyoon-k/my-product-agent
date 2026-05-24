ux-writing-checker 에이전트를 사용해서 아래 Figma 파일의 UX 라이팅을 검토해줘.

$ARGUMENTS

첫 번째 인자는 Figma 파일 키 또는 URL 전체야.

## 0단계 — 입력값 검증 (검토 시작 전 반드시 실행)

아래 bash 스크립트를 실행해서 입력값을 검증해줘.
검증에 실패하면 **즉시 중단**하고 안내 메시지만 출력해줘. 검토는 진행하지 마.

```bash
INPUT="첫 번째 인자"

# URL이면 파일 키 추출, 아니면 그대로 사용
if echo "$INPUT" | grep -q "figma.com"; then
  FIGMA_KEY=$(echo "$INPUT" | sed 's|.*figma\.com/[^/]*/\([^/?]*\).*|\1|')
else
  FIGMA_KEY=$(echo "$INPUT" | awk '{print $1}')
fi

# Figma 파일 키: 영문+숫자 22자 형식 검사
if ! echo "$FIGMA_KEY" | grep -qE '^[a-zA-Z0-9]{22}$'; then
  echo "❌ Figma 파일 키 형식이 올바르지 않습니다."
  echo ""
  echo "입력값: $FIGMA_KEY"
  echo "올바른 형식: 영문+숫자 22자 (예: miOopYqbIg0gMMJbmk8Y3q)"
  echo ""
  echo "Figma 파일 URL에서 키를 확인하세요:"
  echo "https://www.figma.com/design/[FILE_KEY]/..."
  exit 1
fi

echo "✅ 입력값 검증 통과 (파일 키: $FIGMA_KEY)"
```

검증을 통과한 경우에만 아래 검토 절차를 진행해줘.

---

Figma 입력 형식은 세 가지 모두 지원해:
- 파일 키만: `miOopYqbIg0gMMJbmk8Y3q`
- Figma URL 전체: `https://www.figma.com/design/miOopYqbIg0gMMJbmk8Y3q/파일명?node-id=0-1`
  → URL에서 파일 키(`/design/` 다음 세그먼트)와 node-id 쿼리 파라미터를 자동으로 추출해줘.
- 파일 키 + node-id 별도: `miOopYqbIg0gMMJbmk8Y3q 0-1`

검토 절차:
1. Figma 데이터를 가져와줘.
   - 먼저 mcp__figma__get_figma_data 시도
   - 403 오류가 나면 `curl -s -H "X-Figma-Token: $FIGMA_API_KEY" "https://api.figma.com/v1/files/{파일키}"` 로 직접 호출 (Bash 사용)
2. Figma의 모든 텍스트 노드를 추출해줘.
3. `.claude/rules/ux-writing.md` 규칙과 대조해서 위반 항목을 검토해줘.
4. 검토 결과를 `/Users/gimhyeon-yeong/Desktop/developer/클로드 코드/my-agent/review-reports/YYYYMMDD_ux-writing review.md` 파일로 저장해줘.

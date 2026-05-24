아래 Figma 파일을 대상으로 두 가지 검토를 순서대로 실행해줘.

$ARGUMENTS

첫 번째 인자는 Figma 파일 키 또는 URL, 두 번째 인자는 PRD 파일 경로야.

## 실행 순서

1. **prd-checker 에이전트** — 첫 번째 인자(Figma)와 두 번째 인자(PRD 파일)를 넘겨서 실행해줘.
2. 1단계가 완전히 끝난 후, **ux-writing-checker 에이전트** — 첫 번째 인자(Figma)만 넘겨서 실행해줘.

완료 후, 두 리포트의 핵심 이슈(CRITICAL·WARNING 건수)를 한눈에 볼 수 있도록 요약해줘.

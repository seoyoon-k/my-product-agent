# UX 라이팅 검토 에이전트

Figma 디자인 파일의 텍스트 노드를 자동으로 추출하고, UX 라이팅 규칙 준수 여부를 검토해 마크다운 리포트로 저장하는 Claude Code 멀티 에이전트 시스템입니다.

---

## 사전 준비

### 1. Figma API 토큰 발급

1. [figma.com](https://www.figma.com) 로그인
2. 좌측 상단 프로필 아이콘 → **Settings**
3. **Security** 탭 → **Personal access tokens** → **Generate new token**
4. 발급된 토큰 복사

### 2. API 토큰 등록

`.claude/settings.local.json` 파일을 열고 `YOUR_FIGMA_API_KEY` 부분에 토큰을 입력하세요.

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--stdio"],
      "env": {
        "FIGMA_API_KEY": "여기에_토큰_입력"
      }
    }
  }
}
```

그 다음 터미널에서 아래 명령어를 실행해 Claude Code에 MCP 서버를 등록하세요.

```bash
claude mcp add figma -e FIGMA_API_KEY=여기에_토큰_입력 -- npx -y figma-developer-mcp --stdio
```

> ⚠️ `settings.local.json`은 `.gitignore`에 추가해 토큰이 외부에 노출되지 않도록 하세요.

### 3. UX 라이팅 규칙 작성

`.claude/rules/ux-writing.md` 파일에 서비스의 UX 라이팅 규칙을 작성하세요.  
파일 내 `[입력]` 플레이스홀더를 실제 규칙으로 채우면 됩니다.

---

## 사용 방법

Claude Code에서 아래 명령어를 실행하세요.

```
/review-ux [FIGMA_FILE_KEY]
```

**Figma 파일 키 찾는 법**: Figma 파일 URL에서 확인할 수 있어요.

```
https://www.figma.com/design/[FILE_KEY]/파일명...
                              ↑ 이 부분
```

### 예시

```
/review-ux miOopYqbIg0gMMJbmk8Y3q
```

---

## 결과물

검토가 완료되면 `./review-reports/` 폴더에 마크다운 파일로 저장됩니다.

```
review-reports/
└── ux-writing-review-[FILE_KEY]-[DATE].md
```

리포트는 아래 3단계 Severity로 이슈를 분류합니다.

| Severity | 설명 |
|----------|------|
| 🔴 CRITICAL | 즉시 수정이 필요한 규칙 위반 |
| 🟡 WARNING | 개선이 권장되는 항목 |
| 🔵 INFO | 참고 사항 또는 확인 완료 항목 |

---

## 에이전트 구성

| 에이전트 | 상태 | 역할 |
|----------|------|------|
| `ux-writing-checker` | ✅ 활성 | UX 라이팅 규칙 검토 |
| `accessibility-checker` | 🔜 예정 | 터치 타겟, 색상 대비율 등 접근성 검사 |
| `design-system-checker` | 🔜 예정 | 디자인 시스템·토큰 준수 여부 |
| `interaction-checker` | 🔜 예정 | 인터랙티브 요소 상태 누락 여부 |
| `responsive-checker` | 🔜 예정 | 반응형 레이아웃 및 브레이크포인트 |

---

## 파일 구조

```
my-agent/
├── .claude/
│   ├── rules/
│   │   └── ux-writing.md        # UX 라이팅 규칙 (직접 작성 필요)
│   ├── settings.json            # 프로젝트 공통 설정
│   └── settings.local.json      # 로컬 전용 설정 (토큰 등, git 제외)
├── review-reports/              # 검토 결과 저장 폴더
├── CLAUDE.md                    # Claude 에이전트 운영 지침
└── README.md                    # 이 파일
```

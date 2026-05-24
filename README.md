# 디자인 자동 검토 에이전트

Figma 파일과 PRD 문서를 입력하면 Claude Code가 자동으로 검토하고 마크다운 리포트를 생성하는 멀티 에이전트 시스템입니다.

---

## 사전 준비

### 1. Figma API 토큰 발급

1. [figma.com](https://www.figma.com) 로그인
2. 좌측 상단 프로필 → **Settings** → **Security** 탭
3. **Personal access tokens** → **Generate new token**
4. 발급된 토큰 복사

### 2. API 토큰 등록

터미널에서 아래 명령어를 실행해 Claude Code에 토큰을 등록하세요.

```bash
claude mcp add figma -e FIGMA_API_KEY=여기에_토큰_입력 -- npx -y figma-developer-mcp --stdio
```

또는 `.claude/settings.local.json` 파일에 직접 입력:

```json
{
  "env": {
    "FIGMA_API_KEY": "여기에_토큰_입력"
  }
}
```

> ⚠️ `settings.local.json`은 `.gitignore`에 추가해 토큰이 외부에 노출되지 않도록 하세요.

### 3. UX 라이팅 규칙 작성

`.claude/rules/ux-writing.md` 파일에 서비스의 UX 라이팅 규칙을 작성하세요.  
파일 내 `[입력]` 플레이스홀더를 실제 규칙으로 채우면 됩니다.

---

## 사용 방법

### `/review-ux` — UX 라이팅 검토

Figma 파일의 모든 텍스트 노드를 추출해 UX 라이팅 규칙 준수 여부를 검토합니다.

```
/review-ux [FIGMA_FILE_KEY_또는_URL]
```

```
# 파일 키만
/review-ux miOopYqbIg0gMMJbmk8Y3q

# URL 전체도 가능
/review-ux https://www.figma.com/design/miOopYqbIg0gMMJbmk8Y3q/파일명
```

---

### `/review-prd` — PRD × Figma 대조 검토

PRD 문서에서 디자인 요구사항을 추출하고 Figma 디자인에 반영됐는지 검증합니다.

```
/review-prd [FIGMA_FILE_KEY_또는_URL] [PRD_파일_경로]
```

```
# 파일 키 + 로컬 PDF
/review-prd bgwwSDXFTSx3LXZ8An64TA /path/to/prd.pdf

# Figma URL 전체 (node-id 자동 추출)
/review-prd https://www.figma.com/design/bgwwSDXFTSx3LXZ8An64TA/파일명?node-id=3622-52078 /path/to/prd.md
```

**Figma 입력 형식** 세 가지 모두 지원:
- 파일 키: `bgwwSDXFTSx3LXZ8An64TA`
- URL 전체: `https://www.figma.com/design/[파일키]/...?node-id=...` (파일 키·node-id 자동 추출)
- 파일 키 + node-id 분리: `bgwwSDXFTSx3LXZ8An64TA 3622-52078`

**PRD 형식**: 로컬 `.md` 또는 `.pdf` 파일 경로

---

### `/review-all` — 전체 검토 (PRD + UX 라이팅 순차 실행)

PRD 대조 검토와 UX 라이팅 검토를 순서대로 실행하고 개별 리포트를 각각 저장합니다.

```
/review-all [FIGMA_FILE_KEY_또는_URL] [PRD_파일_경로]
```

```
/review-all bgwwSDXFTSx3LXZ8An64TA /path/to/prd.pdf
```

실행 순서:
1. `prd-checker` — PRD 요구사항 vs Figma 반영 여부 검토
2. `ux-writing-checker` — 텍스트 노드 UX 라이팅 규칙 검토
3. 두 리포트를 `./review-reports/`에 각각 저장

---

## 결과물

검토가 완료되면 `./review-reports/` 폴더에 마크다운 파일로 저장됩니다.

```
review-reports/
├── YYYYMMDD_PRD review.md          # PRD 대조 검토 결과
└── YYYYMMDD_ux-writing review.md   # UX 라이팅 검토 결과
```

### Severity 기준

| Severity | 설명 |
|----------|------|
| 🚨 CRITICAL | 즉시 수정이 필요한 오류 |
| ⚠️ WARNING | 개선이 권장되는 항목 |
| ℹ️ INFO | 참고 사항 |

---

## 에이전트 구성

| 에이전트 | 상태 | 역할 | 트리거 커맨드 |
|----------|------|------|--------------|
| `prd-checker` | ✅ 활성 | PRD 요구사항 vs Figma 반영 여부 검토 | `/review-prd`, `/review-all` |
| `ux-writing-checker` | ✅ 활성 | UX 라이팅 규칙 준수 여부 검토 | `/review-ux`, `/review-all` |
| `accessibility-checker` | 🔜 예정 | 터치 타겟, 색상 대비율 등 접근성 검사 | — |
| `design-system-checker` | 🔜 예정 | 디자인 시스템·토큰 준수 여부 | — |
| `interaction-checker` | 🔜 예정 | 인터랙티브 요소 상태 누락 여부 | — |

---

## 파일 구조

```
my-agent/
├── .claude/
│   ├── agents/
│   │   ├── prd-checker.md          # PRD 검수 에이전트 정의
│   │   └── ux-writing-checker.md   # UX 라이팅 검수 에이전트 정의
│   ├── commands/
│   │   ├── review-prd.md           # /review-prd 커맨드
│   │   ├── review-ux.md            # /review-ux 커맨드
│   │   └── review-all.md           # /review-all 커맨드 (순차 실행)
│   └── rules/
│       └── ux-writing.md           # UX 라이팅 규칙 (직접 작성 필요)
├── review-reports/                 # 검토 결과 저장 폴더
├── CLAUDE.md                       # Claude 에이전트 운영 지침
└── README.md                       # 이 파일
```

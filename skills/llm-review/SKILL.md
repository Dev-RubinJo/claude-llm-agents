---
name: llm-review
description: Gemini와 Codex를 병렬로 사용하여 코드를 교차 검토합니다. "교차 리뷰", "llm-review", "Gemini와 Codex로 리뷰해줘", "두 AI로 리뷰해줘" 등에 트리거됩니다.
---

# LLM Cross Review

Gemini Pro(넓은 시각)와 Codex xhigh(좁은 시각)를 병렬로 실행하여 코드를 교차 검토합니다.

## 교차 리뷰 설계

| 에이전트 | 관점 | 집중 영역 |
|---------|------|----------|
| **gemini-pro** | 넓은 시각 | 아키텍처 일관성, 패턴 위반, 의존성 영향, 설계 원칙 |
| **codex-xhigh** | 좁은 시각 | 구체적 버그, 타입 에러, 보안 취약점, 성능 이슈 |

## 실행 워크플로우

### Step 1: diff 추출

```bash
# PR 브랜치 대비 diff
if [ -n "$ARGUMENTS" ]; then
  DIFF=$(git diff "$ARGUMENTS"...HEAD 2>/dev/null || git diff HEAD~1 2>/dev/null)
else
  DIFF=$(git diff --staged 2>/dev/null)
  if [ -z "$DIFF" ]; then
    DIFF=$(git diff HEAD~1 2>/dev/null)
  fi
fi

echo "$DIFF" > /tmp/llm-review-diff-$(date +%s).txt
```

### Step 2: 병렬 실행

두 에이전트를 동시에 실행합니다 (독립적으로 실행 가능):

**gemini-pro 실행 (아키텍처/패턴 관점):**
```bash
TIMESTAMP=$(date +%s)
GEMINI_OUTPUT="/tmp/llm-review-gemini-${TIMESTAMP}.txt"
gemini -m gemini-3.0-pro --output-format text \
  "다음 코드 변경사항을 아키텍처 관점에서 리뷰해줘:
  - 아키텍처 일관성: 기존 패턴과 맞는가?
  - 의존성 영향: 다른 모듈에 미치는 영향은?
  - 설계 원칙: SOLID, DRY, YAGNI 위반 여부
  - 개선 권장사항

  변경사항:
  $(cat "$DIFF_FILE" | head -500)" \
  2>/dev/null > "$GEMINI_OUTPUT" &
GEMINI_PID=$!
```

**codex-xhigh 실행 (버그/보안 관점):**
```bash
CODEX_OUTPUT="/tmp/llm-review-codex-${TIMESTAMP}.txt"
codex exec review --uncommitted \
  2>/dev/null > "$CODEX_OUTPUT" &
CODEX_PID=$!
```

```bash
# 결과 수집
wait $GEMINI_PID 2>/dev/null || true
wait $CODEX_PID 2>/dev/null || true
```

### Step 3: 결과 종합

```bash
GEMINI_RESULT=$(cat "$GEMINI_OUTPUT" 2>/dev/null || echo "GEMINI_FAILED")
CODEX_RESULT=$(cat "$CODEX_OUTPUT" 2>/dev/null || echo "CODEX_FAILED")
```

결과 신뢰도 판단:

```
둘 다 같은 이슈 지적 → [높은 신뢰도] - 반드시 수정
하나만 지적 → [단독 지적] - 검토 후 판단
하나 실패 → 나머지 결과만 활용하여 계속 진행
```

## 전체 실행 스크립트

```bash
#!/bin/bash
set -e

# 의존성 확인
command -v gemini >/dev/null 2>&1 || { echo "ERROR: gemini CLI not found. Install: npm install -g @google/gemini-cli"; exit 1; }
command -v codex >/dev/null 2>&1 || { echo "ERROR: codex CLI not found. Install: npm install -g @openai/codex"; exit 1; }

TIMESTAMP=$(date +%s)
DIFF_FILE="/tmp/llm-review-diff-${TIMESTAMP}.txt"
GEMINI_OUTPUT="/tmp/llm-review-gemini-${TIMESTAMP}.txt"
CODEX_OUTPUT="/tmp/llm-review-codex-${TIMESTAMP}.txt"

# diff 추출
BASE=${1:-"HEAD~1"}
git diff "${BASE}" > "$DIFF_FILE"

if [ ! -s "$DIFF_FILE" ]; then
  git diff --staged > "$DIFF_FILE"
fi

if [ ! -s "$DIFF_FILE" ]; then
  echo "리뷰할 변경사항이 없습니다."
  exit 0
fi

echo "병렬 리뷰 시작..."

# gemini-pro: 아키텍처/패턴 관점
gemini -m gemini-3.0-pro --output-format text \
  "코드 변경사항을 아키텍처 관점에서 리뷰해줘. 아키텍처 일관성, 설계 원칙 위반, 의존성 영향, 개선 권장사항을 알려줘.

변경사항:
$(head -500 "$DIFF_FILE")" \
  2>/dev/null > "$GEMINI_OUTPUT" &
GEMINI_PID=$!

# codex-xhigh: 버그/보안 관점
codex exec review --uncommitted \
  2>/dev/null > "$CODEX_OUTPUT" &
CODEX_PID=$!

wait $GEMINI_PID 2>/dev/null || echo "Gemini 리뷰 완료 (또는 실패)"
wait $CODEX_PID 2>/dev/null || echo "Codex 리뷰 완료 (또는 실패)"

# 결과 출력
echo ""
echo "═══════════════════════════════════════"
echo "  GEMINI PRO 리뷰 (아키텍처/패턴)"
echo "═══════════════════════════════════════"
cat "$GEMINI_OUTPUT" 2>/dev/null || echo "[Gemini 리뷰 실패 - Codex 결과만 활용]"

echo ""
echo "═══════════════════════════════════════"
echo "  CODEX XHIGH 리뷰 (버그/보안)"
echo "═══════════════════════════════════════"
cat "$CODEX_OUTPUT" 2>/dev/null || echo "[Codex 리뷰 실패 - Gemini 결과만 활용]"
```

## 신뢰도 판단 기준

교차 리뷰 결과를 분석하여 Claude(오케스트레이터)가 우선순위를 결정합니다:

```
[높은 신뢰도] - 두 에이전트 모두 지적 → 즉시 수정 필요
[단독 지적]   - 한 에이전트만 지적 → 맥락 검토 후 판단
[정보성]      - 개선 제안 사항 → 선택적 적용
```

## 에러 핸들링

- **하나의 에이전트 실패**: 나머지 에이전트 결과를 활용하여 계속 진행
- **두 에이전트 모두 실패**: 각 CLI 설치/인증 상태를 확인하고 재시도
- **diff가 없음**: staged 변경사항 또는 HEAD~1 대비 diff 사용

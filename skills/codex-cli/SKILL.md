---
name: codex-cli
description: Codex CLI를 사용하여 코드를 구현하고 수정합니다. "codex로 수정해", "codex로 테스트 만들어줘", "codex로 리팩토링해", "codex로 구현해줘" 등에 트리거됩니다.
argument-hint: "[task-description]"
---

# Codex CLI Executor

Codex CLI를 활용하여 코드를 구현하고 수정합니다. `$ARGUMENTS`로 작업 설명을 받습니다.

## CRITICAL: `codex exec` 필수 사용

```bash
# 올바른 사용 (headless 실행)
codex exec -m codex-pro -s workspace-write "작업 설명"

# 잘못된 사용 (interactive 모드 진입 → 에러)
codex "작업 설명"
```

**`codex exec`을 항상 사용해야 합니다.** `codex` 단독 실행은 interactive 모드로 진입하여 Claude Code 환경에서 에러가 발생합니다.

## reasoning effort 선택 가이드

| 작업 유형 | reasoning_effort | 에이전트 |
|-----------|-----------------|---------|
| 대규모 리팩토링, 보안 감사, 복잡한 버그 | xhigh | codex-xhigh |
| 기능 구현, 테스트 작성, 일반 버그 수정 | high | codex-high |
| 보일러플레이트, CRUD, 포맷팅, 단순 변환 | medium | codex-medium |

## sandbox 모드 가이드

| 모드 | 사용 시점 |
|------|----------|
| `read-only` | 분석만 할 때, 파일 수정 전 계획 수립 |
| `workspace-write` | 실제 코드 수정 및 파일 생성 시 |

## 실행 워크플로우

### 1. 작업 결정

`$ARGUMENTS`가 있으면 해당 작업을 수행합니다.

### 2. 복잡도 판단 → 에이전트 선택

```
보일러플레이트/단순 변환 → codex-medium (빠른 처리)
기능 구현/테스트/일반 버그 → codex-high (주력)
대규모 리팩토링/보안/복잡한 버그 → codex-xhigh (최고 품질)
```

### 3. 실행 패턴

**기능 구현:**
```bash
TIMESTAMP=$(date +%s)
OUTPUT_FILE="/tmp/codex-executor-${TIMESTAMP}.txt"
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=high \
  "사용자 프로필 업데이트 PATCH /users/:id API를 구현해줘. 기존 패턴을 따르고 유효성 검사를 포함해." \
  2>/dev/null > "$OUTPUT_FILE"
cat "$OUTPUT_FILE"
```

**테스트 작성:**
```bash
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=high \
  "UserService의 모든 public 메서드에 대한 단위 테스트를 작성해줘. 기존 테스트 패턴을 따라줘." \
  2>/dev/null
```

**코드 리뷰 (커밋되지 않은 변경사항):**
```bash
codex exec review --uncommitted 2>/dev/null
```

**코드 리뷰 (브랜치 대비):**
```bash
codex exec review --base main "보안 이슈와 타입 에러에 집중해서 리뷰해줘. Do NOT modify any files." 2>/dev/null
```

**보일러플레이트 생성:**
```bash
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=medium \
  "Product 모델에 대한 기본 CRUD API 엔드포인트를 생성해줘." \
  2>/dev/null
```

## 유즈케이스 예시

### 복잡한 버그 수정 (xhigh)
```bash
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=xhigh \
  "로그인 시 간헐적으로 발생하는 race condition을 추적하고 수정해줘." 2>/dev/null
```

### 일반 기능 구현 (high)
```bash
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=high \
  "이메일 알림 서비스를 구현해줘. 기존 notification/ 모듈 패턴을 따라줘." 2>/dev/null
```

### 보일러플레이트 (medium)
```bash
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=medium \
  "Category, Tag 모델에 대한 기본 CRUD를 각각 생성해줘." 2>/dev/null
```

### 분석 후 수정 (2단계)
```bash
# 1단계: 분석만
codex exec -m codex-pro -s read-only -c model_reasoning_effort=high \
  "리팩토링 계획을 제안해줘. 파일은 수정하지 마." 2>/dev/null

# 2단계: 승인 후 수정
codex exec -m codex-pro -s workspace-write -c model_reasoning_effort=high \
  "위 계획대로 리팩토링을 실행해줘." 2>/dev/null
```

## 에러 핸들링

| 에러 | 해결 방법 |
|------|-----------|
| `command not found: codex` | `npm install -g @openai/codex` |
| 인증 오류 | `export OPENAI_API_KEY=sk-...` 설정 |
| sandbox 권한 오류 | `-s workspace-write` 플래그 추가 확인 |
| interactive 모드 진입 | `codex exec` 사용 필수, `codex` 단독 사용 금지 |

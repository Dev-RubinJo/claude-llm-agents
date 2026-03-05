---
name: gemini-cli
description: Gemini CLI를 사용하여 코드베이스를 분석하고 탐색합니다. "gemini로 분석해봐", "코드베이스 전체 구조 알려줘", "이거 구현돼있는지 찾아봐", "scout" 등에 트리거됩니다.
argument-hint: "[analysis-prompt]"
---

# Gemini CLI Scout

Gemini CLI를 활용하여 코드베이스를 분석하고 탐색합니다. `$ARGUMENTS`로 분석 프롬프트를 받습니다.

## 모델 선택 가이드

| 작업 유형 | 추천 모델 | 에이전트 |
|-----------|-----------|---------|
| 전체 아키텍처 분석, 보안 감사 | gemini-2.5-pro | gemini-pro |
| 기능 존재 여부, 모듈 요약, 컨벤션 파악 | gemini-2.5-flash | gemini-flash |
| YES/NO 판단, 단순 존재 확인, 한 줄 요약 | gemini-2.5-flash-lite | gemini-flash-lite |

### Auto 모드 (불확실할 때)

```bash
gemini -m gemini-2.5-flash --output-format text "프롬프트"
```

Flash가 충분한 경우가 많습니다. 아키텍처 수준 분석만 Pro를 사용하세요.

## 실행 워크플로우

### 1. 프롬프트 결정

`$ARGUMENTS`가 있으면 해당 내용을 분석 프롬프트로 사용합니다.
없으면 전체 아키텍처 분석을 기본으로 실행합니다:

```
프로젝트의 전체 아키텍처를 분석해줘: 디렉토리 구조, 주요 모듈, 레이어 관계, 핵심 패턴을 요약해줘.
```

### 2. 복잡도 판단 → 에이전트 선택

```
단순 확인 (TODO 있는지, 함수 존재 여부) → gemini-flash-lite
일반 탐색 (모듈 요약, 컨벤션 파악) → gemini-flash
깊은 분석 (아키텍처, 보안, 의존성 분석) → gemini-pro
```

### 3. 실행 패턴

**단일 파일 분석:**
```bash
gemini -m gemini-2.5-flash --output-format text "@src/auth/auth.service.ts 이 파일이 하는 일을 설명해줘"
```

**디렉토리 전체 분석:**
```bash
gemini -m gemini-2.5-pro --output-format text "@src/ 전체 아키텍처를 분석해줘"
```

**전체 파일 포함:**
```bash
gemini -m gemini-2.5-pro --all-files --output-format text "의존성 순환 참조가 있는지 확인해줘"
```

**결과 파일 저장:**
```bash
TIMESTAMP=$(date +%s)
gemini -m gemini-2.5-flash --output-format text "프롬프트" 2>/dev/null > "/tmp/gemini-scout-${TIMESTAMP}.txt"
cat "/tmp/gemini-scout-${TIMESTAMP}.txt"
```

## 유즈케이스 예시

### 아키텍처 분석
```bash
gemini -m gemini-2.5-pro --output-format text "@src/ 이 프로젝트의 레이어 구조와 모듈 간 의존성을 분석해줘. 순환 참조가 있는지도 확인해줘." 2>/dev/null
```

### 기능 구현 여부 확인
```bash
gemini -m gemini-2.5-flash --output-format text "@src/ 이메일 유효성 검사가 구현되어 있는지 확인하고, 있다면 어디에 어떻게 구현되어 있는지 알려줘." 2>/dev/null
```

### 컨벤션 파악
```bash
gemini -m gemini-2.5-flash --output-format text "@src/ 이 프로젝트의 에러 핸들링 패턴, 네이밍 컨벤션, 파일 구조 컨벤션을 요약해줘." 2>/dev/null
```

### 빠른 YES/NO 확인
```bash
gemini -m gemini-2.5-flash-lite --output-format text "@src/ 프로젝트에 TypeScript strict mode가 활성화되어 있나요? YES/NO" 2>/dev/null
```

## 에러 핸들링

| 에러 | 해결 방법 |
|------|-----------|
| `command not found: gemini` | `npm install -g @google/gemini-cli` |
| 인증 오류 | 터미널에서 `gemini` 실행하여 Google 계정 로그인 |
| 레이트 리밋 | 30초 대기 후 재시도 또는 다른 모델 사용 |
| 컨텍스트 초과 | 분석 범위를 좁히거나 파일별로 분리 실행 |

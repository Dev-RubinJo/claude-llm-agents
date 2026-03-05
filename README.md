# llm-agents

Claude Code 내에서 Gemini CLI와 Codex CLI를 모델별 특화 서브에이전트로 활용하기 위한 플러그인.

## 설계 원칙

| 역할 | 모델 | 특징 |
|------|------|------|
| **오케스트레이터** | Claude | 계획, 종합, 판단 |
| **정찰/분석** | Gemini | 1M 토큰 컨텍스트, 읽기 전용 |
| **실행/생성** | Codex | 샌드박스 코드 수정 |

## 에이전트

### Gemini 에이전트 (분석 전문)

| 에이전트 | 모델 | 사용 시점 |
|---------|------|----------|
| `gemini-pro` | gemini-2.5-pro | 아키텍처 분석, 보안 감사, 복잡한 의존성 분석 |
| `gemini-flash` | gemini-2.5-flash | 기능 확인, 모듈 요약, 컨벤션 파악 |
| `gemini-flash-lite` | gemini-2.5-flash-lite | YES/NO 판단, 단순 확인, 한 줄 요약 |

### Codex 에이전트 (실행 전문)

| 에이전트 | reasoning | 사용 시점 |
|---------|-----------|----------|
| `codex-xhigh` | xhigh | 대규모 리팩토링, 보안 패치, 복잡한 버그 |
| `codex-high` | high | 기능 구현, 테스트 작성, 일반 버그 수정 |
| `codex-medium` | medium | 보일러플레이트, CRUD, 포맷팅, 단순 변환 |

## 슬래시 커맨드

| 커맨드 | 스킬 | 설명 |
|--------|------|------|
| `/gemini-cli [prompt]` | gemini-cli | Gemini로 코드베이스 분석/탐색 |
| `/codex-cli [task]` | codex-cli | Codex로 코드 구현/수정 |
| `/llm-review [branch]` | llm-review | Gemini + Codex 교차 리뷰 |

## 추천 워크플로우

### 1. 신규 기능 개발

```
1. /gemini-cli "관련 모듈과 기존 패턴 파악해줘"
2. codex-high로 구현
3. /llm-review로 교차 리뷰
```

### 2. 복잡한 버그 수정

```
1. gemini-pro에게 버그 관련 코드 전체 분석 위임
2. 원인 파악 후 codex-xhigh로 정밀 수정
3. codex-xhigh review로 최종 검증
```

### 3. 일상적인 리팩토링

```
1. gemini-flash로 리팩토링 대상 모듈 요약 확인
2. codex-medium으로 빠른 정리
3. /llm-review로 간단한 교차 확인
```

## 사전 준비

### Gemini CLI 설치

```bash
npm install -g @google/gemini-cli
gemini  # Google 계정 인증
```

### Codex CLI 설치

```bash
npm install -g @openai/codex
export OPENAI_API_KEY=sk-...  # ~/.zshrc 또는 ~/.bashrc에 추가
```

### 설치 확인

```bash
command -v gemini && echo "Gemini OK"
command -v codex && echo "Codex OK"
```

## 플러그인 설치

### GitHub에서 직접 설치

```bash
# Claude Code에서
/install-plugin https://github.com/YOUR_USERNAME/llm-agents
```

### 로컬 개발 중 설치

```bash
# Claude Code에서
/install-plugin /path/to/llm-agents
```

## 개발

```bash
git clone https://github.com/YOUR_USERNAME/llm-agents
cd llm-agents
# 에이전트/스킬 수정 후 Claude Code 재시작
```

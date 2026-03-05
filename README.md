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

## 플러그인 설치

### 사전 준비

플러그인을 설치하기 전에 Gemini CLI와 Codex CLI가 설치되어 있어야 합니다.

#### Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini  # Google 계정 인증 (최초 1회)
```

#### Codex CLI

```bash
npm install -g @openai/codex
export OPENAI_API_KEY=sk-...  # ~/.zshrc 또는 ~/.bashrc에 추가 후 source
```

#### 설치 확인

```bash
command -v gemini && echo "Gemini OK"
command -v codex && echo "Codex OK"
```

---

### Claude Code 플러그인으로 설치

#### 방법 1: Claude Code UI에서 설치 (권장)

1. Claude Code 실행
2. 좌측 하단 **설정(⚙️)** 클릭
3. **Plugins** 탭 선택
4. **Install Plugin** 버튼 클릭
5. 아래 URL 입력:
   ```
   https://github.com/Dev-RubinJo/claude-llm-agents
   ```
6. **Install** 확인 → Claude Code 재시작

#### 방법 2: 슬래시 커맨드로 설치

Claude Code 채팅창에서 직접 입력:

```
/install-plugin https://github.com/Dev-RubinJo/claude-llm-agents
```

#### 방법 3: 로컬 경로로 설치 (개발/수정 시)

레포를 클론한 후 로컬 경로로 설치:

```bash
git clone https://github.com/Dev-RubinJo/claude-llm-agents
```

```
/install-plugin /path/to/claude-llm-agents
```

---

### 설치 확인

설치 후 Claude Code를 재시작하면 다음이 활성화됩니다:

**에이전트** (Task 도구에서 사용 가능):
```
claude-llm-agents:gemini-pro
claude-llm-agents:gemini-flash
claude-llm-agents:gemini-flash-lite
claude-llm-agents:codex-xhigh
claude-llm-agents:codex-high
claude-llm-agents:codex-medium
```

**슬래시 커맨드** (채팅창에서 `/` 입력 시 목록 표시):
```
/gemini-cli
/codex-cli
/llm-review
```

---

## 개발 (수정 시)

```bash
git clone https://github.com/Dev-RubinJo/claude-llm-agents
cd claude-llm-agents
# agents/ 또는 skills/ 파일 수정 후 Claude Code 재시작
```

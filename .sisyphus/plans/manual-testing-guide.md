# Copilot Multi-Agent Orchestration System — Manual Testing Guide

## TL;DR

> **목적**: `copilot-my-orchestra` 프로젝트의 6개 에이전트(.agent.md)가 VS Code Insiders에서 정의된 역할대로 정확하게 동작하는지 수동으로 검증하는 상세 매뉴얼
> 
> **산출물**: 
> - 사전 준비(Pre-Flight) 체크리스트
> - 에이전트별 개별 행동 검증 테스트 (30+ 케이스)
> - 파이프라인 통합 테스트 (10+ 케이스)
> - 알려진 플랫폼 이슈 레퍼런스
> - Pass/Fail 판정 기준 및 결과 기록 템플릿
>
> **예상 소요 시간**: 약 4~6시간 (전체 실행 기준)
> **환경**: VS Code Insiders + GitHub Copilot Chat
> **테스트 대상**: `.github/agents/` 내 6개 영문 에이전트 파일

---

## Context

### 프로젝트 개요
`copilot-my-orchestra`는 GitHub Copilot을 위한 멀티 에이전트 오케스트레이션 시스템입니다. 6개의 전문화된 에이전트가 5단계 파이프라인(Research → Plan → Implement → Test → Review)을 통해 협업합니다.

### 에이전트 구성
| 에이전트 | 파일 | 역할 | 도구 | 수정 권한 |
|---------|------|------|------|----------|
| **Orchestrator** | `orchestrator.agent.md` | 파이프라인 조율, 의도 분류, 서브에이전트 위임 | agent, search, read, fetch | Read-Write (위임만) |
| **Researcher** | `researcher.agent.md` | 코드베이스/문서 조사, 컨텍스트 수집 | search, read, fetch, web | **Read-Only** |
| **Planner** | `planner.agent.md` | 구현 계획 수립, 기술 명세 작성 | search, read, fetch | **Read-Only** |
| **Implementer** | `implementer.agent.md` | 코드 작성, 리팩터링, 기능 구현 | editFiles, search, read, execute, fetch | Read-Write |
| **Tester** | `tester.agent.md` | 테스트 생성 및 실행, 품질 검증 | editFiles, search, read, execute | Read-Write |
| **Reviewer** | `reviewer.agent.md` | 코드 리뷰, 품질 점검, 최종 검토 | search, read | **Read-Only** |

### 실행 모드
1. **Autonomous Mode**: Orchestrator가 서브에이전트를 자동 호출 (VS Code Insiders에서만 완전 지원)
2. **Handoff Mode**: 사용자가 버튼을 클릭하여 단계별 진행 제어
3. **CLI Standalone Mode**: 수동으로 각 에이전트를 순서대로 호출

### 핵심 설계 원칙
- **동적 기술 스택 탐지(Dynamic Technology Discovery)**: 에이전트가 config 파일을 분석하여 프로젝트 스택을 자동 감지
- **기술 불가지론(Technology-Agnostic)**: 특정 기술 스택을 가정하지 않음
- **Read-Only 강제**: Researcher, Planner, Reviewer는 파일을 절대 수정하지 않음

---

## Known VS Code Insiders Platform Issues

> ⚠️ **중요**: 아래 이슈들은 VS Code 플랫폼 버그이므로, 에이전트 설정 오류와 구분해야 합니다.
> 테스트 중 이 이슈에 해당하는 실패가 발생하면 `KNOWN_BUG`으로 표기하세요.

| 이슈 | VS Code Bug # | 영향 | 테스트 영향 |
|------|--------------|------|------------|
| 서브에이전트 tool 상속 깨짐 | [#284431](https://github.com/microsoft/vscode/issues/284431), [#284304](https://github.com/microsoft/vscode/issues/284304) | 서브에이전트가 `agent` 도구를 상실 — 중첩 위임 실패 가능 | Autonomous Mode 테스트 시 주의 |
| 도구 접근 불일치 | [#284093](https://github.com/microsoft/vscode/issues/284093) | Orchestrator가 위임 에이전트의 도구를 항상 존중하지 않음 | 도구 사용 검증 테스트 시 주의 |
| 서브에이전트 model 필드 무시 | [#291883](https://github.com/microsoft/vscode/issues/291883) | 서브에이전트가 부모 모델을 상속할 수 있음 | 직접 제어 불가 |
| 서브에이전트 취소 실패 | [#293783](https://github.com/microsoft/vscode/issues/293783) | "Error invoking subagent: Canceled" — 프리미엄 요청 소모 | 자율 모드 테스트 시 오류 발생 가능 |
| Plan mode 도구 접근 누수 | [#295905](https://github.com/microsoft/vscode/issues/295905) | 동시 에이전트 세션 간 간섭 | **한 번에 하나의 세션만 사용** |
| 터미널 heredoc 실행 실패 | [#296037](https://github.com/microsoft/vscode/issues/296037) | `execute` 도구로 복잡한 스크립트 실행 불안정 | Tester 에이전트 실행 테스트 시 주의 |

**대응 원칙**: 
- 동시 에이전트 세션은 1개만 유지
- 프리미엄 요청 제한 주의 (시간당 5~6회 자율 파이프라인 권장)
- 알려진 버그에 해당하는 실패는 `KNOWN_BUG`으로 표기하고 별도 기록

---

## Pre-Flight Diagnostics (사전 준비)

### 환경 체크리스트

| # | 항목 | 확인 방법 | 기대 결과 | Pass/Fail |
|---|------|----------|----------|-----------|
| PF-1 | VS Code Insiders 설치 확인 | Help > About | VS Code Insiders 최신 버전 | ☐ |
| PF-2 | GitHub Copilot 확장 설치 | Extensions 패널에서 검색 | GitHub Copilot + GitHub Copilot Chat 설치됨 | ☐ |
| PF-3 | GitHub Copilot 로그인 상태 | 하단 상태바 Copilot 아이콘 | 로그인 상태, 정상 동작 | ☐ |
| PF-4 | Chat Debug Panel 접근 | `Ctrl+Shift+P` → "GitHub Copilot: Show Chat Debug View" | 디버그 패널 열림 | ☐ |
| PF-5 | `.github/agents/` 배치 | 테스트 프로젝트에 6개 `.agent.md` 파일 복사 | 6개 파일 존재 확인 | ☐ |
| PF-6 | `copilot-instructions.md` 배치 | `.github/copilot-instructions.md` 복사 | 파일 존재 확인 | ☐ |
| PF-7 | 에이전트 인식 확인 | Copilot Chat에서 `@` 입력 | Orchestrator 포함 6개 에이전트 목록 표시 | ☐ |
| PF-8 | Smoke Test | `@Orchestrator hello, are you ready?` 입력 | Orchestrator로서 응답 (역할 설명 또는 파이프라인 언급) | ☐ |
| PF-9 | 테스트 프로젝트 기술 스택 파일 확인 | 프로젝트 루트에 `package.json`/`pyproject.toml` 등 존재 | 최소 1개 이상의 기술 스택 감지 파일 존재 | ☐ |
| PF-10 | Git 상태 확인 | 테스트 프로젝트에서 `git status` | Clean working tree (변경 없음) — 테스트 전 기준점 | ☐ |

### 도구 준비

- **화면 기록 도구** (권장): Handoff 버튼 동작 등 UI 행동 기록용
- **Output 패널**: `View > Output` → "GitHub Copilot" 채널 선택하여 도구 에러 로그 확인용
- **터미널**: `git diff` 실행용 (Implementer/Tester 후 변경 사항 확인)

---

## 테스트 결과 기록 템플릿

> 각 테스트 케이스 실행 후 아래 형식으로 기록하세요.

```markdown
### [테스트 ID] — [테스트명]
- **실행 일시**: YYYY-MM-DD HH:MM
- **VS Code 버전**: 
- **Copilot 확장 버전**: 
- **결과**: PASS / FAIL / KNOWN_BUG / SKIP
- **실제 응답 요약**: 
- **비고** (FAIL인 경우 상세): 
- **스크린샷/화면 기록**: (파일 경로 또는 링크)
```

---

## Test Section 1: Orchestrator 에이전트 검증

### 1.1 의도 분류 (Intent Classification)

> Orchestrator의 핵심 기능은 요청을 5가지 카테고리로 분류하고 적절한 파이프라인을 선택하는 것입니다.

---

- [ ] TC-ORC-001: 새 기능 요청 의도 분류

  **입력 프롬프트**:
  ```
  @Orchestrator Add a user authentication system with JWT tokens to this project
  ```

  **기대 결과**:
  - "New feature" 또는 "새 기능"으로 분류했음을 **명시적으로** 설명
  - Full pipeline 선택: Research → Plan → Implement → Test → Review
  - 선택 이유를 설명

  **Pass 기준**:
  - [ ] 카테고리를 명시적으로 언급 (예: "This is a new feature request")
  - [ ] 5단계 전체 파이프라인을 언급하거나 시작
  - [ ] 분류 이유를 설명

  **Fail 기준**:
  - 카테고리 언급 없이 바로 코드 작성 시작
  - 잘못된 카테고리로 분류 (예: "quick fix"로 분류)
  - 파이프라인 단계 없이 단일 작업 수행

---

- [ ] TC-ORC-002: 버그 수정 의도 분류

  **입력 프롬프트**:
  ```
  @Orchestrator The login endpoint returns 500 error when email contains special characters. Fix this bug.
  ```

  **기대 결과**:
  - "Bug fix"로 분류
  - Abbreviated pipeline: Research → Implement → Test → Review (Plan 단계 생략)
  - 선택 이유를 설명

  **Pass 기준**:
  - [ ] "Bug fix"로 분류 언급
  - [ ] 축약된 파이프라인 사용 (Plan 단계 생략)
  - [ ] 분류 이유 설명

  **Fail 기준**:
  - "New feature"로 잘못 분류
  - Full pipeline 사용 (Plan 포함)

---

- [ ] TC-ORC-003: 리팩터링 의도 분류

  **입력 프롬프트**:
  ```
  @Orchestrator Refactor the database connection module to use connection pooling instead of creating new connections each time
  ```

  **기대 결과**:
  - "Refactoring"으로 분류
  - Structured pipeline: Research → Plan → Implement → Test → Review
  - 선택 이유를 설명

  **Pass 기준**:
  - [ ] "Refactoring"으로 분류 언급
  - [ ] 전체 파이프라인 사용 (리팩터링은 계획 필요)
  - [ ] 분류 이유 설명

---

- [ ] TC-ORC-004: 질문/조사 의도 분류

  **입력 프롬프트**:
  ```
  @Orchestrator How is error handling currently implemented in this project? What patterns are used?
  ```

  **기대 결과**:
  - "Question" 또는 "Investigation"으로 분류
  - Research only — Researcher만 실행
  - 코드 변경 없이 조사 결과만 보고

  **Pass 기준**:
  - [ ] "Question/Investigation"으로 분류 언급
  - [ ] Research 단계만 실행 (Implement/Test 없음)
  - [ ] 조사 결과를 구조적으로 보고

---

- [ ] TC-ORC-005: Quick Fix 의도 분류

  **입력 프롬프트**:
  ```
  @Orchestrator Fix the typo in the error message on line 42 of src/utils.js — "occured" should be "occurred"
  ```

  **기대 결과**:
  - "Quick fix"로 분류
  - Implement → Test만 실행 (Research/Plan 생략)

  **Pass 기준**:
  - [ ] "Quick fix"로 분류 언급
  - [ ] 축약된 파이프라인 (Implement → Test만)
  - [ ] 분류 이유 설명

---

### 1.2 Dynamic Technology Discovery

- [ ] TC-ORC-006: 기술 스택 자동 감지

  **전제 조건**: 테스트 프로젝트에 `package.json` 존재 (Node.js 프로젝트)

  **입력 프롬프트**:
  ```
  @Orchestrator Analyze this project and tell me what technology stack is in use
  ```

  **기대 결과**:
  - `package.json`, `tsconfig.json` 등 설정 파일을 분석
  - 프레임워크, 빌드 도구, 테스트 프레임워크 등을 감지하여 보고
  - 감지 결과를 명시적으로 나열

  **Pass 기준**:
  - [ ] 설정 파일 분석을 언급 (예: "I found package.json...")
  - [ ] 주요 기술 스택을 정확하게 식별
  - [ ] 테스트 프레임워크/빌드 도구 감지

  **Fail 기준**:
  - 설정 파일 분석 없이 기술 스택을 추정
  - 잘못된 기술 스택 보고

---

### 1.3 서브에이전트 위임 (Autonomous Mode)

- [ ] TC-ORC-007: Researcher 서브에이전트 위임

  **입력 프롬프트**:
  ```
  @Orchestrator I want to add a caching layer to the API endpoints. Start by researching how this project currently handles data fetching.
  ```

  **기대 결과**:
  - Orchestrator가 의도를 분류한 후 Researcher를 호출
  - Chat Debug Panel에서 서브에이전트 호출 로그 확인
  - Researcher 결과를 Orchestrator가 요약

  **확인 방법**:
  1. Chat Debug Panel (`Ctrl+Shift+P` → "Show Chat Debug View") 열기
  2. 서브에이전트 호출 로그에서 `Researcher` 확인
  3. 응답에서 Research 결과 요약 확인

  **Pass 기준**:
  - [ ] Chat Debug Panel에서 Researcher 서브에이전트 호출 확인
  - [ ] 조사 결과가 구조적으로 정리됨
  - [ ] Orchestrator가 다음 단계(Plan)를 안내하거나 자동 진행

  **Fail 기준** (KNOWN_BUG 가능):
  - 서브에이전트 호출 없이 Orchestrator가 직접 조사 수행
  - "Error invoking subagent" 오류 (#293783)
  - 도구 접근 불일치 (#284093)

---

- [ ] TC-ORC-008: Full Autonomous Pipeline 실행

  **⚠️ 이 테스트는 프리미엄 요청을 다수 소모합니다. 시간 여유 확보 후 실행하세요.**

  **입력 프롬프트**:
  ```
  @Orchestrator Add a simple health check endpoint at GET /health that returns { status: "ok" }. Run the full pipeline autonomously.
  ```

  **기대 결과**:
  - Research → Plan → Implement → Test → Review 전체 자동 실행
  - 각 단계 완료 시 요약 제공
  - 최종 결과물: 코드 변경 + 테스트 + 리뷰

  **확인 방법**:
  1. Chat Debug Panel에서 서브에이전트 호출 순서 확인
  2. 각 단계별 결과 요약 확인
  3. `git diff`로 실제 코드 변경 확인

  **Pass 기준**:
  - [ ] 최소 3개 이상의 서브에이전트가 순차적으로 호출됨
  - [ ] 각 단계 결과가 다음 단계로 전달됨 (context forwarding)
  - [ ] 최종 코드 변경이 존재 (`git diff` 결과 비어있지 않음)
  - [ ] 테스트 실행 결과가 포함됨

  **Fail 기준** (KNOWN_BUG 가능):
  - 서브에이전트 체이닝 실패
  - 중간 단계에서 컨텍스트 손실
  - `git diff` 결과 빈 상태

  **테스트 후 정리**: `git checkout .` 으로 변경사항 롤백

---

### 1.4 Handoff Mode (사용자 제어)

- [ ] TC-ORC-009: Handoff 버튼 표시 확인

  **입력 프롬프트**:
  ```
  @Orchestrator I need to implement a rate limiter for the API. Let me control the workflow step by step.
  ```

  **기대 결과**:
  - Orchestrator가 의도를 분류한 후 응답
  - 응답 완료 후 Handoff 버튼들이 표시됨:
    - "Research Codebase"
    - "Create Implementation Plan"
    - "Implement Changes"
    - "Generate Tests"
    - "Review Code"

  **Pass 기준**:
  - [ ] Handoff 버튼이 응답 하단에 표시됨
  - [ ] 버튼이 파이프라인 순서대로 나열됨
  - [ ] 다음 단계로 가장 적절한 버튼을 추천

  **Fail 기준**:
  - 버튼 미표시
  - 바로 자율 모드로 실행

  **참고**: `send: false`이므로 버튼 클릭 시 사용자가 컨텍스트를 확인하고 직접 전송해야 함

---

- [ ] TC-ORC-010: Handoff 버튼 클릭 후 에이전트 전환

  **전제 조건**: TC-ORC-009에서 Handoff 버튼이 표시된 상태

  **동작**:
  1. "Research Codebase" 버튼 클릭
  2. 표시되는 프롬프트 확인 (send: false이므로 자동 전송 안 됨)
  3. Enter로 전송

  **기대 결과**:
  - Researcher 에이전트 세션으로 전환
  - Researcher가 조사 작업 수행
  - 조사 완료 후 "Create Plan from Research" Handoff 버튼 표시

  **Pass 기준**:
  - [ ] Researcher 에이전트로 전환됨
  - [ ] Researcher가 구조적 조사 결과 제공
  - [ ] 다음 Handoff 버튼(Planner) 표시

  **참고**: 화면 기록 권장

---

## Test Section 2: Researcher 에이전트 검증

### 2.1 Read-Only 강제 (Critical)

- [ ] TC-RES-001: 파일 수정 거부 확인

  **입력 프롬프트**:
  ```
  @Researcher Add a comment at the top of the main entry file explaining what it does
  ```

  **기대 결과**:
  - 파일 수정을 거부
  - Read-only 역할임을 설명
  - Implementer로의 핸드오프를 제안

  **확인 방법**:
  1. 응답 내용 확인
  2. `git diff` 실행하여 파일 변경 없음 확인

  **Pass 기준**:
  - [ ] 파일 수정을 명시적으로 거부
  - [ ] Read-only 역할임을 언급
  - [ ] Implementer 핸드오프 제안
  - [ ] `git diff` 결과 빈 상태

  **Fail 기준**:
  - 파일을 실제로 수정
  - 거부 없이 수정 시도
  - `git diff`에 변경사항 존재

---

### 2.2 기술 스택 탐지 (Technology Stack Discovery)

- [ ] TC-RES-002: 기술 스택 탐지 보고서

  **입력 프롬프트**:
  ```
  @Researcher Investigate the technology stack of this project. What languages, frameworks, and tools are in use?
  ```

  **기대 결과**:
  - 설정 파일(package.json, tsconfig.json 등)을 직접 분석
  - 구조화된 보고서 형식으로 출력
  - 최소한 다음 항목 포함:
    - Primary Language(s)
    - Framework(s)
    - Build Tools
    - Key Dependencies

  **Pass 기준**:
  - [ ] 설정 파일을 직접 읽고 분석 (Chat Debug에서 `read` 도구 사용 확인)
  - [ ] "Structured Research Report" 형식 (또는 유사한 구조)
  - [ ] Technology Stack 섹션이 정확함
  - [ ] 기술 스택을 "추정"이 아닌 "감지"로 보고

  **Fail 기준**:
  - 설정 파일 분석 없이 추정
  - 비구조적 자유 서술형 응답
  - 잘못된 기술 스택 보고

---

### 2.3 패턴 분석 (Pattern Discovery)

- [ ] TC-RES-003: 코드 패턴 분석

  **입력 프롬프트**:
  ```
  @Researcher Analyze the existing code patterns in this project. Focus on naming conventions, file organization, testing patterns, and error handling approaches.
  ```

  **기대 결과**:
  - 코드베이스를 검색하여 패턴 식별
  - Naming Conventions, File Organization, Testing Approach, Documentation Style 등 보고
  - 관련 파일 경로 언급

  **Pass 기준**:
  - [ ] `search` 또는 `search/codebase` 도구 사용 (Chat Debug 확인)
  - [ ] 실제 파일 경로와 함께 패턴 보고
  - [ ] Relevant Files and Locations 섹션 포함
  - [ ] Recommendations 섹션 포함

---

### 2.4 외부 리서치

- [ ] TC-RES-004: 외부 문서 조사

  **입력 프롬프트**:
  ```
  @Researcher Research best practices for implementing rate limiting in the detected framework. Check official documentation and GitHub examples.
  ```

  **기대 결과**:
  - `web` 또는 `fetch` 도구를 사용하여 외부 정보 수집
  - 공식 문서 또는 GitHub 예제 참조
  - 조사 결과를 Constraints and Considerations + Recommendations 형식으로 보고

  **Pass 기준**:
  - [ ] 외부 도구(web/fetch) 사용 (Chat Debug 확인)
  - [ ] 외부 소스 참조 명시
  - [ ] 프로젝트의 감지된 기술 스택에 맞춤화된 추천

---

### 2.5 핸드오프

- [ ] TC-RES-005: Planner 핸드오프 제안

  **전제 조건**: TC-RES-003 또는 TC-RES-004 완료 후

  **기대 결과**:
  - 조사 완료 후 Planner로의 핸드오프 제안
  - "Create Plan from Research" Handoff 버튼 표시

  **Pass 기준**:
  - [ ] Planner 핸드오프 언급 또는 버튼 표시
  - [ ] 조사 결과가 Planner에 전달 가능한 형태로 정리됨

---

## Test Section 3: Planner 에이전트 검증

### 3.1 Read-Only 강제

- [ ] TC-PLN-001: 파일 수정 거부 확인

  **입력 프롬프트**:
  ```
  @Planner Create a new configuration file for the caching layer
  ```

  **기대 결과**:
  - 파일 생성/수정을 거부
  - 대신 어떤 파일을 생성해야 하는지 "계획"으로 문서화
  - Implementer 핸드오프 제안

  **Pass 기준**:
  - [ ] 실제 파일 생성/수정 없음 (`git diff` 빈 상태)
  - [ ] 계획 문서로 파일 생성 내용을 설명
  - [ ] Read-only 역할 언급

---

### 3.2 구현 계획 생성

- [ ] TC-PLN-002: 구조화된 계획 생성

  **입력 프롬프트**:
  ```
  @Planner Create a detailed implementation plan for adding a caching layer to the API endpoints. The project uses [테스트 프로젝트 기술 스택].
  ```

  **기대 결과**:
  - 구조화된 Implementation Plan 형식의 출력
  - 필수 섹션 포함:
    - Objective
    - Prerequisites
    - Technology Context
    - Implementation Steps (각 단계별 Files, What to do, Why, Acceptance Criteria)
    - Testing Approach
    - Verification Steps
    - Risks and Mitigations

  **Pass 기준**:
  - [ ] "Implementation Plan" 형식 사용
  - [ ] Objective 섹션 존재
  - [ ] Technology Context가 동적 탐지 기반 (설정 파일 분석)
  - [ ] Implementation Steps가 3개 이상, 각각 Files/What/Why/Acceptance 포함
  - [ ] Testing Approach 섹션 존재
  - [ ] Risks and Mitigations 섹션 존재

  **Fail 기준**:
  - 비구조적 자유 서술형 계획
  - Technology Context 없이 기술 추정
  - Acceptance Criteria 누락

---

- [ ] TC-PLN-003: Dynamic Codebase Discovery

  **입력 프롬프트**:
  ```
  @Planner Plan the addition of user profile picture upload functionality
  ```

  **기대 결과**:
  - 계획 전에 프로젝트 기술 스택을 자동 감지
  - Technology Context 섹션에 감지 결과 명시
  - 감지된 스택에 맞는 도구/명령어 사용 (예: npm test, pytest 등)

  **Pass 기준**:
  - [ ] 설정 파일 분석을 먼저 수행 (Chat Debug에서 `read`/`search` 확인)
  - [ ] "Detected Stack" 또는 유사 표현으로 기술 스택 명시
  - [ ] 테스트 명령어가 감지된 프레임워크에 맞음 (하드코딩 아님)

---

- [ ] TC-PLN-004: 자립적(Self-Contained) 계획 검증

  **입력 프롬프트**: (TC-PLN-002의 응답 사용)

  **검증 포인트**:
  - 계획만 읽고 실행할 수 있는지 확인
  - "as discussed earlier" 같은 대화 참조 없음
  - 모든 단계가 독립적으로 검증 가능

  **Pass 기준**:
  - [ ] 대화 컨텍스트 참조 없음 (자립적)
  - [ ] 각 단계에 Acceptance Criteria 포함
  - [ ] 파일 경로가 구체적

---

### 3.3 핸드오프

- [ ] TC-PLN-005: Implementer 핸드오프

  **기대 결과**:
  - 계획 완료 후 "Start Implementation" Handoff 버튼 표시
  - Implementer에게 전달할 컨텍스트가 준비됨

  **Pass 기준**:
  - [ ] "Start Implementation" 핸드오프 버튼 표시
  - [ ] 계획이 Implementer가 바로 실행 가능한 수준

---

## Test Section 4: Implementer 에이전트 검증

### 4.1 패턴 준수

- [ ] TC-IMP-001: 기존 패턴 따르기

  **입력 프롬프트**:
  ```
  @Implementer Add a simple utility function that validates email addresses. Follow the existing code patterns and conventions in this project.
  ```

  **기대 결과**:
  - 기존 코드 패턴을 먼저 검색 (search 도구 사용)
  - 발견된 패턴과 일치하는 코드 스타일로 작성
  - 네이밍 규칙, 들여쓰기, 모듈 구조 일치

  **확인 방법**:
  1. Chat Debug Panel에서 `search`/`search/codebase` 도구 사용 확인
  2. `git diff`로 변경사항 확인
  3. 기존 코드와 스타일 비교

  **Pass 기준**:
  - [ ] 코딩 전 패턴 검색 수행 (Chat Debug 확인)
  - [ ] 기존 네이밍 규칙 준수 (camelCase vs snake_case 등)
  - [ ] 기존 파일 구조 패턴 준수
  - [ ] 최소한의 변경 (Smallest possible diff)

  **테스트 후 정리**: `git checkout .`

---

### 4.2 코드 품질 규칙

- [ ] TC-IMP-002: 금지 패턴 미사용 확인

  **입력 프롬프트**:
  ```
  @Implementer Implement a function that fetches user data from an external API and caches the result
  ```

  **확인 방법**: `git diff`로 생성된 코드 검사

  **Pass 기준**:
  - [ ] `any` 타입 미사용 (TypeScript인 경우)
  - [ ] 빈 catch 블록 없음
  - [ ] `console.log` 직접 사용 없음 (프로덕션 코드에서)
  - [ ] 주석 처리된 코드 없음
  - [ ] TODO/FIXME 등 미완성 마커 없음

  **테스트 후 정리**: `git checkout .`

---

### 4.3 최소 변경 원칙

- [ ] TC-IMP-003: 범위 외 변경 없음 확인

  **입력 프롬프트**:
  ```
  @Implementer Add input validation to the [specific function] in [specific file]. Only modify this one function.
  ```

  **확인 방법**: `git diff`로 변경 범위 확인

  **Pass 기준**:
  - [ ] 지정된 파일만 수정됨
  - [ ] 지정된 함수만 변경됨
  - [ ] 불필요한 리팩터링 없음
  - [ ] 기존 로직 보존

  **테스트 후 정리**: `git checkout .`

---

### 4.4 핸드오프

- [ ] TC-IMP-004: Tester 핸드오프

  **기대 결과**:
  - 구현 완료 후 변경 사항 요약 제공
  - "Generate Tests" Handoff 버튼 표시
  - 테스트 필요 사항 설명

  **Pass 기준**:
  - [ ] 변경 파일 목록 제공
  - [ ] "Generate Tests" 핸드오프 버튼 표시
  - [ ] 테스트가 필요한 영역 명시

---

## Test Section 5: Tester 에이전트 검증

### 5.1 Dynamic Test Discovery

- [ ] TC-TST-001: 테스트 프레임워크 자동 감지

  **전제 조건**: TC-IMP-001 또는 TC-IMP-002의 변경사항이 있는 상태

  **입력 프롬프트**:
  ```
  @Tester Generate tests for the recently added code changes. Discover the testing framework first.
  ```

  **기대 결과**:
  - Phase 1: 테스트 프레임워크 감지 (설정 파일 분석)
  - Phase 2: 기존 테스트 패턴 분석 (2-3개 테스트 파일 확인)
  - Phase 3: 감지된 패턴 복제하여 새 테스트 생성

  **Pass 기준**:
  - [ ] 테스트 프레임워크를 설정 파일에서 감지 (Chat Debug에서 `read` 확인)
  - [ ] 기존 테스트 파일을 분석 (Chat Debug에서 `search`/`read` 확인)
  - [ ] 감지된 프레임워크의 어서션 스타일 사용 (예: jest의 `expect(...).toBe(...)`)
  - [ ] 기존 테스트 네이밍 규칙 준수

  **Fail 기준**:
  - 프레임워크 하드코딩 (예: jest 가정 없이)
  - 기존 테스트 패턴 무시

  **테스트 후 정리**: `git checkout .`

---

### 5.2 테스트 커버리지

- [ ] TC-TST-002: 테스트 범위 완전성

  **입력 프롬프트**:
  ```
  @Tester Write comprehensive tests for [specific function/module]. Cover happy path, edge cases, and error handling.
  ```

  **기대 결과**:
  - Happy path 테스트
  - Edge case 테스트 (빈 값, null, 경계값)
  - Error case 테스트 (잘못된 입력, 예외 상황)

  **Pass 기준**:
  - [ ] Happy path 테스트 1개 이상
  - [ ] Edge case 테스트 1개 이상
  - [ ] Error handling 테스트 1개 이상
  - [ ] 테스트 이름이 명확하고 설명적

  **테스트 후 정리**: `git checkout .`

---

### 5.3 테스트 실행

- [ ] TC-TST-003: 테스트 실행 및 결과 보고

  **전제 조건**: TC-TST-001 또는 TC-TST-002에서 테스트 파일이 생성된 상태

  **입력 프롬프트**:
  ```
  @Tester Run the tests you just created and report the results
  ```

  **기대 결과**:
  - 감지된 테스트 명령어 실행 (예: `npm test`, `pytest`)
  - 결과 파싱: 통과/실패/스킵 수
  - 실패 시: 에러 메시지, 원인 분석, 수정 제안

  **Pass 기준**:
  - [ ] `execute` 도구로 테스트 실행 (Chat Debug 확인)
  - [ ] 통과/실패 수 요약
  - [ ] 실패 시 원인 분석 및 수정 제안

  **⚠️ KNOWN_BUG 가능**: 터미널 heredoc 실행 실패 (#296037)

  **테스트 후 정리**: `git checkout .`

---

### 5.4 핸드오프

- [ ] TC-TST-004: Reviewer 핸드오프

  **기대 결과**:
  - 테스트 완료 후 커버리지 요약 제공
  - "Review Changes" Handoff 버튼 표시

  **Pass 기준**:
  - [ ] 테스트 커버리지 요약
  - [ ] "Review Changes" 핸드오프 버튼 표시

---

## Test Section 6: Reviewer 에이전트 검증

### 6.1 Read-Only 강제

- [ ] TC-REV-001: 파일 수정 거부 확인

  **입력 프롬프트**:
  ```
  @Reviewer Fix the naming inconsistency in src/utils.js
  ```

  **기대 결과**:
  - 코드 수정을 거부
  - 피드백만 제공 (파일:라인 참조)
  - Implementer 핸드오프 제안

  **확인 방법**: `git diff` 확인

  **Pass 기준**:
  - [ ] 파일 수정 없음 (`git diff` 빈 상태)
  - [ ] Read-only 역할 언급
  - [ ] Implementer 핸드오프 제안 또는 "Request Changes" 버튼

---

### 6.2 6차원 리뷰

- [ ] TC-REV-002: 구조화된 리뷰 출력

  **전제 조건**: TC-IMP-001 또는 TC-IMP-002의 변경사항이 있는 상태

  **입력 프롬프트**:
  ```
  @Reviewer Review the recent code changes in this project for correctness, quality, and security
  ```

  **기대 결과**:
  - 6차원 리뷰 형식:
    1. Correctness (✅/❌)
    2. Patterns & Conventions (✅/❌)
    3. Security (✅/❌)
    4. Quality & Maintainability (✅/❌)
    5. Testing (✅/❌)
    6. Scope & Requirements (✅/❌)
  - Verdict: Approve / Request Changes / Needs Discussion
  - Action Items 목록

  **Pass 기준**:
  - [ ] 6개 차원 모두 포함 (각각 ✅ 또는 ❌)
  - [ ] 파일:라인 참조가 구체적 (예: `auth.js:42`)
  - [ ] Verdict 명시
  - [ ] Action Items 번호 매김

  **Fail 기준**:
  - "looks good" 같은 모호한 피드백
  - 파일:라인 참조 없음
  - 6개 차원 중 누락

---

### 6.3 Dynamic Discovery

- [ ] TC-REV-003: 프로젝트 품질 기준 감지

  **입력 프롬프트**:
  ```
  @Reviewer Before reviewing, discover the project's quality standards, linting rules, and coding conventions
  ```

  **기대 결과**:
  - lint/formatter 설정 파일 분석 (.eslintrc, .prettierrc 등)
  - CONTRIBUTING.md, 스타일 가이드 확인
  - CI/CD 설정 확인
  - 감지된 기준을 명시적으로 보고

  **Pass 기준**:
  - [ ] 설정 파일 분석 수행 (Chat Debug에서 `read`/`search` 확인)
  - [ ] 감지된 품질 기준을 나열
  - [ ] 리뷰 기준을 프로젝트에 맞춤화

---

### 6.4 핸드오프

- [ ] TC-REV-004: Implementer / Orchestrator 핸드오프

  **기대 결과**:
  - 변경 필요 시: "Request Changes" 버튼 → Implementer
  - 승인 또는 논의 필요 시: "Back to Orchestrator" 버튼

  **Pass 기준**:
  - [ ] 적절한 핸드오프 버튼 표시 (최소 1개)
  - [ ] Verdict에 따른 올바른 핸드오프 방향

---

## Test Section 7: Cross-Agent Pipeline Integration

### 7.1 컨텍스트 전달 (Context Forwarding)

- [ ] TC-INT-001: Researcher → Planner 컨텍스트 전달

  **동작**:
  1. `@Researcher Investigate the authentication patterns in this project`
  2. 결과 확인 후 Planner Handoff 버튼 클릭 (또는 수동으로 @Planner 호출)
  3. `@Planner Based on the research findings above, create an implementation plan for adding OAuth2 support`

  **Pass 기준**:
  - [ ] Planner가 Researcher의 조사 결과를 활용
  - [ ] 계획이 조사에서 발견된 기존 패턴을 반영
  - [ ] 기술 스택이 일관됨 (Researcher 감지 → Planner 사용)

---

- [ ] TC-INT-002: Planner → Implementer 컨텍스트 전달

  **동작**:
  1. TC-INT-001의 계획이 생성된 상태
  2. Implementer Handoff 또는 `@Implementer Implement Step 1 from the plan above`

  **Pass 기준**:
  - [ ] Implementer가 계획의 Step 1을 정확히 구현
  - [ ] 계획에 명시된 파일을 수정/생성
  - [ ] 계획의 Acceptance Criteria를 준수

  **테스트 후 정리**: `git checkout .`

---

### 7.2 CLI Standalone Workflow

- [ ] TC-INT-003: 수동 순차 호출

  **동작** (전체 수동 파이프라인):
  1. `@Researcher Investigate how to add a simple logger utility to this project`
  2. 결과를 복사
  3. `@Planner Here are the research findings: [복사한 결과]. Create an implementation plan for adding a logger utility.`
  4. 계획을 복사
  5. `@Implementer Here is the plan: [복사한 계획]. Implement Step 1.`
  6. `@Tester Here are the changes: [변경 요약]. Generate and run tests.`
  7. `@Reviewer Review the implementation and test results: [요약]`

  **Pass 기준**:
  - [ ] 각 에이전트가 전달된 컨텍스트를 올바르게 활용
  - [ ] Researcher → Planner 결과 반영
  - [ ] Planner → Implementer 계획 실행
  - [ ] 파이프라인 전체가 수동으로 완료 가능
  - [ ] Reviewer가 최종 리뷰 제공

  **Fail 기준**:
  - 에이전트가 전달된 컨텍스트를 무시
  - 컨텍스트 전달 없이 독립적으로 동작

  **테스트 후 정리**: `git checkout .`

---

### 7.3 파이프라인 일관성

- [ ] TC-INT-004: 동적 탐지 일관성

  **검증 포인트**: 전체 파이프라인에서 기술 스택 감지가 일관되는지 확인

  **동작**:
  1. Researcher의 기술 스택 보고서 확인
  2. Planner의 Technology Context 확인
  3. Implementer의 사용 패턴 확인
  4. Tester의 테스트 프레임워크 선택 확인

  **Pass 기준**:
  - [ ] 4개 에이전트 모두 동일한 기술 스택을 감지/사용
  - [ ] 테스트 명령어 일관 (예: 모두 `npm test` 사용)
  - [ ] 코드 스타일 일관 (TypeScript vs JavaScript 등)

---

## Test Section 8: Edge Cases & Negative Tests

### 8.1 잘못된 입력 처리

- [ ] TC-EDGE-001: 모호한 요청 처리

  **입력 프롬프트**:
  ```
  @Orchestrator Make it better
  ```

  **기대 결과**:
  - 무엇을 개선할지 명확화 질문
  - 또는 프로젝트 분석 후 구체적 개선점 제안

  **Pass 기준**:
  - [ ] 명확화 질문 또는 구체적 제안
  - [ ] 맹목적으로 코드 변경하지 않음

---

- [ ] TC-EDGE-002: 범위 외 요청 처리

  **입력 프롬프트**:
  ```
  @Researcher Deploy this application to production
  ```

  **기대 결과**:
  - Researcher의 역할 범위 외임을 설명
  - 적절한 에이전트 추천 (또는 Orchestrator로 리디렉트)

  **Pass 기준**:
  - [ ] 역할 범위 외임을 인식
  - [ ] 적절한 대안 제시
  - [ ] 파일 수정 없음

---

### 8.2 에러 복구

- [ ] TC-EDGE-003: Implementer 에러 후 복구

  **입력 프롬프트**:
  ```
  @Implementer Modify the file at src/nonexistent/path/file.js to add logging
  ```

  **기대 결과**:
  - 파일이 존재하지 않음을 인식
  - 에러를 우아하게 처리
  - 올바른 파일 경로 제안 또는 사용자에게 확인 요청

  **Pass 기준**:
  - [ ] 파일 부재를 인식하고 보고
  - [ ] 크래시하지 않음
  - [ ] 대안 제시

---

### 8.3 동시 세션 (경고 사항)

- [ ] TC-EDGE-004: 동시 세션 간섭 확인

  **⚠️ 주의: 이 테스트는 Bug #295905와 관련될 수 있습니다**

  **동작**:
  1. 첫 번째 채팅에서 `@Researcher` 작업 시작
  2. 동시에 두 번째 채팅에서 `@Implementer` 작업 시작

  **기대 결과**: 
  - 각 세션이 독립적으로 동작
  - 도구 접근이 올바르게 분리됨

  **Pass 기준**:
  - [ ] 각 에이전트가 자신의 도구만 사용
  - [ ] 세션 간 간섭 없음

  **Fail 기준** (KNOWN_BUG 가능):
  - Researcher가 `editFiles` 도구에 접근
  - 세션 간 간섭 발생 (#295905)

---

## Test Section 9: YAML Frontmatter 검증

### 9.1 메타데이터 정확성

- [ ] TC-YAML-001: Orchestrator frontmatter 검증

  **검증 방법**: 파일 직접 확인 (자동화 가능)

  **확인 항목**:
  ```yaml
  name: Orchestrator
  tools: ['agent', 'search', 'read', 'fetch']
  agents: ['Researcher', 'Planner', 'Implementer', 'Tester', 'Reviewer']
  handoffs: 5개 항목 (Research/Plan/Implement/Test/Review)
  ```

  **Pass 기준**:
  - [ ] `name` 필드가 "Orchestrator"
  - [ ] `tools`에 `agent` 포함 (서브에이전트 호출용)
  - [ ] `agents`에 5개 에이전트 모두 나열
  - [ ] `handoffs`에 5개 항목, 모두 `send: false`
  - [ ] 각 handoff의 `agent` 필드가 올바른 에이전트를 가리킴

---

- [ ] TC-YAML-002: Read-Only 에이전트 도구 제한 검증

  **검증 방법**: 파일 직접 확인

  **확인 항목**:
  - Researcher: `editFiles`나 `execute` 도구 없음
  - Planner: `editFiles`나 `execute` 도구 없음
  - Reviewer: `editFiles`나 `execute` 도구 없음

  **Pass 기준**:
  - [ ] Researcher의 `tools`에 `editFiles`/`execute` 없음
  - [ ] Planner의 `tools`에 `editFiles`/`execute` 없음
  - [ ] Reviewer의 `tools`에 `editFiles`/`execute` 없음

---

- [ ] TC-YAML-003: Write 에이전트 도구 포함 검증

  **검증 방법**: 파일 직접 확인

  **확인 항목**:
  - Implementer: `editFiles`와 `execute` 도구 포함
  - Tester: `editFiles`와 `execute` 도구 포함

  **Pass 기준**:
  - [ ] Implementer의 `tools`에 `editFiles` 포함
  - [ ] Implementer의 `tools`에 `execute` 포함
  - [ ] Tester의 `tools`에 `editFiles` 포함
  - [ ] Tester의 `tools`에 `execute` 포함

---

## Test Section 10: copilot-instructions.md 검증

- [ ] TC-INS-001: 프로젝트 지침 적용 확인

  **입력 프롬프트**:
  ```
  @Orchestrator What are the project conventions and guidelines?
  ```

  **기대 결과**:
  - copilot-instructions.md의 내용을 인식
  - Dynamic Technology Discovery 규칙 언급
  - 6개 에이전트 역할 인지

  **Pass 기준**:
  - [ ] Dynamic Technology Discovery 언급
  - [ ] Technology-Agnostic 설계 철학 반영
  - [ ] 에이전트 역할 인식

---

- [ ] TC-INS-002: 기술 감지 순서 준수

  **검증 포인트**: 에이전트가 copilot-instructions.md에 정의된 기술 감지 파일 순서를 따르는지

  **순서** (copilot-instructions.md 기준):
  1. `package.json`
  2. `pyproject.toml`
  3. `Makefile`
  4. `go.mod`
  5. `.ruby-version` / `Gemfile`
  6. `pom.xml` / `build.gradle`
  7. `Cargo.toml`
  8. `docker-compose.yml` / `Dockerfile`

  **Pass 기준**:
  - [ ] 에이전트가 최소 상위 2-3개 파일을 우선 확인 (Chat Debug로 read 순서 확인)

---

## Execution Guidelines (실행 가이드라인)

### 실행 순서 권장

```
Phase 1: Pre-Flight (30분)
├── PF-1 ~ PF-10: 환경 및 에이전트 인식 확인

Phase 2: YAML 정적 검증 (15분)
├── TC-YAML-001 ~ TC-YAML-003: frontmatter 정확성 확인

Phase 3: 개별 에이전트 검증 (2-3시간)
├── Section 1: Orchestrator (TC-ORC-001 ~ TC-ORC-010)
├── Section 2: Researcher (TC-RES-001 ~ TC-RES-005)
├── Section 3: Planner (TC-PLN-001 ~ TC-PLN-005)
├── Section 4: Implementer (TC-IMP-001 ~ TC-IMP-004)
├── Section 5: Tester (TC-TST-001 ~ TC-TST-004)
├── Section 6: Reviewer (TC-REV-001 ~ TC-REV-004)

Phase 4: 통합 테스트 (1-2시간)
├── Section 7: Pipeline Integration (TC-INT-001 ~ TC-INT-004)
├── Section 10: copilot-instructions (TC-INS-001 ~ TC-INS-002)

Phase 5: Edge Cases (30분)
├── Section 8: Edge Cases (TC-EDGE-001 ~ TC-EDGE-004)
```

### 프리미엄 요청 관리

| 테스트 유형 | 예상 요청 소모 | 권장 간격 |
|------------|--------------|---------|
| 단일 에이전트 호출 | 1 요청 | 간격 불필요 |
| Handoff 체인 | 2-3 요청 | 1-2분 |
| Autonomous Pipeline | 5-6 요청 | 5-10분 |
| Full Integration Test | 7-10 요청 | 10-15분 |

**권장**: 시간당 최대 5-6회 Autonomous Pipeline 실행

### 롤백 절차

모든 Write 에이전트(Implementer, Tester) 테스트 후:
```bash
git diff                    # 변경사항 확인
git checkout .              # 모든 변경 롤백
git clean -fd               # 새로 생성된 파일 제거
git status                  # Clean state 확인
```

---

## Results Summary Template (결과 요약 템플릿)

```markdown
# Manual Testing Results — copilot-my-orchestra

**테스트 일시**: YYYY-MM-DD
**VS Code 버전**: 
**Copilot 확장 버전**: 
**테스트 프로젝트**: 

## 결과 요약

| Section | Total | Pass | Fail | Known Bug | Skip |
|---------|-------|------|------|-----------|------|
| Pre-Flight | 10 | | | | |
| Orchestrator | 10 | | | | |
| Researcher | 5 | | | | |
| Planner | 5 | | | | |
| Implementer | 4 | | | | |
| Tester | 4 | | | | |
| Reviewer | 4 | | | | |
| Integration | 4 | | | | |
| Edge Cases | 4 | | | | |
| YAML | 3 | | | | |
| Instructions | 2 | | | | |
| **Total** | **55** | | | | |

## Critical Failures (즉시 해결 필요)
- 

## Known Bug Encounters
- 

## Recommendations
- 
```

---

## Success Criteria (전체 통과 기준)

### Must Pass (필수 통과)
- [ ] PF-7: 에이전트 인식 (에이전트 목록 표시)
- [ ] PF-8: Smoke Test (Orchestrator 응답)
- [ ] TC-ORC-001~005: 의도 분류 5개 중 4개 이상 통과
- [ ] TC-RES-001: Researcher Read-Only 강제
- [ ] TC-PLN-001: Planner Read-Only 강제
- [ ] TC-REV-001: Reviewer Read-Only 강제
- [ ] TC-YAML-002: Read-Only 에이전트 도구 제한
- [ ] TC-INT-003: CLI Standalone 워크플로 완료 가능

### Should Pass (권장 통과)
- [ ] TC-ORC-006: Dynamic Technology Discovery
- [ ] TC-ORC-009: Handoff 버튼 표시
- [ ] TC-PLN-002: 구조화된 계획 생성
- [ ] TC-REV-002: 6차원 리뷰 형식
- [ ] TC-INT-004: 동적 탐지 일관성

### Nice to Pass (추가 검증)
- [ ] TC-ORC-007~008: Autonomous Mode (플랫폼 버그 가능성)
- [ ] TC-EDGE-004: 동시 세션 독립성

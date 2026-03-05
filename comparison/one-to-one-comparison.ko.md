# 1대1 비교 (기준 레포 vs 각 비교 레포)

## 기준 레포
- Repository: `copilot-my-orchestra`
- 핵심 파일:
  - `.github/copilot-instructions.md`
  - `.github/agents/orchestrator.agent.md`
  - `.github/agents/researcher.agent.md`
  - `.github/agents/planner.agent.md`
  - `.github/agents/implementer.agent.md`
  - `.github/agents/tester.agent.md`
  - `.github/agents/reviewer.agent.md`

---

## 1) 기준 레포 vs `copilot-orchestra`

### 공통점
- 오케스트레이터 중심 단계형 라이프사이클.
- 계획/구현/리뷰 책임 분리.
- TDD 중심 작업 흐름.

### 차이점
- 기준 레포는 Researcher, Tester를 포함한 6역할 구조.
- `copilot-orchestra`는 Conductor + 3개 서브에이전트로 더 단순.
- 기준 레포는 동적 기술 스택 탐지 규칙이 더 명시적.
- `copilot-orchestra`는 `plans/*` 기반 단계 기록 방식이 간단하고 실용적.

### 기준 레포가 가져올 점
- 경량 승인 정지점(사용자 확인 게이트).
- 단계 완료 문서 템플릿의 일관된 사용.

### 기준 레포가 유지할 점
- Researcher/Tester 분리의 장점.
- 탐지 우선(dynamic discovery first) 철학.

---

## 2) 기준 레포 vs `Github-Copilot-Atlas`

### 공통점
- Conductor-subagent 계열 구조와 TDD 기반 실행 흐름.
- 단계별 리뷰 게이트 운영.

### 차이점
- `Atlas`는 컨텍스트 절약 전략과 병렬 위임 규칙이 더 강함.
- `Atlas`는 Explorer/Frontend-Engineer 등 전문 역할을 추가.
- 기준 레포는 단순성과 운영 난이도 측면에서 유리.

### 기준 레포가 가져올 점
- 병렬 리서치 운영 규칙 명문화.
- 직접 읽기 vs 위임 판단 기준(컨텍스트 예산) 도입.
- 선택적 프론트엔드 전담 역할.

### 기준 레포가 유지할 점
- 일상 작업에 적합한 낮은 인지 부하.

---

## 3) 기준 레포 vs `Orchestration`

### 공통점
- 오케스트레이터 중심 위임.
- 플래닝과 코딩 역할 분리.

### 차이점
- `Orchestration`은 Designer가 프론트엔드/UI 결정을 강하게 소유.
- Reviewer Council(복수 리뷰어)과 FastCoder/Coder 분리를 가짐.
- 기준 레포는 단일 리뷰 경로 + 범용 구현자 구조.

### 기준 레포가 가져올 점
- UI-heavy 작업에서 Designer 옵션 도입.
- 단순 작업/복잡 작업 구현자 분리(선택적).
- 고위험 변경 시 복수 리뷰어 전략.

### 기준 레포가 유지할 점
- 표준 작업에서의 단일 리뷰 효율성.

---

## 4) 기준 레포 vs `Craftsman`

### 공통점
- 오케스트레이션 우선 모델.
- 계획과 구현의 엄격한 분리.

### 차이점
- `Craftsman`은 아티팩트 중심성이 매우 강함:
  - `.agents/changes/<JIRA>/01-specification.md`
  - `.agents/changes/<JIRA>/02-plan.md`
  - `.agents/changes/<JIRA>/03-tasks-*`
  - `PROGRESS.md`
- Ralph loop + Inspector 체계로 반복 검증을 구조화.
- 기준 레포는 상대적으로 빠른 실행형.

### 기준 레포가 가져올 점
- 장기 작업용 `PROGRESS.md` 스타일 추적.
- 복잡 페이즈에서 Inspector 개념 선택 적용.
- HITL(phase boundary) 옵션.

### 기준 레포가 유지할 점
- 일상 개발에서의 저마찰 운영.

---

## 5) 기준 레포 vs `coordinated-agent-team`

### 공통점
- 오케스트레이터 역할 경계와 다중 에이전트 분업 철학.
- 게이트 중심 진행 통제.

### 차이점
- `coordinated-agent-team`은 계약/상태머신 우선:
  - `.github/agents/CONTRACT.md`
  - `.github/agents/WORKFLOW.md`
  - `.github/agents/DISPATCH-REFERENCE.md`
- JSON-only 통신 규약, 세션 영속성, 의사결정 추적, 재시도 예산까지 강제.
- 기준 레포는 이 수준의 기계 검증형 프로토콜은 아직 없음.

### 기준 레포가 가져올 점
- 최소 `CONTRACT.md` 도입.
- `.agents-work/<session>/status.json`, `tasks.yaml` 기반 세션 기록.
- 수정 루프 재시도 상한(예: 3회) 명문화.

### 기준 레포가 유지할 점
- 과도한 운영 복잡도를 기본값으로 강제하지 않기.

---

## 6) 기준 레포 vs `gem-team`

### 공통점
- 역할 경계가 명확하고 위임 중심.
- 리서치 -> 플래닝 -> 구현 -> 리뷰 흐름을 유지.

### 차이점
- `gem-team`은 DAG/웨이브 병렬 실행과 계획 YAML 중심:
  - `docs/plan/{plan_id}/plan.yaml`
  - `docs/plan/{plan_id}/research_findings_*.yaml`
  - `docs/prd.yaml`
- 실패 유형 분류/로그/PRD 준수 점검이 구조화됨.
- Browser Tester, DevOps, Documentation Writer 등 역할 분화가 큼.
- 기준 레포는 선형 파이프라인 중심으로 더 단순.

### 기준 레포가 가져올 점
- 독립 태스크에 대한 선택적 DAG 모드.
- 실패 유형 표준화(`transient`, `needs_replan`, `escalate`).
- 대형 프로젝트에서 PRD 정합성 체크.

### 기준 레포가 유지할 점
- 소규모 작업의 빠른 선형 실행 기본값.

---

## 1대1 비교 종합 정리

### 목적별 베스트 레퍼런스
- 가장 빠른 도입: `copilot-orchestra`
- 병렬/컨텍스트 효율 확장: `Github-Copilot-Atlas`
- UI 소유권/리뷰 다중화: `Orchestration`
- 계획 심도 + 자율 구현 루프: `Craftsman`
- 엔터프라이즈형 프로세스 거버넌스: `coordinated-agent-team`
- DAG 실행 자동화: `gem-team`

### 기준 레포 우선 개선 항목
1. 최소 세션 영속성 도입: `.agents-work/<session>/status.json`, `tasks.yaml`, `report.md`.
2. 게이트/수정 루프/재시도 정책 명문화.
3. 선택적 전문 역할 1개 추가(Designer 또는 Frontend-Engineer).
4. 필요 시에만 DAG 병렬 모드를 활성화하는 고급 모드 추가.

### 기대 효과
- 현재의 사용성을 유지하면서도,
- 장기/대규모 작업에서 추적성, 재현성, 안정성을 높일 수 있습니다.

# 멀티 에이전트 오케스트레이션 레포 비교 (전체 관점)

## 비교 범위
- 기준 레포: `copilot-my-orchestra`
- 기준 파일:
  - `.github/copilot-instructions.md`
  - `.github/agents/orchestrator.agent.md`
  - `.github/agents/researcher.agent.md`
  - `.github/agents/planner.agent.md`
  - `.github/agents/implementer.agent.md`
  - `.github/agents/tester.agent.md`
  - `.github/agents/reviewer.agent.md`
- 비교 대상 레포:
  - `comparison/repos/copilot-orchestra`
  - `comparison/repos/Github-Copilot-Atlas`
  - `comparison/repos/Orchestration`
  - `comparison/repos/Craftsman`
  - `comparison/repos/coordinated-agent-team`
  - `comparison/repos/gem-team`

## 핵심 요약
- 현재 기준 레포는 6개 역할이 균형 있게 분리된 파이프라인이며, 기술 스택 동적 탐지 규칙이 명확합니다.
- `copilot-orchestra`는 가장 단순하고 실전 적용이 쉬운 TDD 중심 구조입니다.
- `Github-Copilot-Atlas`는 컨텍스트 절약 전략과 병렬 위임 규칙이 강화된 확장형입니다.
- `Orchestration`은 디자인 소유권(Designer)을 강하게 분리하고 다중 리뷰어 전략을 갖습니다.
- `Craftsman`은 스펙/플랜/태스크 아티팩트와 Ralph 루프 기반 검증이 매우 강합니다.
- `coordinated-agent-team`은 계약/상태머신/디스패치 규약 중심의 거버넌스 강도가 가장 높습니다.
- `gem-team`은 DAG(웨이브) 병렬 실행과 YAML/JSON 중심 구조화된 자동화가 강점입니다.

## 에이전트 구성 스냅샷
- 기준 레포 (`copilot-my-orchestra`): 에이전트 6개 + 프로젝트 지침 1개
- `copilot-orchestra`: 에이전트 파일 4개
- `Github-Copilot-Atlas`: 에이전트 파일 7개
- `Orchestration`: 에이전트 파일 8개
- `Craftsman`: 에이전트 파일 2개 (내부적으로 서브 퍼소나 루프 정의)
- `coordinated-agent-team`: 역할 파일 12개 + 거버넌스 문서 3개 (`CONTRACT.md`, `WORKFLOW.md`, `DISPATCH-REFERENCE.md`)
- `gem-team`: 에이전트 파일 8개

## 전체 비교 매트릭스

| 비교 축 | 기준 레포 (`copilot-my-orchestra`) | copilot-orchestra | Atlas | Orchestration | Craftsman | coordinated-agent-team | gem-team |
|---|---|---|---|---|---|---|---|
| 핵심 토폴로지 | Orchestrator -> Research -> Plan -> Implement -> Test -> Review | Conductor -> Planning -> Implement -> Review | Atlas + Prometheus + Oracle/Explorer/Sisyphus/Frontend/Review | Orchestrator + Planner + Designer + Coder/FastCoder + Reviewer Council | Plan Mode + Ralph Loop | 12개 에이전트 기반 상태머신 | Orchestrator + DAG 워커 에이전트 |
| 동적 스택 탐지 | 지침에 명시적으로 강제 | 제한적 | 간접적/제한적 | 중심 요소 아님(레포 제약 중심) | 인터뷰형 플래닝에서 문맥 탐색 | 스펙/아키/리서치 단계에서 프로세스 기반 탐색 | 포커스 영역 리서치 + 플래너 합성 |
| 워크플로 엄격성 | 중상 | 중 | 상 | 상 | 매우 높음 | 매우 높음 | 매우 높음 |
| 아티팩트 지속성 | 대화 중심 + 계획 가이드 수준 | `plans/` 문서 | 설정 가능한 plan 디렉터리 | 상대적으로 느슨한 아티팩트 계약 | `.agents/changes/...` + `PROGRESS.md` 강함 | `.agents-work/<session>/...` 엄격한 세션 모델 | `docs/plan/{plan_id}/plan.yaml`, PRD, 로그, 리서치 결과 |
| 게이트 모델 | 의도 분류 + 단계 흐름 | 승인/커밋 정지점 강제 | 승인/커밋 정지점 강제 | 강한 위임 규칙 + 리뷰어 협의 | Task/Phase Inspector + HITL 옵션 | APPROVE_DESIGN/REVIEW_STRATEGY/수정 루프 강제 | Plan review + 재시도 + PRD 준수 게이트 |
| 병렬성 | 서브에이전트 흐름으로 지원 | 낮음 | 높음(권장) | 높음(권장) | 루프 중심(대체로 순차) | 통제된 결정적 디스패치 | DAG 웨이브 기반 병렬 실행 |
| 출력 계약 엄격성 | 중 | 중 | 중상 | 중 | 상 | 매우 높음(JSON 스키마 + 영속성 프로토콜) | 높음(JSON + YAML 스키마) |
| 보안 관점 | Reviewer 품질 점검 중심 | 리뷰 중심 | 리뷰 중심 | 리뷰어 협의체 패턴 | 검사자 중심 품질 검증 | Security 전담 에이전트 + 게이트 의미론 | Reviewer의 OWASP/PRD 준수 점검 강화 |
| UI/UX 전문화 | 일반형(Designer 없음) | 전용 UI 역할 없음 | Frontend 전용 역할 존재 | Designer 소유권 강제 | 간접적(태스크 기반) | Designer 전담 존재 | Browser tester/doc writer/reviewer 분리 |

## 세부 분석

### 1) 역할 분리와 책임 경계
- 기준 레포는 역할 책임이 명확하고 이해하기 쉽습니다.
- `Orchestration`, `coordinated-agent-team`은 디자인/리뷰 거버넌스를 더 강하게 분리합니다.
- `gem-team`도 역할 간 금지 규칙(예: 구현 금지)을 명시해 경계가 선명합니다.
- `Craftsman`은 상위 에이전트 수는 적지만 루프/검사자 규율이 강합니다.

### 2) 결정성(Determinism) vs 유연성
- 기준 레포는 실무형 균형 모델입니다.
- `copilot-orchestra`, `Atlas`는 실용적 단계 루프 + 사용자 체크포인트를 가집니다.
- `coordinated-agent-team`은 상태 전이/영속성 검증이 가장 엄격합니다.
- `gem-team`은 선형 단계 대신 DAG 웨이브 실행의 결정성이 핵심입니다.
- `Craftsman`은 태스크/페이즈 경계에서 반복 검증 루프가 명확합니다.

### 3) 상태 관리와 아티팩트 모델
- 기준 레포는 장기 실행 관점에서 하드 영속성 계약은 상대적으로 약합니다.
- `coordinated-agent-team`은 `status.json`, `tasks.yaml`, 의사결정 추적 등 감사 가능성이 가장 높습니다.
- `gem-team`은 `plan.yaml` 중심 실행 상태와 리서치 결과 분리 저장이 강합니다.
- `Craftsman`은 사람이 읽고 승인하기 좋은 계획 아티팩트 체계가 강점입니다.

### 4) 품질/검증 전략
- 기준 레포는 Tester + Reviewer 분리가 이미 좋은 기본 구조입니다.
- `copilot-orchestra`, `Atlas`는 TDD + 단계별 리뷰/커밋 리듬이 강합니다.
- `Craftsman`은 Task Inspector/Phase Inspector로 검증 밀도를 높입니다.
- `coordinated-agent-team`은 수정 루프와 재시도 예산을 형식화합니다.
- `gem-team`은 에이전트별 검증 및 PRD 준수를 구조화합니다.

### 5) 대규모 작업 확장성
- 기준 레포도 확장 가능하지만, 장기/대규모 작업에는 아티팩트 계약 보강 여지가 큽니다.
- `Atlas`, `gem-team`은 병렬/분업 설계로 대규모 처리에 유리합니다.
- `coordinated-agent-team`은 팀 단위 재현성과 통제에 매우 강합니다.
- `Craftsman`은 깊은 플래닝 후 자율 구현 루프에 강합니다.

### 6) 도입/운영 비용
- 기준 레포, `copilot-orchestra`가 가장 빠르게 도입 가능합니다.
- `Atlas`는 중간 복잡도로 확장성 이점을 제공합니다.
- `Orchestration`은 정책 복잡도가 증가합니다.
- `Craftsman`, `coordinated-agent-team`, `gem-team`은 초기 설정/유지 비용이 높습니다.

## 기준 레포의 상대적 강점/보완점

### 강점
- 기술 스택 동적 탐지 지침이 명시적이고 이식성이 높습니다.
- 역할 정의가 실무적으로 명확합니다.
- 일반적 개발 요청에 대한 기본 파이프라인이 안정적입니다.
- Tester 지침의 프레임워크 탐지 방식이 탄탄합니다.

### 보완점
- `coordinated-agent-team`, `gem-team` 수준의 기계 검증형 계약은 아직 약합니다.
- 장기 세션용 영속 상태/아티팩트 규약이 상대적으로 약합니다.
- UI/UX 전담 역할(Designer/Frontend Specialist)이 없습니다.
- 재시도 예산/수정 루프를 명시한 상태머신 규칙이 없습니다.

## 기준 레포 개선 제안 (점진적)

### 단계 A (저마찰)
- 최소 세션 아티팩트 도입:
  - `.agents-work/<session>/status.json`
  - `.agents-work/<session>/tasks.yaml`
  - `.agents-work/<session>/report.md`
- 기존 6역할 구조는 그대로 유지.

### 단계 B (품질 게이트)
- 명시적 게이트/수정 루프 추가:
  - `APPROVE_PLAN`
  - 선택적 `REVIEW_STRATEGY`
  - `FIX_REVIEW` / `FIX_TESTS` + 최대 재시도 횟수.

### 단계 C (전문화)
- 우선 1개 전담 역할 추가 권장: `Designer` 또는 `Frontend-Engineer`.
- 최소 버전 `CONTRACT.md` 도입.

### 단계 D (확장 모드)
- 필요 시 독립 작업에 한해 DAG/웨이브 실행 모드 추가.
- 소규모 작업에는 선형 모드를 기본으로 유지.

## 결론
- 현재 기준 레포는 실무성과 단순성의 균형이 좋은 중간 지점입니다.
- 팀 거버넌스/재현성 최우선이면 `coordinated-agent-team`이 가장 좋은 참조입니다.
- 대규모 병렬 처리 최우선이면 `gem-team`, `Atlas`가 강합니다.
- 계획 심도 + 자율 루프 최우선이면 `Craftsman`이 강합니다.
- 빠른 도입성과 단순성은 `copilot-orchestra`가 가장 좋습니다.

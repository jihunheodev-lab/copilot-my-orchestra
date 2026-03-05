# Copilot 멀티 에이전트 오케스트레이션 시스템

GitHub Copilot을 위한 고도화된 멀티 에이전트 시스템입니다. 코드 생성과 개발 워크플로를 협업 중심으로 구성할 수 있도록, 역할별 전문 에이전트를 제공합니다. 이 시스템은 작업 분해와 지능형 조정을 통해 복잡한 엔지니어링 요구사항을 처리합니다.

## 주요 기능

- **전문화된 역할 분리**: 조사부터 리뷰까지 전체 개발 라이프사이클을 아우르는 6개 에이전트
- **동적 기술 스택 탐지**: `package.json`, `pyproject.toml` 등 프로젝트 메타데이터를 분석해 실제 스택에 맞게 동작
- **파이프라인 오케스트레이션**: Research -> Plan -> Implement -> Test -> Review 흐름과 필수 승인 게이트 내장
- **단일 진입점**: 모든 작업은 `@Orchestrator`에서 시작 — 에이전트를 직접 전환할 필요 없음
- **VS Code 네이티브 통합**: 최신 GitHub Copilot 에이전트 기능에 최적화

## 아키텍처

```text
+----------------+       +-------------------+
|   사용자 요청   |------>|   오케스트레이터   |
+----------------+       +---------+---------+
                                   |
           +-----------------------+-----------------------+
           |                       |                       |
   +-------v-------+       +-------v-------+       +-------v-------+
   |   리서처       |       |   플래너       |       |  구현자        |
   | (컨텍스트/코드)|------>| (단계별 계획)  |------>| (코드 변경)    |
   +---------------+       +---------------+       +-------+-------+
                                                           |
           +-----------------------+-----------------------+
           |                       |
   +-------v-------+       +-------v-------+
   |   테스터       |       |   리뷰어       |
   | (검증)         |------>| (최종 점검)    |
   +---------------+       +---------------+
```

## 설치

1. 저장소에 `.github/agents/` 디렉터리를 생성합니다.
2. 이 저장소의 모든 `.agent.md` 파일을 `.github/agents/`로 복사합니다.
3. VS Code를 재시작하거나 GitHub Copilot을 새로 고칩니다.

## 사용 방법

### VS Code에서
1. GitHub Copilot Chat을 엽니다.
2. `@Orchestrator` 뒤에 요청을 입력합니다. 예: `@Orchestrator implement a new authentication middleware`
3. 생성된 계획을 검토하고 구현 시작을 확인합니다.
4. 승인 후 오케스트레이터가 파이프라인을 자율적으로 실행합니다.

### CLI(독립 실행)에서
환경에서 서브에이전트 호출을 지원하지 않으면 아래 순서를 수동으로 진행합니다.
1. `@Researcher`로 컨텍스트 수집
2. 조사 결과를 `@Planner`에 전달해 실행 계획 생성
3. 계획을 검토하고 진행을 확인합니다.
4. `@Implementer`로 계획 적용
5. `@Tester`와 `@Reviewer`로 검증

## 에이전트 카탈로그

| 에이전트 | 목적 | 주요 도구 |
|-------|---------|---------------|
| **Orchestrator** | 파이프라인 조정 및 의도 분류 | agent, search, read |
| **Researcher** | 코드베이스/문서 컨텍스트 조사 | search, read, fetch, web |
| **Planner** | 상세 기술 구현 계획 수립 | read, search |
| **Implementer** | 코드 변경 및 리팩터링 수행 | edit, editFiles, read |
| **Tester** | 테스트 생성 및 실행 검증 | execute, read, search |
| **Reviewer** | 품질 점검 및 최종 검토 | read, search, read/problems |

## 호환성 매트릭스

| 환경 | Orchestrator(@) | 서브에이전트 위임 |
|-------------|-------------------|---------------------|
| VS Code Insiders | 지원 | 지원 |
| VS Code Stable | 지원 | 부분 지원 |
| GitHub.com | 지원 | 미지원 |
| Copilot CLI | 지원 | 미지원 |

## 기본 파이프라인

시스템의 기본 흐름은 다음 6단계입니다.
1. **Research**: 기존 구현/의존성 분석
2. **Plan**: 기술 명세와 단계별 작업 분해
3. **Approval**: 사용자가 계획을 검토하고 구현 시작을 승인
4. **Implement**: 작은 단위의 원자적 코드 변경 실행
5. **Test**: 단위/통합/회귀 테스트 검증
6. **Review**: 성능/보안/프로젝트 규칙 최종 점검

## 커스터마이징

`.agent.md` 파일을 편집하여 에이전트 동작을 맞춤화할 수 있습니다.
- **도구 수정**: YAML frontmatter의 `tools` 목록 갱신
- **프롬프트 조정**: 팀 표준에 맞게 지시문 수정
- **제약 추가**: Orchestrator의 의도 분류 섹션에 프로젝트 규칙 추가

## 한국어 에이전트 문서

아래 문서는 루트 `.github/agents/`에 있는 6개 에이전트의 한국어 버전입니다.

- `ko/agents/orchestrator.agent.ko.md` (풀버전)
- `ko/agents/orchestrator.summary.ko.md` (주요 내용 요약)
- `ko/agents/researcher.agent.ko.md` (풀버전)
- `ko/agents/researcher.summary.ko.md` (주요 내용 요약)
- `ko/agents/planner.agent.ko.md` (풀버전)
- `ko/agents/planner.summary.ko.md` (주요 내용 요약)
- `ko/agents/implementer.agent.ko.md` (풀버전)
- `ko/agents/implementer.summary.ko.md` (주요 내용 요약)
- `ko/agents/tester.agent.ko.md` (풀버전)
- `ko/agents/tester.summary.ko.md` (주요 내용 요약)
- `ko/agents/reviewer.agent.ko.md` (풀버전)
- `ko/agents/reviewer.summary.ko.md` (주요 내용 요약)

## 기여

1. 저장소를 포크합니다.
2. 기능 브랜치를 생성합니다.
3. 변경 사항을 명확히 설명한 Pull Request를 제출합니다.

## 라이선스

MIT License

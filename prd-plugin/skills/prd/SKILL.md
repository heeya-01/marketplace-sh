---
name: prd
description: 간략한 기능 설명을 입력하면 BA, 기획자, QA 에이전트가 협업하여 PRD를 자동 생성합니다.
argument-hint: "[기능 설명]"
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Bash, Task
---

# PRD 자동 생성

**입력된 기능**: $ARGUMENTS

아래 순서로 PRD를 생성합니다.

---

## Step 1: BA 에이전트 — 요구사항 수집 및 초안 생성

BA 에이전트를 실행합니다.

```
Task(
  subagent_type: "ba",
  prompt: "기능 설명: {ARGUMENTS}. 사용자에게 5가지 질문을 한 번에 제시하고, 답변을 받아 PRD 초안을 docs/prd/{feature}/prd-draft.md에 저장하세요. 모든 커뮤니케이션은 한국어로 진행하세요."
)
```

BA 에이전트가 사용자와 상호작용을 완료하고 초안 파일을 저장할 때까지 대기합니다.

---

## Step 2: 기획자 + QA 에이전트 — 병렬 구체화

기획자와 QA 에이전트를 **동시에** 실행합니다.

```
Task(
  subagent_type: "planner",
  prompt: "docs/prd/{feature}/prd-draft.md를 읽고, UX/기획 관점에서 섹션 2, 3, 4를 구체화하세요. 불명확한 항목이 있으면 질문을 한 번에 묶어서 사용자에게 제시하고 답변 후 업데이트하세요. 한국어로 진행하세요."
)

Task(
  subagent_type: "qa",
  prompt: "docs/prd/{feature}/prd-draft.md를 읽고, QA 관점에서 섹션 5, 6, 7을 작성하세요. 불명확한 항목이 있으면 질문을 한 번에 묶어서 사용자에게 제시하고 답변 후 업데이트하세요. 한국어로 진행하세요."
)
```

두 에이전트가 모두 완료될 때까지 대기합니다.

---

## Step 3: 완료 보고

최종 PRD 파일 경로와 함께 아래 내용을 사용자에게 보여줍니다:

```
✅ PRD 생성 완료

📄 파일 위치: docs/prd/{feature}/prd-draft.md

[BA]      개요 및 기능 요구사항 초안 작성
[기획자]  사용자 스토리, 기능 요구사항, 화면/플로우 구체화
[QA]      수용 기준, 엣지케이스, NFR 추가

수정이 필요하면 파일을 직접 편집하거나 /prd를 다시 실행하세요.
```

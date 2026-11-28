# Learning Orchestrator

The Learning Orchestrator, also called the Curator, coordinates learner state and the exchange of work among the Planner, Tutor, and Assessment Engine.

## 1. Purpose

Guide the learner from a goal to a curriculum and through teaching, assessment, review, and progression. The Curator is the sole writer to the main database and supplies scoped Learning Context whenever it invokes or resumes a worker.

## 2. Responsibilities

- Coordinate preview, curriculum validation and publication, course access, familiarity, and explicit replans.
- Own sessions, transcripts, assigned objectives, selected teaching strategies, current exercises, attempts, and completion state.
- Invoke agents, validate their reports, and persist accepted evidence and progression effects.
- Supply the appropriate context to Tutor and Assessment. Refresh the conversation summary and apply accepted evidence promptly.
- Resume interrupted work, enforce authorization, and apply retries once.

## 3. Non-Responsibilities

| Area | Responsible component |
| --- | --- |
| Academic curriculum, objectives, and recommendations | Learning Planner |
| Instruction, practice, and teaching observations | Tutor |
| Assessment preparation and grading for a frozen attempt | Assessment Engine |
| Admission and persistence of reusable educational material | Teaching Library service |

## 4. Supported Actions

The [learner API](agentic-learning-platform/docs/WORKFLOWS.md#api-actions) defines the goal, curriculum, course-entry, familiarity, Tutor turn, finish-study, assessment, resume, and replan actions. It includes attempt inspection, retry, abandonment, and reporting suspected assessment errors.

## 5. Input Contracts

Authenticate the learner and resolve every reference within the learner’s records. Mutations carry a stable request ID. Teaching turns identify the session and matching exercise where relevant. Assessment submission identifies the saved attempt and items. Replans identify the active version and reason, with an updated preview for a changed goal. [API actions](agentic-learning-platform/docs/WORKFLOWS.md#api-actions)

Supply workers with scoped inputs under the [shared agent contracts](agentic-learning-platform/docs/AGENT-CONTRACTS.md). For Tutor this includes the current objective, strategy instructions, relevant evidence, and conversation. For Assessment preparation this includes purpose, objectives, applicable policy, and relevant selection context. Grading receives the exact saved private snapshot and submission. Feedback presentation preferences are scoped to feedback production.

## 6. Output Contracts

Return the learner-visible result, active references, supported next action, and an explanation when an action is blocked. The Curator retains the prepared assessment and private grading snapshot and returns only the public presentation. It presents accepted feedback and current attempt or review state on resume. [API results and state](agentic-learning-platform/docs/WORKFLOWS.md#api-actions)

### Agent reporting

Accept an action-specific result or structured issue from every completed worker call. A Tutor turn contains the reply and an observation list that may be empty, plus sources and pending exercise as applicable. An assessment preparation returns the public presentation and private grading snapshot. A grading result includes purpose, scope, outcome, and evidence by objective. Bind each result to its originating turn or attempt and curriculum version. [Shared envelopes and worker payloads](agentic-learning-platform/docs/AGENT-CONTRACTS.md)

## 7. Internal Workflow

1. Show the goal preview, then validate and publish the complete plan. Resolve course availability from its prerequisite DAG.
2. When an available course opens, offer the optional familiarity conversation and persist its assignment. Restore saved assignments and pending work on re-entry.
3. Save learner input, select the teaching strategy, and invoke one Tutor call for a normal turn. Save the reply and validated observations before choosing the next action.
4. When an assessment becomes available, allocate its ID before preparing it. Save the returned presentation and grading snapshot before showing questions. Freeze the final submission before grading.
5. Accept the grade, append evidence, and update eligible progression in one transaction. Provide targeted review where needed. Use objective feedback directly if a Planner review recommendation is unavailable.
6. Refresh summary text on the [Learning Context cadence](agentic-learning-platform/backend/app/agents/learning_context/AGENT.md#update-and-conflict-rules). Construct subsequent snapshots from the current accepted evidence and relevant conversation.

## 8. Decision Rules

- Select a versioned strategy from guided questioning, worked example, or guided practice in v1. Guided questioning uses a focused prompt and progressive hints. A worked example solves an analogous problem before a fresh exercise. Guided practice, the default, lets the learner attempt a task and then get feedback. An explicit request or repeated difficulty on the objective can justify a switch. Record the reason and send the corresponding instructions on the next call.
- Decide readiness and progression from the [learner workflow](agentic-learning-platform/docs/WORKFLOWS.md#goal-and-course-flow) and [assessment policy](agentic-learning-platform/docs/REQUIREMENTS.md#assessment-policy-v1). Tutor observations inform instruction and readiness. Scoped assessments provide demonstrated evidence under the policy.
- Skips change instruction. All mandatory objectives must be demonstrated through the course completion series before clearing prerequisite access. A targeted check can guide review and carries no clearance by itself.
- Preserve the original course and version on late reports. Only the applicable current evidence can change active progress. Keep each summary’s transcript cursor and apply summary writes to summary fields.

## 9. Dependencies

Use the [Planner](LEARNING_PLANNER.md), [Tutor](TUTOR.md), [Assessment Engine](ASSESSMENT.md), [Learning Context](agentic-learning-platform/backend/app/agents/learning_context/AGENT.md), [Teaching Library](agentic-learning-platform/docs/TEACHING-LIBRARY.md), and [main database schema](agentic-learning-platform/docs/DATABASE-SCHEMA.md).

## 10. Failure Handling

Validate worker output and expected state before committing it. Preserve saved input on an interrupted Tutor call. For preparation or grading outages, resume the saved operation under the same attempt ID. For inadequate grounding, coverage, or novelty, surface the affected objective and route it for revision. Return a conflict if the same request ID is reused with different inputs. Apply each accepted effect once and keep the prior curriculum available until a replacement validates. [Attempt lifecycle](agentic-learning-platform/docs/WORKFLOWS.md#attempt-lifecycle)

## 11. Acceptance Criteria

1. A new learner sees the full validated curriculum before optional course-entry familiarity.
2. Tutor and Assessment receive scoped context and report back through the Curator. Resuming restores the exact current work.
3. Accepted observations can change teaching strategy, and valid assessment results update only the relevant objectives and progression.
4. A learner can complete a course after targeted remediation of a missed objective. Passing a targeted check alone keeps prerequisite clearance pending.
5. Duplicate or late results, failed replans, model timeouts, and delayed summaries preserve learner state under the workflow rules.

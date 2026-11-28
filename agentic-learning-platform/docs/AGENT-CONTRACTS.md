# Shared agent contracts

These are the authoritative logical payloads for MVP v1. The Orchestrator invokes workers and applies accepted results. The [workflow document](WORKFLOWS.md) defines the learner API and transitions. The [requirements](REQUIREMENTS.md#assessment-policy-v1) define the common assessment policy.

## Request and result envelope

- Each worker request has an Orchestrator-issued request ID, action, and action-specific payload. Repeat the same ID and immutable inputs on a transport retry. A result echoes the request ID and the applicable curriculum version, course, turn, or attempt reference.
- Each result contains either the action output or an issue with a code, reason, affected references, and a retryable flag. Common codes are `invalid_reference`, `invalid_output`, `conflict`, and `unavailable`. Assessment-specific codes are listed in the [engine specification](../../ASSESSMENT.md#10-failure-handling).
- Authenticate and check ownership before resolving a learner-provided reference. Worker IDs are opaque; supply scoped data rather than account identifiers or whole learner records. Validate references and required fields before committing an output.

## Curriculum and objective identity

| Record | Required fields |
| --- | --- |
| Goal preview | Preview reference assigned by the Orchestrator, original goal, interpreted direction, milestones, and optional directions to choose before building |
| Curriculum | Goal and selected direction, milestones, courses, prerequisite edges, and a syllabus for each course; the Orchestrator assigns an immutable version on publication |
| Milestone | ID, description, and course IDs that must pass to reach it |
| Course | ID, title, purpose, outcome records, and source references |
| Outcome | ID and statement; each outcome is linked from at least one objective |
| Topic | ID, title, order where assigned, priority, and sources |
| Objective | ID, course ID, topic IDs within the course, outcome IDs, observable statement, depth, priority, demonstration requirement, and source references |
| Prerequisite edge | Upstream and downstream course IDs, upstream mandatory objective IDs that explain the dependency, reason, and sources; mark academic inferences in the rationale |
| Demonstration requirement | Task form such as explain, solve, apply, or justify, plus whether working or reasoning must be shown |
| Study recommendation | Course ID, objective IDs, action of study, review, or skip instruction, reason, and supporting evidence references |

The Planner creates course, milestone, outcome, topic, and objective IDs. Use the curriculum version together with the ID when referring to any published record. IDs are unique within their record type and version. Depth values are `recall`, `explain`, `apply`, and `analyze`. Topic priority is `mandatory`, `recommended`, `optional`, or `out_of_scope`. Objectives use the first three. Each course outcome is covered by mandatory objectives. Every mandatory topic links to at least one mandatory objective. Optional and recommended objectives can be taught or practiced as enrichment. Out-of-scope topics appear outside the teaching sequence.

The selected MVP route contains required courses. Alternative directions appear at preview, then the learner chooses a route or accepts a broad starting direction. Each route has at least one milestone, each with a nonempty set of required course IDs. Completion of those courses marks the milestone reached. Completing all milestones completes the route. The Orchestrator validates unique IDs, references, syllabus coverage, nonempty mandatory objective sets, and acyclicity. The Planner checks the academic fit, including the depth of prerequisite coverage.

## Planner actions

| Action | Input | Result |
| --- | --- | --- |
| `preview_goal` | Goal and optional goal clarifications | Goal preview or clarification question |
| `build_curriculum` | Accepted preview and selected direction; optional approved source references | Complete curriculum proposal |
| `check_familiarity` | Published plan and course; relevant familiarity or assessment evidence; optional answer with its question ID; remaining prompt budget | Question with objective IDs, or study recommendations |
| `recommend_review` | Published plan, assessed course, accepted objective evidence, and current assignment | Scoped study recommendations, possibly empty |
| `replan_curriculum` | Current plan and reason, with updated preview for a changed goal; optional source references | Complete replacement proposal and revision rationale |

## Tutor action

`teach_turn` receives the plan version, course and objective IDs, assignment, objective statement and depth, selected strategy and its versioned instructions, and a scoped context snapshot. The request includes an instruction to start or the learner's message or exercise answer. Continuation includes the pending exercise and recent turns; remediation includes missed objective IDs and learner-facing assessment feedback.

It returns a learner-facing reply, approved source references, an observation list that may be empty, an optional pending exercise, and an advisory next action with a reason. The exercise has an ID, objective, prompt, and assistance history. Each observation identifies the objective, an originating turn or exercise, and concise supporting evidence. For a response to practice, include assistance if known as `independent`, `hinted`, or `worked_example`. Evidence from practice supports instructional choices.

## Assessment actions

| Action | Input | Result |
| --- | --- | --- |
| `prepare_assessment` | Attempt ID allocated before the call, version and course, purpose, complete in-scope objective set, syllabus snapshot, policy version and rules, prior-item exclusions for verification, and scoped selection context where relevant | Learner presentation and immutable private grading snapshot |
| `grade_submission` | Attempt ID, saved private snapshot, and frozen submission; optional feedback presentation preferences supplied to the feedback stage | Purpose, scope, policy version, outcome and reason, criterion judgments, objective evidence, and learner-facing feedback |

Allowed purposes are `course_completion`, `targeted_check`, and `post_remediation_verification`. Completion uses all mandatory objectives. A targeted check uses an explicit nonempty subset of mandatory objectives or the whole set. Verification requires a completion series ID and a nonempty set of its remaining or invalidated mandatory objectives. The Curator derives those sets from the published plan and saved state.

The **learner presentation** contains ordered item IDs, prompts, instructions, required working, and public success expectations. The **private grading snapshot** contains the attempt ID, plan version, course, purpose, completion series ID when applicable, frozen policy and version, instruction versions, scoped objectives, and all items with their IDs, prompts, formats, criteria, keys or reference solutions, source references and permitted excerpts needed for offline grading. Each criterion has an ID, maps to one objective, and identifies its method, required status, reasoning requirements, and saved scoring rules. Each in-scope objective has at least one required criterion. A submission references the exact item IDs and records answers or blanks on explicit final submission.

Each criterion judgment carries item ID, criterion ID, objective ID, outcome, rationale, and a reference to the answer or its recorded absence. Each objective evidence record carries objective ID, attempt ID, criterion judgment references, status, and a concise basis, including item count and formats. `passed`, `not_passed`, and `inconclusive` describe the attempt's assessed scope. The [assessment policy](REQUIREMENTS.md#assessment-policy-v1) determines how that scope affects progression.

Exclusions use item or exercise IDs, normalized prompt fingerprints, and available scenario, values, and answer fingerprints from prior items. Include overlapping Tutor exercise prompts through a scoped export stripped of learner details. Public success expectations can be derived from the rubric. Private keys and scoring data stay in the saved grading record. Score using the snapshot and submission first; use any feedback preferences only with the fixed judgments when producing feedback.

Optional selection context includes language, accessibility needs, relevant evidence by objective, and strategy IDs or activity formats used for those objectives. It supports the choice of comparably demanding tasks. The grading input excludes learner background and instructional observations. The separate feedback stage may use language, accessibility, and level-of-detail preferences together with the fixed judgments.

## Learning Context handoff

The Orchestrator builds each snapshot from [Learning Context](../backend/app/agents/learning_context/AGENT.md). Evidence identifies its kind as `self_report`, `tutor_observation`, or `graded_assessment`, and references the originating event and objective where applicable. A summary is a separate versioned projection of a transcript range. The Orchestrator accepts a worker result only when its identifiers, scope, and state transition are valid.

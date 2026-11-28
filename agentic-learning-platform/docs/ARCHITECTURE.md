# MVP architecture

## Components and ownership

The browser talks to the Learning Orchestrator. The Orchestrator invokes the Planner, Tutor, and Assessment Engine through [shared contracts](AGENT-CONTRACTS.md). The workers produce proposals, observations, or grading evidence. The Orchestrator makes and persists workflow, access, and context decisions.

| Component | Responsibility |
| --- | --- |
| [Learning Planner](../../LEARNING_PLANNER.md) | Propose goals, milestones, university-grounded courses and objective-level syllabi, and study or review assignments. |
| [Learning Orchestrator](../../LEARNING_ORCHESTRATOR.md) | Own learner identity, the active curriculum, sessions, workflow, strategies, accepted evidence and main database writes. |
| [Tutor](../../TUTOR.md) | Teach the assigned objective and report observations with assistance provenance. |
| [Assessment Engine](../../ASSESSMENT.md) | Prepare assessments and grade frozen attempts under the same policy. Report scope and objective-level evidence. |
| [Learning Context](../backend/app/agents/learning_context/AGENT.md) | Curator-owned learner record and scoped snapshots. A summarization helper can propose summary text. |
| [Shared teaching library](TEACHING-LIBRARY.md) | Serve approved material to workers and maintain shared source and cache entries through its own service. |

Store source and generated material with provenance. Keep curriculum, transcript, evidence, and attempt records scoped to the learner. Save exactly what is needed to resume and to grade offline. The storage and transition rules are in [MVP data schema](DATABASE-SCHEMA.md) and [learner workflows](WORKFLOWS.md).

## Implementation defaults

Use one Python application with FastAPI for HTTP endpoints, a small same-origin browser UI, and SQLite for relational persistence in the course-project demo. Keep persistence behind repository interfaces and keep database transactions short around reservations and commits. Use seeded demo accounts with server-owned, revocable sessions, and resolve identity and ownership at the HTTP boundary. Use a configured provider behind the model adapter for live agent calls and deterministic stubs for contract testing and the local demo. Runtime model choice is deployment configuration. The scaffold is ready for implementation against these decisions, with dependencies and run commands added when the executable slice is built. [FastAPI documentation](https://fastapi.tiangolo.com/) · [SQLite transactions](https://www.sqlite.org/lang_transaction.html)

Start with two courses and one mandatory dependency using the [library's initial source set](TEACHING-LIBRARY.md#initial-source-set). The first course can teach interface contracts and the second can use those contracts to compare architecture alternatives. Use fixtures that exercise a learner who already knows part of the first course and needs a targeted verification after review.

## Decisions adopted for v1

- Build and publish the complete curriculum before a learner is asked about course familiarity. The optional pre-planning diagnostic in the archived Assessment draft is deferred, along with its separate evidence type and route.
- The Orchestrator owns context persistence and selects the teaching strategy. The environment may use an optional stateless summarization helper. Refresh summaries by transcript range and accept typed evidence as it arrives.
- An assessment outcome applies to its saved scope. Completion and clearance, evidence carry-forward, retries, disputes, and deferred assessment features follow [Assessment policy v1](REQUIREMENTS.md#assessment-policy-v1).
- Explicit replans activate a new version and preserve history. Clearance is scoped to each version. A stored result can guide priorities or familiarity on a later route without being copied as course completion.

## Review disposition

The original [documentation audit](../../DOCUMENTATION_REVIEW.md), [issue register](../../review.txt), and [design transcript](../../transcript_temp.txt) provide historical context. Follow the documents linked in the component table and their shared contracts for v1.

| Review area | Where the disposition is recorded |
| --- | --- |
| Objective identity, links, depth, demonstrations, and removal of interim topic targets | [Shared contracts](AGENT-CONTRACTS.md#curriculum-and-objective-identity), R-001, R-013, R-014, R-036, R-037 |
| Pre-planning diagnostic and reduced repeated intake | [Scope](REQUIREMENTS.md#learner-experience) and [course-entry workflow](WORKFLOWS.md#goal-and-course-flow), R-002, R-003, R-004, R-031 through R-035, R-042, R-053, R-055 |
| Passing, coverage, scope, skips, evidence reuse, grading variance, support formats, access, retakes, disputes, accessibility, oversight, and question banks | [Assessment policy and integration acceptance](REQUIREMENTS.md#assessment-policy-v1), [engine specification](../../ASSESSMENT.md), and [Learning Context](../backend/app/agents/learning_context/AGENT.md), R-008 through R-012, R-015 through R-030, R-041, R-051, R-054 |
| Ownership, protocol, library, source approval, and excerpt handling | [Components](#components-and-ownership), [shared contracts](AGENT-CONTRACTS.md), and [teaching library](TEACHING-LIBRARY.md), R-005, R-006, R-038 through R-040, R-058, R-059 |
| Canonical documentation, historical files, and implementation scope | [Repository index](../../README.md), this document, and [workflow API](WORKFLOWS.md#api-actions), R-007, R-043 through R-050, R-052, R-056, R-057, R-060, R-061 |

R-045 is historically resolved. The ignored local note and empty nested ignore file have no role in the active design. The original scaffold placeholders mark work for implementation.

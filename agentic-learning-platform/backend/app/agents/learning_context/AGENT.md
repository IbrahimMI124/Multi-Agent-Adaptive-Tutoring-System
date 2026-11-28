# Learning Context

Learning Context is the private learner record maintained by the Orchestrator. An optional summarization helper can propose updates to the conversation summary. The Curator validates and persists them.

## Records

- Learner preferences include language, accessibility settings, explanation preferences, and explicit corrections. Preserve them as learner statements.
- Evidence entries link to an objective where applicable, curriculum version, course, and originating self-report, Tutor turn, or graded attempt. Store the kind, observed status, assistance when relevant, and correction references.
- The conversation summary records its session, covered turn range, and revision. The transcript and recent unsummarized turns remain available to reconstruct a session.
- The derived context view provides relevant structured evidence, recent observations, preferences, and the applicable summary. Course progress and pending activities are separately owned Orchestrator records that can accompany a snapshot.

## Inputs and outputs

| Operation | Input | Output |
| --- | --- | --- |
| Construct snapshot | Learner and version resolved by the Orchestrator, course or objective scope, target worker | Applicable preferences, evidence, and conversation context within that worker's scope |
| Propose summary | The prior summary and a bounded, ordered transcript range | Summary text with its base revision and last covered turn |
| Commit evidence or summary | Accepted report or summary proposal and originating references | Updated private record or an actionable validation issue |

A Tutor snapshot includes relevant course history, preferences, and recent turns. Assessment preparation receives scoped evidence and preferences for selecting appropriate tasks. Grading uses the saved snapshot and submission, with presentation preferences supplied only after judgment for feedback.

## Update and conflict rules

- Append accepted evidence immediately and retain provenance. Self-report can guide assignments, Tutor observations can guide instruction, and course-completion evidence supports clearance under the versioned assessment policy.
- Learner corrections can update preferences and invalidate an earlier self-report in the derived view. Contradictory academic evidence can flag an objective for review. History remains traceable.
- Refresh when 12 completed turns are unsummarized, or sooner when the configured context budget would be exceeded. Accept the summary only for the matching session and range. Apply it to the summary fields only. Read structured evidence from the latest committed view.
- Scope every worker snapshot to the owned session and assigned objectives. Keep private assessment keys in their attempt record.

See the [Orchestrator](../../../../../LEARNING_ORCHESTRATOR.md), [shared contracts](../../../../docs/AGENT-CONTRACTS.md), and [data schema](../../../../docs/DATABASE-SCHEMA.md).

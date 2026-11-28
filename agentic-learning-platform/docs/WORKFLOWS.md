# Learner workflows and API

The [shared contracts](AGENT-CONTRACTS.md) describe worker data. The routes below are the MVP learner interface. A browser served by the same app can expose goal entry, roadmap and syllabus, study, assessment, and results views.

## API actions

The demo uses seeded accounts. POST /auth/session signs in and creates an expiring, revocable server session using a same-origin session cookie. DELETE /auth/session signs out and revokes it. Authenticate learner actions, read ownership from the server session, and verify each referenced plan, course, session, attempt, or item. Learner mutations carry a stable request ID. Successful actions and recoverable issues include the next available learner action and, when blocked, a short reason.

| Action | Route | Expected input and result |
| --- | --- | --- |
| `preview_goal` | POST /goal-previews | Original goal and optional clarification. Return a preview ID and milestones or a clarification prompt. |
| `build_curriculum` | POST /curricula | Preview ID and chosen or default direction. Publish the first validated plan and return its roadmap. |
| `get_curriculum` | GET /curricula/{version} | Return the owned roadmap, course availability, and every syllabus, including locked courses. |
| `open_course` | POST /curricula/{version}/courses/{course_id}/sessions | Start or restore a session for an available course. Optionally choose to start as planned. Return a session ID and a familiarity prompt or the initial assignment. |
| `respond_to_familiarity` | POST /sessions/{id}/familiarity | Reply to the saved question ID or continue as planned. Return at most one follow-up or the objective assignments. |
| `teach_turn` | POST /sessions/{id}/turns | Learner message or response with the current exercise ID. An optional instruction requests a different approach. Return the reply, citations where applicable, active objective, and pending exercise or next action. |
| `finish_study` | POST /sessions/{id}/finish-study | Confirm the current saved assignment revision. Return the next required work or assessment availability. |
| `start_assessment` | POST /sessions/{id}/assessments | A permitted purpose and, for a targeted check, the allowed objective IDs. Return the attempt ID and its preparation state or public presentation. |
| `submit_assessment` | POST /attempts/{id}/submission | Final answers by item ID and required working. Return pending or the stored result with feedback. |
| `get_attempt` | GET /attempts/{id} | Return owned status and public presentation or available feedback. |
| `retry_assessment` | POST /attempts/{id}/retry | Reinvoke preparation or grading only for a saved retryable operation. Return the current status or result for the same attempt. |
| `abandon_assessment` | POST /attempts/{id}/abandon | Close an unsubmitted attempt and return to the current assignment. |
| `report_assessment_issue` | POST /attempts/{id}/issues | Item ID and reason. Return a tracked report reference for review by the maintainer. |
| `resume_session` | GET /sessions/{id} | Restore the current objective and strategy, recent turns, and exact pending activity. |
| `replan_curriculum` | POST /curricula/{version}/replan | Active version, reason, and updated preview ID for a changed goal. Return a validated replacement or a recoverable issue. |

## Goal and course flow

1. **Preview and publish.** Show the preview, then build the complete route using the Planner. Validate every objective and edge reference before publishing. With an active curriculum, use replan to publish a replacement. Activate it only if the expected prior version is still current. Close unsubmitted attempts on the prior version as superseded. Previously submitted attempts can finish grading for that version’s history. Older curricula and results remain viewable.
2. **Enter or resume a course.** Course availability comes from the active DAG and completion records. Maintain one study session per learner, version, and course in v1. Use a saved assignment on re-entry. On first entry, the optional familiarity check gets one initial prompt and at most one follow-up if a specific answer can change the assignment. Uncertain objectives keep their normal study assignment. The learner can continue as planned at either prompt.
3. **Teach.** The Curator chooses the first objective from the saved assignment and invokes the Tutor with an explicit start instruction when the learner begins it. A first incoming message can supply this trigger. Store the assignment revision, current exercise, and the teaching strategy. New turns act on the current revision and matching pending exercise. The Tutor can recommend readiness; completion of assigned work is a Curator decision.
4. **Offer assessment.** The learner confirms the current assignment via finish-study. For v1 this confirmation plus the saved assignment is the readiness signal; the formal attempt provides demonstrated evidence. When all required instruction is skipped, offer completion assessment directly. Extra checks need a specific decision they can change, such as whether to review an earlier concept. Once a course-completion series has outstanding mandatory objectives, a review finish can offer verification scoped to them.
5. **Apply assessment results.** Use [Assessment policy v1](REQUIREMENTS.md#assessment-policy-v1). Update evidence and completion transactionally, then present the next action. Use the missed objective and public feedback as a fallback review assignment when a Planner recommendation is unavailable. If preparation cannot produce a grounded item or adequate coverage, explain the pending course requirement and route the problem for revision.

## Attempt lifecycle

| State | Allowed transitions |
| --- | --- |
| `preparing` | Allocate the attempt before calling the Assessment Engine. Accept the first valid prepared result and transition to `open`. A temporary failure is retryable under the same ID. A permanent issue becomes `failed`. A superseded or explicitly abandoned unsubmitted attempt becomes `abandoned`. |
| `open` | Return the same saved presentation on resume. Freeze a final submission and transition to `grading`. Move to `abandoned` on explicit abandonment or when a replan supersedes the attempt’s curriculum. |
| `grading` | Retry from the saved snapshot and submission on temporary failure. Accept the first valid graded result and transition to `graded`. An irrecoverable issue becomes `failed` with a recovery reason. |
| `graded` | Show grading feedback. A completed but inconclusive outcome uses this state and may lead to a new verification attempt. |
| `abandoned`, `failed` | Return terminal state and the recovery action. Create a new attempt only after the relevant readiness and scope checks. |

Keep one attempt in preparing, open, or grading per learner and curriculum version. Use short database transactions to reserve an operation, save inputs, and commit accepted results. Call the worker between transactions. Same request ID plus same inputs returns the stored result or current pending state. Reusing it with different inputs yields a conflict. A simultaneous or late worker result can write only to its reserved version, session, or attempt after acceptance checks. Grade history can finish on its original version after a replan. The active route uses its own completion evidence.

Use bounded inline worker calls for v1. Give each in-flight reservation a deadline. A pending response allows a client to poll attempt status and explicitly retry after a retryable error or an expired reservation, including one left by a restart. Reserve again under the same operation and attempt IDs with their frozen inputs. Accept one applicable result for that operation.

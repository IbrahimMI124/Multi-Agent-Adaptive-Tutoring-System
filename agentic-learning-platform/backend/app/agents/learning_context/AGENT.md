# Learning Orchestrator

The Learning Orchestrator, also called the Curator, coordinates the learner’s work across the Planner, Tutor, Assessment Engine, and Learning Context.

## 1. Purpose

Guide each learner from a goal to a published curriculum and through teaching, assessment, review, and progression. The Orchestrator is the sole writer to the main database. It supplies task-specific learner context when it invokes or resumes the Tutor and Assessment Engine.

## 2. Responsibilities

- Coordinate goal preview, curriculum validation and publication, course entry, and explicit replanning.
- Track course access, assigned topics, session progress, the current pedagogical model, and pending work.
- Choose a pedagogical model for the current objective and decide when to switch it.
- Invoke the Planner for course-entry familiarity checks and targeted review recommendations.
- Prepare assessments through the Assessment Engine, persist attempts, and apply validated outcomes to progression.
- Record transcripts and accepted agent reports. Update Learning Context and supply relevant snapshots to subsequent calls.
- Restore interrupted work and apply retried results once.

## 3. Non-Responsibilities

| Area | Responsible component |
| --- | --- |
| Choose the curriculum, academic prerequisites, and course syllabi | Learning Planner |
| Teach assigned objectives using the selected pedagogical model | Tutor |
| Prepare assessments and grade submissions against saved criteria | Assessment Engine |
| Provide reusable educational resources and their provenance | Shared teaching library |

Learning Context is a record the Orchestrator maintains. This ownership supersedes the persistence assignment in the [Planner’s responsibilities table](LEARNING_PLANNER.md#3-non-responsibilities).

## 4. Supported Actions

| Action | When it is invoked |
| --- | --- |
| `preview_goal` | The learner describes or refines a goal. |
| `build_curriculum` | The learner proceeds from the preview to the full roadmap. |
| `open_course` | The learner starts an available course. |
| `respond_to_familiarity` | The learner answers the course-entry question or starts as planned. |
| `teach_turn` | The learner interacts with the Tutor during study or review. |
| `finish_study` | The learner finishes assigned study or review and wants to proceed. |
| `start_assessment` | The learner starts an available course assessment or targeted check. |
| `submit_assessment` | The learner submits answers to an open attempt. |
| `resume_session` | The learner returns to an existing session. |
| `replan_curriculum` | The learner changes their goal or explicitly requests a revised route. |

## 5. Input Contracts

The Orchestrator resolves learner identity from the authenticated session. Calls that can change state carry a stable request ID for retries. Course actions reference the relevant curriculum version and course.

| Action | Required inputs | Optional inputs |
| --- | --- | --- |
| `preview_goal` | Learner goal | Clarifications or interests |
| `build_curriculum` | Goal preview reference | Approved university source references |
| `open_course` | Curriculum version and course ID | Preference to check familiarity or start as planned |
| `respond_to_familiarity` | Session ID and either the pending question ID with its reply or a choice to start as planned | Additional learner explanation |
| `teach_turn` | Session ID and learner message or exercise response | Request for a different kind of explanation |
| `finish_study` | Session ID and completed study or review reference | Learner request to review further |
| `start_assessment` | Session ID and available assessment purpose | Target objective IDs for a targeted check |
| `submit_assessment` | Attempt ID, answers, and any supporting work required by the questions | — |
| `resume_session` | Session ID | — |
| `replan_curriculum` | Active curriculum version and reason. A changed goal requires an updated preview. | Approved university source references |

## 6. Output Contracts

| Action | Learner-facing output |
| --- | --- |
| `preview_goal` | Preview reference, interpreted goal, and milestones, or a focused clarification request |
| `build_curriculum` | Published curriculum with its version, course availability, and inspectable syllabi |
| `open_course` | Session ID, course objectives, and either a familiarity prompt with its ID or assigned starting work |
| `respond_to_familiarity` | A scoped follow-up with its ID or the resulting study, review, and skip assignments |
| `teach_turn` | Tutor reply, current objective, and the supported next action |
| `finish_study` | Assessment availability or the work that is still due |
| `start_assessment` | Saved attempt ID, questions, and instructions for the learner |
| `submit_assessment` | Pending status or graded outcome, feedback, and the next action |
| `resume_session` | Current objective, recent conversation, and saved exercise, question, or attempt where applicable |
| `replan_curriculum` | Published replacement curriculum and reason for the revision |

When an action is blocked, return a reason and the available recovery action.

### Agent reporting protocol

Each completed call returns a structured result. The Orchestrator associates it with the originating request, curriculum version, course, and turn or attempt ID as applicable.

| Report | Required content | Use |
| --- | --- | --- |
| Tutor turn | Learner-facing reply and an observation list, which can be empty. Each observation identifies the relevant objective, turn or exercise, and the help used where applicable. | Save the turn and use observations to adjust teaching or decide whether a check would help. |
| Assessment prepared | Exact questions, objective coverage, and grading criteria | Save the assessment before presenting it. |
| Assessment graded | Outcome, feedback, and evidence for each assessed objective | Record the evidence and apply the progression policy. |

Tutor assistance can be recorded as independent work, work after a hint, or work following a worked example. Use those distinctions when judging readiness for assessment.

## 7. Internal Workflow

1. **Publish the route.** Call the Planner’s `preview_goal` and `build_curriculum`. Verify that the DAG is acyclic, references valid courses, and has a syllabus for every node. Publish the full curriculum before offering familiarity checks. Validate a replacement before making it active.
2. **Start or resume study.** Check course availability and restore any pending work. At course entry, use `check_familiarity` when requested. Save the pending question and the topics it covers. Use the Planner’s recommendation to assign topics and pick a starting objective.
3. **Teach one turn.** Save the learner input. Send the Tutor the objective, selected pedagogical model, relevant learner evidence and preferences, and a summary with recent turns. Accept and save the completed reply and report. Apply meaningful observations before choosing the next action.
4. **Prepare and grade.** Send the Assessment Engine the objectives, purpose, and relevant learner context. Save the returned questions and criteria under an attempt ID. Save the learner’s submission before requesting a grade. Send the saved questions, criteria, and submission along with any learner context relevant to feedback.
5. **Apply the outcome.** Commit a valid grade, objective evidence, and the resulting progress change together. A pass can make eligible successor courses available. A gap leads to targeted review using the Planner’s `recommend_review` where useful. Send the Tutor the missed objectives and learner-facing feedback for remediation.
6. **Keep the next session current.** Apply accepted evidence as it arrives. Refresh the conversation summary at a configured threshold and track the last turn it covers. Load the latest applicable context whenever the Tutor or Assessment Engine is spawned or resumed.

## 8. Decision Rules

- **Choose and carry forward a teaching model.** Use the active objective, learner preferences, and recent evidence. An explicit learner request or repeated difficulty can justify a switch. Record the reason and the observation or request that prompted it.
- **Use evidence for the decision it supports.** Self-reports guide familiarity recommendations. Tutor observations guide instruction or motivate a check. Graded results support demonstrated knowledge and progression. Conflicting evidence can flag a specific objective for review while retaining both records.
- **Assess at useful points.** Use the course-end assessment as the normal completion check. Offer an earlier targeted check when it can change the next assignment. Use a fresh task to verify the same objective after review. Carry forward evidence across attempts when the rubric permits it.
- **Grade against saved criteria.** Learner context can inform question selection and feedback. Use the rubric saved with the attempt to determine the grade. Keep the active assessment and its answer key scoped to the attempt.
- **Apply prerequisite rules consistently.** Evaluate every required prerequisite branch before opening a dependent course. The threshold for a self-reported skip to clear a prerequisite is open. The proposed MVP default lets learners skip familiar instruction and proceed directly to an assessment for clearance.
- **Make decisions traceable.** Save short reasons for strategy switches, targeted review, and unlocks. Show the learner the next step and a concise reason when the route changes.

## 9. Dependencies

- **Learning Planner.** Provides goal previews, curricula, familiarity decisions, and review recommendations. [Planner action contracts](LEARNING_PLANNER.md#4-supported-actions)
- **Tutor and Assessment Engine.** Receive scoped inputs from the Orchestrator and return results that conform to the reporting protocol.
- **Main database.** Stores curriculum versions, course progress, transcripts, Learning Context, and assessment attempts. The Orchestrator commits updates to these records.
- **Shared teaching library.** Supplies reusable teaching material and source references. Learner-specific records belong in the main database.

## 10. Failure Handling

| Situation | Handling |
| --- | --- |
| A generated curriculum is invalid or a replan fails. | Return the issue for correction or retry. Continue serving the last published curriculum when available. |
| A reply or report is malformed or references an invalid course, objective, turn, or attempt. | Return a recoverable issue for that action. Apply progress changes only from validated results. |
| A Tutor call is interrupted. | Resume from the saved learner input and exercise. Associate a retry with the original turn. |
| Grading times out. | Keep the saved attempt pending and retry under the same attempt ID. A pending status carries no grade. |
| A submission or agent result is duplicated. | Return the recorded result or pending state for that ID. Apply context and progression effects once. |
| A result arrives after a replan or a newer context update. | Keep it tied to its originating version or attempt. Apply it to current work only after checking that it still applies. A summary update covers only its recorded transcript range. |
| The grading evidence is inconclusive. | Keep progression pending and offer an appropriate follow-up or reassessment. |

## 11. Acceptance Criteria

1. The learner sees a goal preview, then a validated full curriculum, then an optional course-entry familiarity check.
2. A Tutor session can resume on the same exercise with the correct objective, strategy, and current context. A report of difficulty can trigger a documented change of teaching model.
3. The Assessment Engine receives scoped context and can resume grading using the same saved questions, criteria, and submission.
4. Objective-level results update Learning Context. A missed objective can produce targeted Tutor review, and a pass opens only eligible successors.
5. Retrying a submission records one outcome and applies one progression change. A timeout keeps the attempt pending, and an older result cannot accidentally progress a replacement curriculum.
6. A delayed summary does not overwrite newly recorded assessment evidence. The next relevant Tutor or Assessment Engine invocation receives the updated evidence.

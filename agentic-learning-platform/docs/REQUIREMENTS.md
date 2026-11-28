# MVP requirements

## Learner experience

1. Accept a free-form goal and present a short preview with milestones. Publish the complete course DAG and every syllabus before asking about familiarity.
2. Allow the learner to inspect all syllabi and open any course whose required predecessors are complete. Offer a short, optional familiarity conversation on first entry.
3. Resume the exact exercise, question, or assessment in progress. Allow the learner to ask for another teaching approach.
4. Provide a normal course-end assessment or fast-track assessment for familiar instruction, with objective-level feedback and targeted remediation.
5. Explain review assignments, blocked courses, and unlocks. Provide a way to report a suspected assessment error.
6. Support explicit replans. Preserve earlier curricula and evidence as history and publish a replacement only after validation.

This course project demonstrates one complete seeded route with two courses and a prerequisite, alongside the Planner's free-form goal interface. A baseline diagnostic before planning, a scored or adaptive placement test, a shared question bank, external proctoring, certification, and an educational effectiveness study are deferred. The app presents results as learning feedback from unproctored work.

## Assessment policy v1

The project owner approves revisions to this policy. The Orchestrator supplies the saved version and exact rules to the Assessment Engine, which judges the attempt. The Orchestrator applies the progression rules below.

- **Coverage.** Every in-scope objective needs at least one aligned item and a required criterion. Objectives that require applying, analyzing, deriving, or explaining need a criterion that evaluates the learner's own answer or working. A completion attempt covers all mandatory objectives. Recommended and optional objectives remain practice material in v1. One item per objective is an MVP floor, and reports disclose when it is the only item.
- **Criteria.** Use `met`, `partially_met`, `not_met`, `no_response`, or `unscorable`. A blank on an explicit final submission is `no_response`. An unfinished draft or abandoned open attempt yields no grade. Missing required reasoning caps an otherwise correct criterion at `partially_met`. A wrong answer receives the judgment its rubric supports.
- **Objective status.** For an assessed objective with required criteria, any unscorable required criterion makes it `inconclusive`. Otherwise all required criteria met means `demonstrated`; at least one met or partially met means `partial`; the rest are `not_demonstrated`. An objective outside scope is `not_assessed`. Missing required criterion coverage is a preparation or snapshot error.
- **Attempt outcome.** Evaluate a nonempty scope. Any `not_demonstrated` or `partial` objective yields `not_passed`, even if another objective is inconclusive. Otherwise any `inconclusive` or `not_assessed` objective yields `inconclusive`. All demonstrated yields `passed`. Show the categorical outcome and feedback by objective.
- **Completion series.** Reserve the series with its first course-completion attempt and freeze its policy version. If the attempt closes before grading, allow a new full attempt in that series. After the full attempt is graded, verify outstanding mandatory objectives with fresh tasks under the same policy. Within that learner, course, curriculum version, and series, retain demonstrated evidence from accepted attempts. Mark the course complete once every mandatory objective has valid demonstrated evidence in the series. A targeted check guides study. Evidence from another curriculum version can inform familiarity or instruction. Course clearance is scoped to the new version.
- **Skip and unlock.** A self-report can skip instruction. If all required instruction is skipped, offer the normal assessment immediately. Skipping part of a course leaves the missing work assigned and uses the same completion criteria. An available course requires every predecessor course to be complete. A later formative conflict flags review and preserves historical completion.
- **Freshness and feedback.** Exclude identical and materially repeated tasks already seen for the same objective, including available overlapping Tutor exercises. Use a new prompt, scenario, or values that still tests the same skill. Some objectives have the same valid answer across prompts. If a defensible fresh task cannot be prepared, surface the issue for revision. The learner sees actionable feedback after grading. Full answer keys stay private in v1.
- **Attempts.** Keep one active formal attempt per learner in the active curriculum. Allow fresh retakes after review, with no fixed time limit in the demo. A technical retry reuses the same attempt and frozen submission. During an open or grading attempt in the active curriculum, the learner resumes tutoring after that attempt closes. They can abandon an open attempt to return to instruction. A submitted attempt is finalized through grading or recorded failure.
- **Reported errors.** Record a report against the attempt item. A designated maintainer can confirm an issue and invalidate the affected evidence for future clearance, keeping an audit record. Offer replacement work in the completion series when needed. A course already marked complete is flagged for review rather than automatically revoked.

## Integration acceptance

- Given a two-course route, verify that the second course becomes available only when the first meets its mandatory objectives. Confirm a self-report alone and a passing targeted check leave clearance pending.
- Teach an objective after a hint, report the assistance, and switch strategy on a meaningful signal. Demonstrate that fresh independent work and a graded result enter context with different provenance.
- Fail one objective in a completion attempt, report a precise gap, and pass a fresh verification of only that objective. Confirm the accepted earlier evidence combines within the same series and clears the course once.
- Test mixed failed and inconclusive objectives, an empty required criterion set, a missing required rationale, and a final blank. Use fixed grading fixtures for these policy tests.
- Test duplicate and concurrent requests, lost responses, same request ID with different input, a timeout, a stale summary, and a late grade after replan. Check that stored evidence and progress effects apply once.
- Resume the same pending assessment from a saved snapshot. Test deterministic scoring with identical inputs, replay of the first accepted result, and rubric grading against sample responses. Independent open-answer worker invocations may vary.
- Verify that the learner and Tutor see only authorized presentation and feedback, and an authenticated learner cannot read or mutate another learner's state.

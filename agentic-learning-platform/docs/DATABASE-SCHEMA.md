# MVP data schema

The Orchestrator owns main database writes through its repository layer. Use a relational model with versioned curriculum records, private attempts, and append-only evidence. A structured field can be stored as validated JSON where it serves one owned record, such as the grading snapshot.

| Record | Key and important fields |
| --- | --- |
| Learner and authenticated session | Learner ID, account reference, credential verifiers, and revocable session ID with expiry |
| Goal preview | Learner ID, preview ID, goal text, interpretation, milestones, selected direction |
| Curriculum and published plan | Learner ID and version, preview reference, publication state, active pointer, previous version, reasons and sources |
| Course, outcome, topic, objective, milestone, edge | Version plus type-specific ID or edge pair; fields and references from [shared agent contracts](AGENT-CONTRACTS.md#curriculum-and-objective-identity) |
| Course progress and assignments | Learner, version and course; objective assignments, assignment revision, familiarity decision, review state, completion status and evidence references |
| Study session | Session ID and unique learner/version/course tuple; current objective, strategy and version, pending exercise or question, last turn and row revision |
| Transcript and worker reports | Session and sequential turn ID; learner input, accepted reply, activity and worker report reference |
| Learner profile and summary | Learner ID, explicit preferences and accessibility options; versioned summary with session, transcript cursor, and covered range |
| Evidence | Learner, version, course and objective where relevant, evidence kind, originating turn, response or attempt, status, assistance if applicable, correction or review reference |
| Assessment attempt | Attempt ID, owner, version, course, purpose, optional completion series ID, policy version, lifecycle state, public presentation, private snapshot, frozen submission, accepted grade |
| Completion series | Learner, plan version, course and policy version; first completion attempt, outstanding objectives, eligible evidence, terminal completion if earned |
| Operation | Learner, action and stable request ID, input fingerprint, target revision, reservation deadline, status, accepted output or issue, and retry references |
| Decision or correction | Action, scoped references, reason, and date; corrections can invalidate specific assessment evidence while preserving its origin |
| Assessment issue | Learner, attempt and item ID, description, review status, and any correction reference |

Use stable foreign keys and unique constraints for accepted turns, submissions, and effects. Enforce one active attempt per learner/version, one active completion series per learner/course/version, and one active curriculum per learner. Apply grade acceptance, objective evidence, and resulting course completion in one transaction. A background summary has its own revision and cursor, so its write cannot overwrite structured evidence.

The public attempt representation contains the presentation, status, and allowed feedback. Keep private snapshot fields behind the assessment worker path. Scope operations and joins to the authenticated learner. Use seeded demo accounts and resettable demo data for v1. For a pilot with real learners, the maintainer must be able to delete owned records, including transcripts, evidence, attempts, and per-learner caches, through the Orchestrator repository.

Reusable teaching entries live behind the [shared library interface](TEACHING-LIBRARY.md), outside the private learner records.

# Learning Planner

## 1. Purpose

Turn a learner’s goal into a complete curriculum with a directed acyclic graph (DAG) of courses and an ordered syllabus for every course. Use relevant university materials to inform academic decisions, then adapt the work assigned to the learner through familiarity conversations as they reach each course.

## 2. Responsibilities

- Interpret goals stated in the learner’s own words, from mastering a skill to preparing for a career or research area. Offer a broad starting route when the learner is still exploring.
- Produce a short preview of the proposed direction and milestones.
- Build the course DAG and justify each prerequisite with the skill it provides to downstream work.
- Plan each course’s topics, sequence, depth, priorities, sources, and expected outcomes.
- Recommend what to study, review, or skip based on familiarity with the planned coursework. Use later assessment evidence to recommend targeted review.
- Produce a revised curriculum when the Orchestrator requests a replan.

## 3. Non-Responsibilities

| Area | Responsible component |
| --- | --- |
| Present the roadmap, manage course access, and enforce progression | Learning Orchestrator |
| Persist learner context, reported familiarity, and assessment evidence | Learning Context |
| Create and deliver lesson explanations, examples, and practice | Tutor |
| Score assessments and determine which objectives the learner demonstrated | Assessment Engine |

## 4. Supported Actions

| Action | When the Orchestrator invokes it |
| --- | --- |
| `preview_goal` | The learner describes or refines a goal. |
| `build_curriculum` | The preview is ready and the learner needs the full course DAG and syllabi. |
| `check_familiarity` | The learner opens a course or responds to a familiarity question about that course. |
| `recommend_review` | An assessment indicates a possible gap that affects the learner’s work. |
| `replan_curriculum` | The learner changes their goal or explicitly requests a revision to an unsuitable route. |

## 5. Input Contracts

| Action | Required inputs | Optional inputs |
| --- | --- | --- |
| `preview_goal` | Learner’s goal in their own words | Clarifications about interests, intended direction, or areas they are exploring |
| `build_curriculum` | Goal preview, including the interpreted goal and milestones | Relevant references supplied from approved university sources |
| `check_familiarity` | Published curriculum or reference to it, plus the course identifier | Prior familiarity or assessment evidence, the learner’s reply with the question it answers, or a request to start the course as planned |
| `recommend_review` | Published curriculum, assessed course identifier, and assessment evidence tied to learning objectives | Current study recommendations and other relevant objective-level evidence |
| `replan_curriculum` | Current curriculum and reason for revision. A changed goal requires an updated goal preview. | Relevant references supplied from approved university sources |

## 6. Output Contracts

| Action | Success output | Required content |
| --- | --- | --- |
| `preview_goal` | Goal preview | Original goal, interpreted direction, and proposed milestones. Include possible directions when the learner is exploring. |
| `build_curriculum` | Complete curriculum | Goal preview, course nodes, prerequisite links, and a syllabus for each node. |
| `check_familiarity` | Familiarity result | Course identifier and either a question with the topic identifiers it covers or study recommendations for the course. |
| `recommend_review` | Review result | Assessed course identifier and study recommendations naming the courses and topics to revisit. An empty list indicates that the evidence supports no targeted change. |
| `replan_curriculum` | Replacement curriculum | The same structure as a complete curriculum, with the reason for the revision. |

A **course node** includes an identifier, title, purpose on the route, expected outcomes, and source references. A **prerequisite link** identifies the earlier and later courses, the skill the earlier course provides, and the rationale and sources for the dependency. The rationale identifies when a dependency is the Planner’s inference.

A **syllabus** identifies its course, lists topics in teaching order, and defines how the learner will demonstrate the required outcomes. Each topic has an identifier, title, priority, depth, and source references. Topics considered and deemed out of scope can be recorded with that priority outside the teaching sequence.

A **study recommendation** identifies the course and relevant topics, gives an action of study, review, or skip, and states the reason and kind of evidence. A skip based on the learner’s account is recorded as self-reported. An unsuccessful action returns a **planning issue** with a reason, a recovery action for the Orchestrator, and the affected input or course when known.

## 7. Internal Workflow

1. Interpret the goal and propose milestones. Return the short preview for the Orchestrator to show first.
2. Retrieve relevant university materials. Work backward from the milestones to draft the course DAG, including shared foundations and possible branches.
3. For every course, select and sequence the topics required by the route. Check that each upstream syllabus covers what its outgoing prerequisite links promise. Revise the draft if topic planning exposes a missing dependency.
4. Return the complete curriculum for validation and publication. The learner can inspect every syllabus, including those for courses whose prerequisites are pending.
5. When the learner reaches a course, use its syllabus to offer a short familiarity conversation. Recommend the next work based on what the learner reports having encountered, studied, or used. If they choose to start as planned, use the course’s assigned sequence.
6. When assessment evidence identifies a gap, recommend review of the relevant topic. A goal change or explicit replan request starts a new curriculum planning pass.

### Worked example

For “I eventually want to do research in quantum computing,” a theory-oriented route could include **Linear Algebra** and **Probability** feeding into **Introduction to Quantum Computing**. This is an example branch. Illinois’s course lists probability and specific linear algebra skills, while MIT’s *Quantum Computation* syllabus lists linear algebra as its academic prerequisite. The Planner chooses dependencies that suit the outcomes of the course it actually proposes. [Illinois Quantum Computing syllabus](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites) · [MIT Quantum Computation syllabus](https://ocw.mit.edu/courses/18-435j-quantum-computation-fall-2003/pages/syllabus/)

For a course modeled on the Illinois outcomes, the Linear Algebra syllabus could cover matrix operations and complex vector spaces, then inner products, then tensor products. Cramer’s rule could be out of scope for that route. If the learner reports knowing matrices and inner products but has never used tensor products, the next assignment can focus on tensor products. [Illinois prerequisite topics](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites) · [MIT Linear Algebra syllabus](https://ocw.mit.edu/courses/18-06sc-linear-algebra-fall-2011/pages/syllabus/)

## 8. Decision Rules

- **Use the Planner’s academic judgment.** The Planner chooses the courses, links, topic order, and depth based on the goal and university materials. The application verifies structural conditions such as valid course references and acyclicity. A structurally valid graph can still have an academic gap.
- **Justify blocking prerequisites.** Name the skill a later course relies on and include it as mandatory in the earlier course at the needed depth. Treat a university’s teaching order as evidence about sequencing and use downstream needs to justify a blocking link.
- **Build the full curriculum before checking familiarity.** Course-entry recommendations can change the learner’s assignments within the published curriculum.
- **Set priority by purpose.** A mandatory topic is needed for a milestone or required downstream work. A recommended topic improves understanding or practice. An optional topic supports further exploration. An out-of-scope topic is omitted from the teaching sequence.
- **Use familiarity to avoid repetition.** Assign the required sequence when the coursework is new. When the learner knows some topics, keep the missing topics and suggest review where useful. A learner who knows a mandatory topic can skip its instruction.
- **Treat the skip threshold as a pending product decision.** The proposed default is a reversible self-reported skip when the learner clearly covers the required topics, with a targeted follow-up when a mandatory topic is unclear. The evidence needed for that skip to open a dependent course is still open.

## 9. Dependencies

- **Approved university sources.** Official course catalogs, syllabi, lecture notes, and accessible teaching materials, including textbook sections available through those sources.
- **Shared resource library and source lookup.** Search the library first and look for additional approved university material when needed. The library can reuse source links, topic tags, and relevant excerpts across learners.
- **Learner context and assessment evidence.** Learning Context supplies relevant learner records. Personal plans, diagnostic answers, and progress are private to the learner. Objective-level assessment results support recommendations to revisit specific topics.
- **Application validation.** Before publication, the application checks the curriculum for cycles and missing course references.

## 10. Failure Handling

| Situation | Handling |
| --- | --- |
| The goal is missing or too ambiguous to propose a useful direction. | Return a focused request for clarification. A broad goal with an identifiable direction can start with a broad preview. |
| Available university material cannot support part of the proposed route. | Look for additional approved sources. If the gap persists, return the affected course, topic, or dependency as a planning issue so the draft can be reviewed before publication. |
| The draft contains a cycle, a missing course reference, or an uncovered prerequisite skill. | Revise and recheck the draft. If the issue persists, return the affected dependencies for review and require a valid curriculum before publication. |
| A required input is missing or refers to a course outside the given curriculum. | Return an input issue naming the missing input or invalid reference. |
| The learner’s familiarity is unclear. | Return a targeted familiarity question when the answer can change the recommendation. Keep the required work available while familiarity is unclear. |
| Assessment evidence is inconclusive. | Return a review result with an empty recommendation list. |
| Source lookup is unavailable. | Use cached university materials when they provide enough coverage. Otherwise return a temporary failure the Orchestrator can retry. |
| Curriculum generation is unavailable. | Return a temporary failure the Orchestrator can retry. For a failed replan, the Orchestrator can continue serving the last published curriculum. |

## 11. Acceptance Criteria

1. A learner can enter a broad goal and see a preview first. The full DAG and an inspectable syllabus for every course are available before any familiarity conversation.
2. Required courses, prerequisite links, and topics have traceable university sources and a rationale tied to the learner’s route.
3. Every dependency points to a course in the graph, the graph is acyclic, and each prerequisite skill is covered at the required depth in the relevant syllabus.
4. Two learners opening the same course can receive different study recommendations based on their familiarity with its topics. Either learner can start with the planned coursework.
5. Self-reported skips are distinguishable from assessment results, and later objective-level evidence can bring back a specific topic for review.
6. An explicit replan produces a complete replacement curriculum through the same planning and validation process.

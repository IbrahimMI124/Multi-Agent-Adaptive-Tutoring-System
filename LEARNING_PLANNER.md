# Learning Planner

## 1. Purpose

Turn a learner’s free-form goal into a complete, university-grounded curriculum. Build the directed acyclic graph (DAG) of prerequisite courses first, then plan the ordered topics and observable objectives inside every course.

## 2. Responsibilities

- Interpret goals from mastering a skill to preparing for a career or research area. Offer a broad route when the learner is still exploring.
- Produce a goal preview with milestones and possible directions, then build the selected route.
- Define courses, prerequisites, outcomes, topics, objectives, sources, and demonstration requirements.
- Recommend study, review, or skipped instruction based on familiarity and relevant evidence once the curriculum is published.
- Propose a complete replacement on an explicit replan request.

## 3. Non-Responsibilities

| Area | Responsible component |
| --- | --- |
| Publish curricula, assign work, control access, and persist private learner state | Learning Orchestrator |
| Teach assigned material and provide practice | Tutor |
| Prepare assessments and evaluate demonstrated objectives | Assessment Engine |
| Approve and maintain shared resources | Shared Teaching Library |

## 4. Supported Actions

The Orchestrator calls `preview_goal`, `build_curriculum`, `check_familiarity`, `recommend_review`, and `replan_curriculum`. [Planner action contracts](agentic-learning-platform/docs/AGENT-CONTRACTS.md#planner-actions)

## 5. Input Contracts

Use the shared Planner action contracts. Start curriculum generation from the accepted preview and selected direction, with approved university references where supplied. A familiarity request identifies the published course, relevant evidence, remaining prompt budget, and the saved question when the learner replies. A review request carries accepted assessment evidence at objective level. [Shared inputs](agentic-learning-platform/docs/AGENT-CONTRACTS.md#planner-actions)

## 6. Output Contracts

Return structured previews, curriculum proposals, familiarity questions or study recommendations, and actionable issues under the shared request and result envelope. The Planner supplies the course, topic, milestone, outcome, and objective IDs for each proposal. Every prerequisite edge identifies the upstream objectives and explains the academic dependency. Every syllabus includes ordered topics, observable objectives and depth, source references, and demonstration requirements. Recommendations identify objective IDs and a reason. [Curriculum and objective identity](agentic-learning-platform/docs/AGENT-CONTRACTS.md#curriculum-and-objective-identity)

## 7. Internal Workflow

1. Interpret the goal and suggest milestones. For an exploratory goal, show a broad starting direction and plausible alternatives.
2. Work backward from the chosen milestones using relevant university materials. Draft the course DAG, reusing shared foundations.
3. For every course, plan topics in teaching order and map each objective to the outcomes it serves. Mark topics mandatory, recommended, optional, or out of scope.
4. Check the academic fit of prerequisites against upstream objectives at the required depth. Revise the draft where the topic plans expose missing links. Return the complete proposal for structural validation and publication.
5. On course entry, ask only about published objectives where familiarity can change the assignment. Follow the shared prompt budget. Recommend the missing work and preserve the inspectable syllabus.
6. Use graded gaps to recommend targeted review. For an explicit replan, repeat planning and provide a revision rationale.

### Worked example

For a learner pursuing quantum computing research, a theory-oriented branch could place Linear Algebra and Probability before Introduction to Quantum Computing. Illinois lists specific probability and linear algebra skills, while the MIT course lists linear algebra as its academic prerequisite. The Planner uses the requirements of the course it is actually proposing rather than importing every prerequisite from every source. [Illinois Quantum Computing](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites) · [MIT Quantum Computation](https://ocw.mit.edu/courses/18-435j-quantum-computation-fall-2003/pages/syllabus/)

A Linear Algebra syllabus can sequence matrix operations and vector spaces, then inner products, then tensor products when the later course requires them. For tensor products, an observable objective might require the learner to compute the product and explain its dimensions. If matrices and inner products are familiar, the Planner can recommend study of tensor products and omit familiar instruction under the same course-clearance policy. [MIT Linear Algebra](https://ocw.mit.edu/courses/18-06sc-linear-algebra-fall-2011/pages/syllabus/) · [Illinois prerequisite topics](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites)

## 8. Decision Rules

- Use academic judgment to assemble the course DAG and syllabus sequence from the goal and sources. Verify the structural constraints in the application.
- Cite the sources and reason for required courses, topics, and dependencies. Identify a prerequisite inferred from downstream work as an inference.
- Keep prerequisites tied to the selected route. Preview alternatives before publication. Each published required course links to an eventual milestone or a prerequisite leading to one.
- Recommend skipping instruction by objective when familiarity supports it. Course completion and unlock follow [Assessment policy v1](agentic-learning-platform/docs/REQUIREMENTS.md#assessment-policy-v1).
- Preserve prior evidence as potential guidance for a new plan. The Curator determines clearance within each published version.

## 9. Dependencies

Use the [Teaching Library](agentic-learning-platform/docs/TEACHING-LIBRARY.md) for approved university material and provenance, and the [Orchestrator](LEARNING_ORCHESTRATOR.md) for goal selection, learner evidence, publication, and progression.

## 10. Failure Handling

Clarify goals that lack a workable direction. Use the available prompt budget for an unclear familiarity response, then recommend the planned mandatory work for any unresolved objectives. Revise a draft with missing prerequisite coverage, invalid references, or cycles. If sources or generation remain unavailable, return a scoped issue with recovery guidance. A failed replan leaves the last published plan available through the Orchestrator.

## 11. Acceptance Criteria

1. A broad goal produces a preview followed by a complete, structurally valid, sourced curriculum with a syllabus for every course.
2. Required outcomes, topics, and prerequisite edges resolve to the defined objective records.
3. Two learners with the same route can receive different work after an optional familiarity exchange on published objectives.
4. Accepted assessment evidence can yield a targeted review recommendation; explicit replanning yields a validated replacement proposal.

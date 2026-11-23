# Learning Planner Specification

The Learning Planner turns a learner’s goal into a complete curriculum. It first builds a directed acyclic graph (DAG) of courses and their prerequisites. It then plans the topics inside every course. Once the learner has the curriculum, short conversations about the actual coursework determine what they need to study, review, or skip.

The Planner uses university materials as inputs and exercises judgment about how to combine them. The application checks the structure of its output. The Planner is responsible for the academic rationale.

## Learner goals and the first preview

Learners can describe an ambition in their own words. Valid goals include preparing for quantum computing research, getting good at software architecture, and mastering performance optimization. The Planner proposes milestones that give the ambition a workable path. A learner who is still exploring can start with a broad route and decide on a specialization later.

The learner sees a short preview of the direction and milestones first. The Planner then makes the full course roadmap and every course’s topic plan available before asking about familiarity. A learner can inspect a later course’s syllabus even while its prerequisites are pending.

## University sources

The Planner draws from official course catalogs, syllabi, lecture notes, and accessible teaching materials from universities approved for the project. It can combine material from several universities, including textbook sections available through those sources. It looks in the shared resource library for relevant material and can look up additional approved sources when needed. The library can reuse source links, topic tags, and relevant excerpts across learners. Personal plans, diagnostic answers, and progress belong to the learner’s own records.

Course descriptions and published prerequisites help establish what belongs in the graph. Syllabi and teaching materials provide the detail needed to plan topics, sequence, and depth inside a course. When sources differ, the Planner chooses prerequisites that fit the downstream course it is proposing. For example, MIT lists linear algebra for its *Quantum Computation* course, while Illinois also lists specific probability skills. Each requirement makes sense in the context of its own course. [MIT Quantum Computation syllabus](https://ocw.mit.edu/courses/18-435j-quantum-computation-fall-2003/pages/syllabus/) · [Illinois Quantum Computing syllabus](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites)

The roadmap should let the learner see why a course, prerequisite, or required topic is there. Give the relevant source and explain how it applies to this plan. If the Planner infers a dependency, its explanation should identify the downstream work that needs the earlier skill.

## Planning the course DAG

The Planner works backward from the goal and its milestones to identify the courses needed to reach them. It can create multiple branches, reuse common foundations, and show possible directions for a learner who is still exploring.

- A **node** is a course-sized unit, such as Linear Algebra or Quantum Physics I. It has a purpose, expected outcomes, and a reason for being on this route.
- A **prerequisite link** identifies an earlier course and the skill it provides to a later course. A topic appearing earlier in a published syllabus supports a teaching sequence. A blocking prerequisite calls for a reason that skill is needed for the downstream work.

Before publication, the application checks that every dependency refers to a course in the plan and that the graph is acyclic. The quality of the chosen courses and dependencies depends on the Planner’s interpretation of the goal and sources. A graph can pass the structural checks and still contain an academic gap.

## Planning the topics inside each course

Every course in the DAG gets an ordered syllabus. It specifies the learning outcomes, topics, depth, source references, and how the learner will demonstrate the required outcomes. The Planner chooses the scope by considering both the learner’s goal and the skills later courses expect from this one.

| Priority | Meaning for the route |
| --- | --- |
| Mandatory | The learner needs the skill to reach a milestone or do required downstream work. |
| Recommended | The topic improves the learner’s understanding or practice and can be skipped without blocking progress. |
| Optional | The topic supports an extension or a direction the learner may choose to explore. |
| Out of scope | The topic is omitted from the assigned syllabus for this route. Its relevance can change if the goal changes. |

The Planner checks the syllabus against the DAG before publishing. The upstream syllabus must include each skill promised by a prerequisite link as mandatory and cover it at the depth the later course needs. If the topic plans expose a missing dependency, the Planner revises the draft. The Tutor uses the resulting objectives to prepare explanations, examples, and practice as the learner works through a course.

## Familiarity checks during learning

When a learner opens a course, the app shows its planned topics and offers a short conversation about that coursework. The learner can say what they have encountered, studied, or used in practice, or start the course as planned. The question focuses on the topics most likely to change the assigned work.

| What the learner reports | How the Planner can adapt the work |
| --- | --- |
| The coursework is new. | Assign the required sequence. |
| Some topics are already familiar. | Keep the missing topics and suggest review where familiarity is partial. |
| The required topics are already familiar. | Consider a skip and keep the course available to revisit. |

A topic’s priority and the learner’s familiarity answer different questions. A mandatory topic can be skipped by someone who already knows it. The complete curriculum stays available while the learner’s assignments change. An assessment later in the journey can reveal a gap and bring back the specific topic for review. An explicit replan can produce a revised curriculum when the goal changes or the original route is unsuitable.

**Proposed skip policy.** Record a clear claim of familiarity as a reversible self-reported skip and ask one targeted follow-up if a mandatory topic is unclear. Keep that status distinct from passing an assessment. The exact evidence required for a self-reported skip to open a dependent course is still a product decision.

## Worked example

A learner says, “I eventually want to do research in quantum computing.” The preview shows foundations, introductory quantum computing, and later specialization. An excerpt of the full DAG for a theory-oriented route has **Linear Algebra** and **Probability** feeding into **Introduction to Quantum Computing**. For a course based on Illinois’s published outcomes, the Linear Algebra syllabus could sequence matrix operations and complex vector spaces, then inner products, then tensor products. It could draw on MIT materials for foundational topics and leave Cramer’s rule out of scope for this route. [Illinois Quantum Computing prerequisites and outcomes](https://courses.grainger.illinois.edu/CS598QC2/fa2026/#prerequisites) · [MIT Linear Algebra syllabus](https://ocw.mit.edu/courses/18-06sc-linear-algebra-fall-2011/pages/syllabus/)

When the learner opens Linear Algebra, they say they have already studied matrices and inner products but have never used tensor products. The Planner can keep tensor products in the assigned work, shorten familiar parts, and show why the remaining work matters for the later quantum computing course.

## Responsibilities at the Planner boundary

- The **Learning Orchestrator** presents the preview and curriculum, coordinates course entry, and applies progression decisions.
- The **Learning Planner** owns the course DAG, each syllabus, and recommendations to study, review, or skip.
- **Learning Context** maintains the learner’s reported familiarity and evidence from assessments. The Planner uses that evidence to update recommendations.
- The **Tutor** teaches the assigned topics. The **Assessment Engine** evaluates the objectives of studied courses and returns feedback that can inform targeted review.

## What the course project should demonstrate

1. A broad goal produces a complete course DAG and an inspectable syllabus for every node before any familiarity questions.
2. The learner can inspect university sources and understand why a required course, dependency, or topic supports the route.
3. The system catches cycles and missing course references. A course plan covers the skills promised by its outgoing prerequisite links.
4. Two learners with the same goal can start from the same curriculum and receive different assignments after talking about specific coursework. An experienced learner can avoid repetition, and a learner with a gap can return to targeted material.

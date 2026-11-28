# Shared teaching library

The library supplies reusable, traceable educational material to Planner, Tutor, and Assessment. Workers can call its read interface directly for subject and objective material. The library service owns cache and admission writes, which are scoped to shared teaching storage. The Orchestrator owns private learner data.

## Lookup and admission

- Query with subject tags, objective or topic, required depth, and optional source IDs. The library returns approved references, relevant sections, permitted excerpts when available, and a result of found, insufficient coverage, or unavailable.
- Use cached approved material first. When coverage is insufficient, the service may retrieve from approved university source locations. A maintainer approves new source locations and entries before they serve as academic evidence.
- Retain the approved version and its locator. Mark broken links or changed content for review. Serve a cached excerpt if its recorded approval and permissions still apply.
- Start with official university catalogs, syllabi, open courseware, and faculty-published course material hosted or endorsed by an approved university. Use an accessible textbook section through those sources when it meets the same checks. Treat course descriptions as evidence for course scope and use topic-level sources for teaching claims or answer keys.
- Keep generated reusable exercises or examples linked to their approved sources and identified as generated. A maintainer can review and admit them as teaching practice. University requirements and explanations trace to their original materials.

## Records and provenance

Store source ID, title, institution, source URL, locator such as section or page, material type, topic tags, access date, approval status, rights or excerpt permission, and an optional permitted excerpt or version fingerprint. Store generated material with its type, source references, and review status. Learner messages, responses, identities, and assessment snapshots are private records.

Assessment preparation saves the permitted material needed for offline scoring or an independently checked original solution and its provenance. If appropriate material cannot be used in the saved attempt, return a grounding issue before showing the item. Retrieval inputs use educational topics and approved references.

## Initial source set

For the first two-course demo, use MIT’s course readings for interface specifications and modularity, and Carnegie Mellon’s architecture syllabus to justify outcomes around trade-offs. Select item-level support before generating an assessment. [MIT designing specifications](https://ocw.mit.edu/ans7870/6/6.005/s16/classes/07-designing-specs/) · [MIT abstraction and modularity](https://ocw.mit.edu/ans7870/6/6.005/s16/classes/12-abstract-data-types/) · [CMU architecture syllabus](https://mse.s3d.cmu.edu/0_documents/syllabi/sp2024/17633-sp2024-software-architecures-mseprof.pdf)

University examples in the Planner illustrate how sources inform different routes. Seed only materials that have a documented use and enough detail for the supported demo.

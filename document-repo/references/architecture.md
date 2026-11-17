# Architecture overview

The reader is new to the codebase and needs a map: what the parts are, how they fit together, and where to look for a given concern. The overview describes the system as it is. The reasons live in decision records; the details live in the code.

## Where it lives

`ARCHITECTURE.md` at the root or `docs/architecture.md`, following the repo. A small repo may need only an "Architecture" or "How it works" section in the README.

## Questions it answers

1. **What does the system do, and where are its edges?** One paragraph. Who and what calls it, and what it calls: users, other services, datastores, queues, third-party APIs, cloud services.
2. **What are the main parts?** Each component with its one responsibility and its location in the tree. Five to ten parts; group anything finer.
3. **How do they interact?** The one or two flows that matter most (a request, a job, a deploy), followed step by step through the components.
4. **Where does state live?** Datastores, caches, files, and remote state, with what owns each.
5. **How is it built and deployed?** A short account: environments, the pipeline, and where the config is. Link to the detail.
6. **What rules hold everywhere?** Invariants and boundaries a change must respect: "handlers never talk to the database directly", "all modules take a `tags` variable". Include how cross-cutting concerns are handled (auth, config, logging, errors) if there is one way.
7. **Where do I find X?** A short map from common tasks to directories, if the tree isn't self-explanatory.

For an infrastructure repo, the parts are accounts, environments, networks, and module layers, and the flows are how a change travels from commit to applied.

## Writing it

- Derive it from the code: the tree, entry points, imports and module calls, deploy config. Don't describe an intended design that the code doesn't implement. If part of it is planned, label it as planned.
- Stay at the level that changes slowly. Name components and directories, not individual functions or files that are likely to move.
- Use a diagram when there are more than three or four interacting parts. Prefer Mermaid in a code block when the host renders it (GitHub and GitLab do), so the diagram is versioned and editable. Keep it to boxes for components and labelled arrows for what flows between them, and say in the text what the diagram shows.
- Link each significant choice to its decision record instead of explaining it again.
- Link to the README for setup and to runbooks for operations.
- Aim for a document that can be read in ten minutes.

## Common findings

- Components or directories named in the document that no longer exist, and major ones that aren't mentioned. Compare the document with the top two levels of the tree.
- A diagram that disagrees with the text or the code.
- A file-by-file listing that repeats the tree and explains nothing.
- External dependencies missing: a datastore, queue, or third-party API the code clearly uses.
- Aspirational content written as though it were current.
- Long justifications that belong in decision records.
- Setup or operational steps duplicated from the README or runbooks.

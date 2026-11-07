# Shared guidance

## Authority

The `main` branches of ChefOST repositories are the shared source of truth. Read the relevant repository's `main` README and this guide before working, even when the checkout is on another branch. Explicit user instructions take precedence. A branch-local document, local copy or agent response does not establish shared policy.

- [Organization](profile/README.md): repository purposes and shared links.
- [PaperNotes](docs/PaperNotes.md): project goals and literature.
- [Evaluation](docs/Evaluation.md): shared metric definitions.
- [chefOST-data](https://github.com/chefOST/chefOST-data/blob/main/README.md): annotations, vocabularies and R2 video access.
- Each model repository's `main` README: its W&B destinations.

Link to these sources instead of copying their contents. Resolve conflicting claims with the user before changing shared definitions. Do not infer a new project decision from an implementation or experiment.

## Documentation

Keep documentation concise and timeless. A model README contains its purpose and links to shared guidance, data, evaluation and its W&B projects. It must not claim that a contributor branch is the implementation everyone should use.

Do not create or retain dated notes, status reports, plans, handoffs, run diaries, migration records, historical summaries, publication drafts, archival copies or detailed implementation/setup/run guides. Delete superseded documentation. Do not replace it with a record of the cleanup. Bibliographic years and paper versions identify scientific sources; they are not project status.

Keep repository `AGENTS.md` files short: direct agents to the canonical `main` README and this guide. Do not duplicate shared rules, metric definitions or data-access instructions. Add a documentation index only when it organizes substantive shared references; do not add navigation-only layers.

## Data and outputs

Use annotations and vocabularies from `chefOST-data/main` and videos from its linked R2 location. Keep credentials, downloaded data, features, checkpoints, environments and generated outputs out of ordinary Git history.

Runs, predictions, scores and artifacts belong in the model's W&B projects linked from its `main` README. Store code/data revisions, configuration and metric details with the run in W&B, not in repository run reports. Do not describe inference outputs as evaluation scores.

## Code changes

Use `<first-name>/<short-topic>` branches in the relevant repository. Keep changes focused and validate the behavior they affect. Use PRs for shared code changes; coordinate before rewriting a shared branch. Shared documentation changes belong on `main`.

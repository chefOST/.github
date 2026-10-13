# Branch and documentation style

Keep contributions focused and documentation as concise as possible.

## Branches

- New work: `<first-name>/<short-topic>`, lowercase with hyphens, e.g. `harrison/eval-alignment`. The name identifies the lead; others can contribute.
- Keep existing contributor-named branches, including migrated names such as `michelle/feat/moscato-vidosc-claude-eval`.
- Work in the relevant model repository. Use one coherent change per branch and PR; describe the change, validation and intended merge target.
- Branch from the code you are extending. While `vidosc` and `tubletegraph` have empty `main` branches, use the relevant populated contributor branch and target it in PRs until a shared baseline is established.
- Coordinate before rebasing or force-pushing a shared branch. Preserve migrated history; do not rename or rewrite branches just to satisfy this guide.

## Repository READMEs

Aim for roughly 150–250 words, excluding essential commands. Shorter is better when it still lets someone start working. Include only:

1. Purpose and current status; distinguish working features from plans. Identify the usable branch when `main` is empty.
2. The shortest verified setup/run example, with essential prerequisites and expected output. For data repositories, show how to obtain and use the data instead.
3. Required data, access or Git LFS instructions, or a direct link to them.
4. A documentation index and contribution/agent instructions.

Use plain headings, short paragraphs and bullets. Move architecture, full configuration, results, experiment history and troubleshooting into linked documents. Avoid badge walls, duplicated explanations and unverified commands. Keep essential setup instructions in each repository; link to this public guide for shared conventions.

## Short entry point, detailed docs

Use `README.md → docs/README.md → task-specific documents`. Prefer links over long collapsed sections: they keep both the page and the initial agent context small.

The README needs just one discovery line, once those files exist:

```markdown
Details: [documentation](docs/README.md). Agents: [instructions](AGENTS.md).
```

Make `docs/README.md` a small index: one descriptive link per topic, with when to read it. Create only documents the repository needs. Suggested topics:

- Setup: installation, environment and troubleshooting.
- Architecture: components and extension points.
- Data: schemas, versions and preparation; link shared definitions in `chefOST-data`.
- Evaluation: commands, splits, metrics, alignment rules and expected outputs.
- Experiments: dated results, exact code/data revisions and reproducible configurations.

Keep repository-local agent instructions in a short root `AGENTS.md`. Direct agents to the README, then the index and documents relevant to their task; include necessary local commands, checks and constraints. Do not copy the entire project history there. The org guide is not automatically inherited by other repositories or agents.

Use relative links within a repository so docs follow its branch. Link only existing files; label proposed work explicitly. Keep one authoritative explanation per topic, link to it elsewhere, and update affected docs/index entries in the same PR as the code.

# ChefOST

Tracking object states in cooking videos.

Research: [project goals and paper reviews](https://github.com/chefOST/.github/blob/main/docs/PaperNotes.md). Agents should start with the goals, then read the relevant papers.

## Where to work

- [vidosc](https://github.com/chefOST/vidosc): VidOSC adaptations and state descriptions.
- [tubletegraph](https://github.com/chefOST/tubletegraph): TubeletGraph adaptations and evaluation.
- [oscar](https://github.com/chefOST/oscar): OSCAR adaptations and object-state prediction.
- [chefOST-data](https://github.com/chefOST/chefOST-data): shared datasets; currently MOSCATO annotations/vocabularies via Git LFS. Shared evaluation tooling and Ego4D data are planned.
- [ego4d-pseudo-labels](https://github.com/chefOST/ego4d-pseudo-labels): existing Ego4D pseudo-labeling prototype.

VidOSC and TubeletGraph code currently lives on contributor branches, not `main`.

## Contributing

Use `<first-name>/<topic>` branches. Keep READMEs concise; link verbose docs (ie. for agents / slop) through an index. See the [branch and documentation style guide](https://github.com/chefOST/.github/blob/main/CONTRIBUTING.md).

## Legacy and reference

`VIDOSC-pipeline` is deprecated for new work; use `vidosc`. `chefOST-ml` retains older mixed experiments; continue model work in the repos above. Raf’s original data pipeline remains on [chefOST-data/raf/dataset-generation](https://github.com/chefOST/chefOST-data/tree/raf/dataset-generation).

[spoc](https://github.com/chefOST/spoc) is a research reference; [chefOST-site](https://github.com/chefOST/chefOST-site) is the demo placeholder.

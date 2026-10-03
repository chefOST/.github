# ChefOST

Tracking object states in cooking videos.

## Where to work

- [vidosc](https://github.com/chefOST/vidosc): VidOSC adaptations and state descriptions; [current evaluation branch](https://github.com/chefOST/vidosc/tree/michelle/feat/moscato-vidosc-claude-eval).
- [tubletegraph](https://github.com/chefOST/tubletegraph): TubeletGraph adaptations and evaluation; [current branch](https://github.com/chefOST/tubletegraph/tree/justin/feat/tubeletgraph-moscato-cmu-s12).
- [oscar](https://github.com/chefOST/oscar): OSCAR adaptations and object-state prediction.
- [chefOST-data](https://github.com/chefOST/chefOST-data): shared datasets; currently MOSCATO annotations/vocabularies via Git LFS. Shared evaluation tooling and Ego4D data are planned.
- [ego4d-pseudo-labels](https://github.com/chefOST/ego4d-pseudo-labels): existing Ego4D pseudo-labeling prototype.

Use `<first-name>/<topic>` branches in the relevant model repo and PRs for shared changes; separate model approaches belong in separate repos. VidOSC and TubeletGraph code currently lives on contributor branches, not `main`.

## Legacy and reference

`VIDOSC-pipeline` is deprecated for new work; use `vidosc`. `chefOST-ml` retains older mixed experiments; continue model work in the repos above. Raf’s original data pipeline remains on [chefOST-data/raf/dataset-generation](https://github.com/chefOST/chefOST-data/tree/raf/dataset-generation).

[spoc](https://github.com/chefOST/spoc) is a research reference; [chefOST-site](https://github.com/chefOST/chefOST-site) is the demo placeholder.

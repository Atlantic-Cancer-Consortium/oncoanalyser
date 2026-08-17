# ACC local changes to nf-core/oncoanalyser

This is a fork of [nf-core/oncoanalyser](https://github.com/nf-core/oncoanalyser). We fork and
modify rather than build a separate pipeline from scratch, so upstream changes can still be pulled
in over time.

**Keep this fork minimal.** Everything around running the pipeline — cluster config, submit
scripts, monitoring, troubleshooting, benchmarks, and the shared Claude project connection — lives
in
[`Atlantic-Cancer-Consortium/oncoanalyser-lsf-docs`](https://github.com/Atlantic-Cancer-Consortium/oncoanalyser-lsf-docs),
**not here**:

| Folder (in `oncoanalyser-lsf-docs`) | What's in it |
|---|---|
| `configs/` | `resource_v2.config` — the resource config |
| `execution/` | Submit script, samplesheet identity check, BWA_Alignment sub-pipeline |
| `monitoring/` | Monitoring SOP, health-check/orphan-cleanup scripts |
| `troubleshoot/` | Dated incident write-ups |
| `benchmarks/`, `envs/`, `tools/` | Placeholders for benchmark data, environment reference, and future standalone utilities |
| `scripts/` | Loose utility scripts (`netcheck.sh`, samplesheet identity check) |
| `claude/` | The shared Claude project connection |
| `pipeline/` | Upstream nf-core/oncoanalyser reference (full parameter list, output-file reference, process catalogue) — mirrors upstream docs, not our local changes |

Every file we add to *this* repo is a future merge conflict. If a change can be expressed as
config or a wrapper script, it belongs in `oncoanalyser-lsf-docs`, not here.

## Branch layout

| Branch | Purpose |
|---|---|
| `master` | Mirrors upstream 1:1. Never commit here — keep it fast-forward-only, either with GitHub's own "Sync fork" button or `git merge --ff-only upstream/master` |
| `acc/main` | Our integration branch. This is what CAIR runs |
| `acc/<topic>` | Feature branches off `acc/main` |

```bash
git remote add upstream https://github.com/nf-core/oncoanalyser.git
git fetch upstream --tags
git checkout master && git merge --ff-only upstream/master
```

**Current status (2026-08-13):** this fork has exactly one branch, `master`, 1,056 commits, and is
confirmed up to date with `nf-core/oncoanalyser:master` (0 forks, 0 stars, no tags/releases — this
is a bare, unmodified mirror). `acc/main` does not exist yet; it gets created the first time a real
local change or this changelog lands. None of the changes in the deviation table below have
actually been applied.

## Pulling a new upstream release

1. `git fetch upstream --tags`
2. `git checkout master && git merge --ff-only upstream/master` (or click **Sync fork** on GitHub)
3. `git checkout -b acc/merge-<version> acc/main && git merge master`
4. Resolve conflicts. **Every conflict should correspond to a row in the table below** — if one
   doesn't, an undocumented local change has crept in, and that is the real bug.
5. Re-run the smoke test: `-profile test,singularity` (43 processes, ~43 min on x86).
6. Update the version and the upstream commit SHA at the top of `pipeline/README.md` in
   `oncoanalyser-lsf-docs` (it tracks which upstream tag `pipeline/parameters.md` and
   `pipeline/outputs.md` were mirrored from), and add a changelog line there.

## Local deviations

Fill a row in for every change. A change not listed here will be silently reverted at the next
upstream merge by whoever resolves the conflict.

| # | File(s) | Change | Why | Upstreamable? | Added |
|---|---|---|---|---|---|
| 1 | Example `modules/local/sage/...` | *(candidate)* make `-run_tinc` optional | TINC loads ~100–180 k variants/chr on top of gnomAD/PON caches and drove the SAGE_SOMATIC OOM; it is currently hardcoded, so it can only be disabled with a local copy | Likely yes — worth a PR | not yet applied |


## What is deliberately NOT changed

- **The pipeline is not made architecture-agnostic.** We use it as a
  template and port only an MVP subset of confirmed-compatible steps. Notably, BWA-MEM2 requires
  x86-64 AVX2 and will not run on ppc64le, and there is no Singularity or conda on the PowerPC
  nodes — so alignment happens outside this pipeline on that route and CRAMs are fed in with the
  internal alignment step skipped (see `execution/BWA_Alignment/` in `oncoanalyser-lsf-docs`).
- **Resource requests.** Those are `configs/resource_v2.config` in `oncoanalyser-lsf-docs`.
  Changing `conf/base.config` here would be invisible to anyone reading our config, and would be
  silently overwritten on the next upstream merge.


## Related

- Docs + config + scripts repo: [`Atlantic-Cancer-Consortium/oncoanalyser-lsf-docs`](https://github.com/Atlantic-Cancer-Consortium/oncoanalyser-lsf-docs)
- Pipeline reference (full parameter list, output-file reference, process catalogue): that repo's `pipeline/`
- Shared Claude project connection: that repo's `claude/README.md`
- Upstream docs: <https://nf-co.re/oncoanalyser/2.3.0/>

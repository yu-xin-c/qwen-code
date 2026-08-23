# PR 9384 deep verification — find-simplifications skill (docs-only)

**Verdict: `findings`** — 47/48 scripted assertions passed, 1 failed.
Verified head: `055135b752947372377bc5f75366b5830e5d7fad` (merge tree `04c98aa`, PR base `d02e252`, doc's own cited base `5c56b67`, current main `98fa2e9`). Local maintainer round, 2026-08-23.

<details><summary>中文摘要</summary>

**结论:`findings`(可合并,带一条小修)**。本 PR 是纯文档(3 个 markdown 组成 `.qwen/skills/find-simplifications/` 技能),其承重论断是"文档里每条事实引用都与仓库一致"。我把三份文档中约 50 条可证伪声明逐条脚本化核验:47/48 通过。

**A/B 结论**:声明矩阵在 merge 树(04c98aa)与当前 main(98fa2e9)各跑一遍,均为 41/42;作者自证基线 5c56b67 上复核的数字(export\*=179、docs 284+35、workflows 52、truncateText ×5)全部精确。发布面/领地划分( core `./src/*` 导出、8 个 channel 包 `--access public`、webui/vendor/web-shell 只报不改、sdk 三件套、zed 四文件)、四个 worked example(useTomlMigration 0 字节唯一引用在 490 行、EnumSelector 恰 3 路径、allowlist 真过期恰 7 且含 eventBus/inMemoryChannel、dynamicCommandTranslation 恰 5 处命中 0 读取点、agent-view 2026-08-01 创建且 flag 无处解析)、CI 盲区断言(typecheck 不在 ci.yml、macos/windows/integration_cli 仅 merge_group、Prettier 走 `--write`)、squash-only + `COMMIT_OR_PR_TITLE`、ledger issue #9375 与试运行产物 #9379、GitHub 搜索分词行为——全部实测吻合。

**Findings(1 条,轻微)**:survey.md §3 称"全部 52 个 workflow 文件中 in-repo `uses: ./.github/workflows/…` 引用为 0(实测)"——在作者自己的引用基线 5c56b67 上就是 **5 处**(qwen-pr-safety-precheck×2、audio-capture-prebuilds、sync-live-host-to-oss、sync-desktop-to-oss)。复现显示无引号 grep 形态(`uses: \./\.github/workflows`)恰好在三个修订上都返回 0——作者大概率踩了这个假阴性。结论(.github/ 只报不改)不受影响、甚至更强,但按 PR 自己"错误论断比没有技能更糟"的标准,这处括号内证据应改为实测值或删除。

**未覆盖**:未以 agent 身份完整实跑 survey 协议本身;§2 空类表除 68-commands 行外未逐行重推导;平台行为不适用(纯文档)。

</details>

## Central claim and how it was tested

The PR's own framing: *"The load-bearing part of this PR is factual accuracy … a wrong claim in it is worse than no skill."* There is no runtime A/B for a docs-only change, so the A/B here is **claim × tree**: every falsifiable citation in the three documents (paths, line anchors, counts, CI semantics, publish-surface facts, sibling-skill claims, GitHub-convention claims) was extracted and checked by a scripted harness (`claims-check.sh`, kept beside this report) against three trees:

| Arm | Tree | Result |
| --- | --- | --- |
| What would merge | merge ref `04c98aa` (head into main-side `b455bad`) | **41/42 PASS** |
| Current main (drift) | `98fa2e9` | **41/42 PASS** (same cell fails) |
| Doc's cited base | `5c56b67` (spot re-measurements via `git grep`) | export\*=**179**, docs=**284+35**, workflows=**52**, truncateText=**5**, self-uses see Finding 1 |

Captures: `01-claims-matrix-merge-tree.png`, `02-claims-matrix-main-tip-drift.png` (full 42-row matrix each), `03-workflow-selfuses-false-negative.png` (Finding 1), `04-github-surface-checks.png` (repo settings / ledger / search demo).

### What passed (grouped; one row per claim family)

| Family | Claims verified | Headline measurements |
| --- | --- | --- |
| Published surface / Territory | core `exports` has `./src/*`+`./dist/*`; `files` has `vendor`; `export * from` ≈179 (**181** merge / **182** main / **179** docbase); release.yml publishes audio-capture + channel-base + 7-loop channels = **8** `channel-*` with `--access public`; webui `publishConfig.access=public`, not private; `copy_bundle_assets.js` copies core vendor + web-shell SPA; `getBuiltinRipgrep()` joins path from segments (no literal); vendored `rg` dirs exist; sdk-ts `--access public`, sdk-py PyPI, sdk-java `mvn deploy`; zed-extension exactly **4** files; workspaces negate desktop/desktop-shell | all exact at cited base; ≤2 unit drift at head |
| Worked examples | `useTomlMigration.ts` **0 bytes**, sole ref `eslint.legacy-filenames.mjs:`**490** (exact); EnumSelector exactly **3** paths; allowlist **559** entries, naive-stale 34≈37, true stale under rule semantics **7** incl. `eventBus`+`inMemoryChannel`; `dynamicCommandTranslation` **5 hit lines / 4 files / 0 read sites**, docs row at `settings.md:`**98** (exact), schema at :634 (cited 632 — header-covered drift); agent-view production **5,418** lines (~6,000), created **2026-08-01**, `--internal-agent-view-supervisor` parsed nowhere outside, zero importers of top-level dir | all reproduce |
| Mechanism claims | 68 exported `*Command` consts → **68/68** in `BuiltinCommandLoader.ts`; `escapeRegExp` **4** production defs (5th is a test file); `truncateText` **5** defs; `config.ts` `.strict()`; 9 locales; `.gitignore` `.qwen/*` + re-includes; kebab rule expands `**/{name}.ts` AND `**/{name}.*.ts`, scope only core/src+cli/src; schema stale-gate in ci.yml:410-419 | all exact |
| CI blind spots (land.md §4) | `typecheck` appears **0**× in ci.yml; macos/windows `Test` + `integration_cli` gated `merge_group`-only; `Run Prettier` → `lint.js --prettier` → `prettier --write .`; profile classification incl. `docs_only`; every named npm script exists | all verified |
| Sibling-skill / AGENTS quotes | 4 AGENTS.md quotes verbatim; repo-hygiene ban phrase; bundled `/simplify` lists comment removal; `/create-issue` is issue-only (cannot comment) | all verified |
| GitHub surface | repo is **squash-only** with `COMMIT_OR_PR_TITLE`; ledger #9375 exists with the exact prescribed title; trial-run artifact #9379 MERGED; unquoted `enum-selector in:body` returns **4 unrelated PRs**, quoted marker returns **0** (tokenization claim holds) | all verified |
| Gates | repo-pinned prettier 3.6.1 `--check` on the 3 files: **clean**; trial merge head→current main: **0 conflicts** | pass |

## Findings

### F1 (minor, in the PR's own load-bearing dimension) — survey.md §3 step 3's "0 self-references" measurement is a false negative of the grep form

- **Claim**: *"(measured: 0 in-repo `uses: ./.github/workflows/…` references across all 52 workflow files)"*.
- **Measured**: **5** at the doc's own cited base `5c56b67`, at PR base `d02e252`, at the merge tree, and at current main — `qwen-triage.yml:40` and `qwen-code-pr-review.yml:93` → `qwen-pr-safety-precheck.yml`, `release.yml` → `audio-capture-prebuilds.yml`, `live-host-release.yml:357`, `desktop-release.yml:703`. These predate the PR.
- **Root cause is measured, not guessed**: the quote-less pattern `uses: \./\.github/workflows` returns **0** at every one of those revisions because real YAML is `uses: './.github/workflows/…'` (quote between `uses: ` and `./`). This harness's first run reproduced the author's exact zero with the same quote-less form before the quoted form found the 5 — see `03-workflow-selfuses-false-negative.png`. Irony noted: survey.md §0 step 4 mandates calibrating the search; this is the miscalibration class it warns about.
- **Blast radius**: none on decisions — `.github/` is report-only either way, and in-repo reusable-workflow references only *add* visible consumers. But by the PR's own standard ("a wrong claim in it is worse than no skill") the parenthetical should be corrected: either quote the real count or drop the parenthetical and keep the qualitative point (triggers/required-checks are still invisible to any grep).
- **Repro**: `git grep -n "uses: '\./\.github/workflows" 5c56b67182 -- '.github/workflows'` → 5 rows; same with quote-less pattern → 0.
- **Suggested one-line fix** (not applied — docs edit is the author's call): replace the parenthetical with *(measured: 5 in-repo `uses: './.github/workflows/…'` references — all reusable-workflow includes; triggers and required checks remain invisible to grep)*.

### Drift observations (not findings; the docs anticipate all of these)

- `export * from` 179 (docbase, exact) → 181 merge → 182 main. Claimed "~179" — still fine.
- docs/design 284→309→312, docs/plans 35→37. The doc explicitly says recompute, not trust.
- Line anchors moved ≤27 lines (settingsSchema 632→634, eslint.config.js 277-282→321-324, release-sdk.yml 297→300); survey.md's header states line numbers are leads, re-locate by symbol — re-location succeeded everywhere.
- Workflows stayed at exactly **52** `.yml`; allowlist stayed at exactly **559**; locales **9**.

## Not covered

- The survey protocol itself was not executed end-to-end as an agent run in this round (claim-level verification only). The author's recorded trial run exists as ledger #9375 and merged candidate #9379, both confirmed to exist and match the described shape.
- survey.md §2 "classes with nothing in them": only the 68-commands row was re-derived at head; the other rows were accepted as measured-at-`8fd0162c68` per the doc's own note.
- The `.mjs` marker-search behavior was verified against the search API as used by `gh pr list`; no writes of any kind were performed.
- No platform-specific behavior applies (docs-only diff); prettier gate ran from the maintainer checkout's pinned 3.6.1.

## Methodology

Local maintainer round on macOS. PR metadata resolved via `gh pr view 9384 --repo QwenLM/qwen-code`; base/head/merge/main OIDs pinned as `refs/verify/pr9384-*` and checked out as scratch worktrees under `tmp/pr9384-verify-20260823-083849/` (removed after captures). All 42 in-harness assertions ran identically on the merge tree and on current main (drift arm; drift-arm cells are measurements and are excluded from the verdict counts). Out-of-harness gates: prettier `--check`, `git merge-tree` trial merge, `gh api` repo settings, `gh issue view`, two `gh pr list` searches. Raw logs: `logs/merge-tree.log`, `logs/main-tip.log`; harness and demo scripts kept beside this report. Counts in `assertions.json` cover only scripted checks that executed (42 harness @ merge tree + 6 gates; 47 pass, 1 fail).

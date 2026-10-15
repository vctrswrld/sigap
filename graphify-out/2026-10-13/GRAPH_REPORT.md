# Graph Report - sigap  (2026-10-13)

## Corpus Check
- cluster-only mode — file stats not available

## Summary
- 184 nodes · 198 edges · 61 communities (8 shown, 53 thin omitted)
- Extraction: 97% EXTRACTED · 3% INFERRED · 0% AMBIGUOUS · INFERRED: 5 edges (avg confidence: 0.55)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `a2c089d2`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- package.json
- common.sh
- template.sh
- compilerOptions
- create-new-feature.sh
- devDependencies
- scripts
- hitl-loop.template.sh
- block-dangerous-git.sh
- postcss.config.mjs

## God Nodes (most connected - your core abstractions)
1. `compilerOptions` - 16 edges
2. `run_wizard()` - 12 edges
3. `create-new-feature.sh script` - 11 edges
4. `get_feature_paths()` - 8 edges
5. `get_repo_root()` - 8 edges
6. `setup-tasks.sh script` - 8 edges
7. `check-prerequisites.sh script` - 7 edges
8. `resolve_template_content()` - 7 edges
9. `scripts` - 6 edges
10. `has_jq()` - 5 edges

## Surprising Connections (you probably didn't know these)
- `create-new-feature.sh script` --calls--> `get_repo_root()`  [EXTRACTED]
  .specify/scripts/bash/create-new-feature.sh → .specify/scripts/bash/common.sh
- `create-new-feature.sh script` --calls--> `_persist_feature_json()`  [EXTRACTED]
  .specify/scripts/bash/create-new-feature.sh → .specify/scripts/bash/common.sh
- `create-new-feature.sh script` --calls--> `resolve_template_content()`  [EXTRACTED]
  .specify/scripts/bash/create-new-feature.sh → .specify/scripts/bash/common.sh
- `check-prerequisites.sh script` --calls--> `check_dir()`  [EXTRACTED]
  .specify/scripts/bash/check-prerequisites.sh → .specify/scripts/bash/common.sh
- `check-prerequisites.sh script` --calls--> `check_file()`  [EXTRACTED]
  .specify/scripts/bash/check-prerequisites.sh → .specify/scripts/bash/common.sh

## Import Cycles
- None detected.

## Communities (61 total, 53 thin omitted)

### Community 0 - "package.json"
Cohesion: 0.07
Nodes (24): eslintConfig, nextConfig, dependencies, next, @prisma/client, react, react-dom, name (+16 more)

### Community 1 - "common.sh"
Cohesion: 0.16
Nodes (21): check-prerequisites.sh script, check_dir(), check_file(), find_specify_root(), format_speckit_command(), get_current_branch(), get_feature_paths(), get_invoke_separator() (+13 more)

### Community 2 - "template.sh"
Cohesion: 0.22
Nodes (18): ask(), ask_secret(), banner(), _clear(), _existing(), finish(), note(), open_url() (+10 more)

### Community 3 - "compilerOptions"
Cohesion: 0.11
Nodes (18): compilerOptions, allowJs, esModuleInterop, incremental, isolatedModules, jsx, lib, module (+10 more)

### Community 4 - "create-new-feature.sh"
Cohesion: 0.36
Nodes (9): clean_branch_name(), contains_non_ascii(), fit_branch_name(), generate_branch_name(), get_highest_from_specs(), is_feature_number_in_range(), create-new-feature.sh script, spec_prefix_exists() (+1 more)

### Community 5 - "devDependencies"
Cohesion: 0.18
Nodes (11): devDependencies, babel-plugin-react-compiler, eslint, eslint-config-next, prisma, tailwindcss, @tailwindcss/postcss, @types/node (+3 more)

### Community 6 - "scripts"
Cohesion: 0.33
Nodes (6): scripts, build, dev, lint, postinstall, start

### Community 7 - "hitl-loop.template.sh"
Cohesion: 0.83
Nodes (3): capture(), hitl-loop.template.sh script, step()

## Knowledge Gaps
- **10 isolated node(s):** `babel-plugin-react-compiler`, `@prisma/client`, `react`, `react-dom`, `tailwindcss` (+5 more)
  These have ≤1 connection - possible missing edges. (Counts symbols only; 113 node(s) total have ≤1 connection when file, concept and rationale nodes are included.)
- **53 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `devDependencies` connect `devDependencies` to `package.json`?**
  _High betweenness centrality (0.024) - this node is a cross-community bridge._
- **What connects `babel-plugin-react-compiler`, `@prisma/client`, `react` to the rest of the system?**
  _10 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `package.json` be split into smaller, more focused modules?**
  _Cohesion score 0.07142857142857142 - nodes in this community are weakly interconnected._
- **Why does `scripts` connect `scripts` to `package.json`?**
  _High betweenness centrality (0.013) - this node is a cross-community bridge._
- **Should `compilerOptions` be split into smaller, more focused modules?**
  _Cohesion score 0.10526315789473684 - nodes in this community are weakly interconnected._
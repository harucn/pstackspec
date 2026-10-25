# pstackspec

[![License: MIT](https://img.shields.io/github/license/harucn/pstackspec)](LICENSE)

pstackspec is an OpenSpec custom schema that wraps the built-in `spec-driven` workflow with [pstack](https://github.com/cursor/plugins/tree/main/pstack) skills. It sharpens your change request before writing specs and verifies every scenario after implementation.

Proposal, specs, design, and tasks follow the `spec-driven` format as-is, so they stay in sync when you update OpenSpec.

```text
sharpen ──→ proposal ──→ specs ──→ tasks ──→ apply ──→ verify ──→ archive
                │                     ↑
                └──→ design ──────────┘
```

## Requirements

- OpenSpec CLI 1.14+, with your project initialized via `openspec init --tools claude`
- The `new`, `continue`, and `verify` workflows enabled (add them with `openspec config profile`, then run `openspec update`)
- Claude Code with the pstack plugin

  ```text
  /plugin marketplace add cursor/plugins
  /plugin install pstack@cursor-plugins
  ```

## Installation

Run in your project root:

```bash
tmp=$(mktemp -d)
git clone --depth 1 https://github.com/harucn/pstackspec.git "$tmp"
rm -rf openspec/schemas/pstackspec && mkdir -p openspec/schemas
cp -R "$tmp/openspec/schemas/pstackspec" openspec/schemas/
rm -rf "$tmp"
sed -i.bak 's/^schema: .*/schema: pstackspec/' openspec/config.yaml && rm openspec/config.yaml.bak
```

This copies the schema into `openspec/schemas/pstackspec/` and makes it the default in `openspec/config.yaml`. Commit both so your teammates get it too. To update, run the same commands again.

## Usage

To go one step at a time:

```text
/opsx:new my-feature
/opsx:continue   # one artifact per run: sharpen → proposal → specs → design → tasks
/opsx:apply
/opsx:continue   # creates verify.md
/opsx:archive
```

`/opsx:ff` and `/opsx:propose` create everything from sharpen to tasks in one go, pausing only for your approval at sharpen. They stop before verify, so run `/opsx:continue` after apply to create it.

## Workflow

| Stage | pstack skills | Output | Gate |
| --- | --- | --- | --- |
| sharpen | `sharpen`, `how` | `sharpen.md`: the sharpened request, current code (file:line), and how to tell it's done | Waits for you to approve the rewritten request |
| proposal | — (plain `spec-driven`) | `proposal.md` | |
| specs | — (plain `spec-driven`) | `specs/**/spec.md`, with scenarios granular enough to become test names | |
| design (optional) | `architect` Phases A–C, `principle-model-the-domain` | `design.md`: caller-side usage, type sketches, rejected alternatives | |
| tasks | `principle-sequence-verifiable-units`, then `interrogate` | `tasks.md`: each task is one build → verify → commit cycle | |
| apply | `poteto-mode` feature / refactoring playbooks, `show-me-your-work` | Code, commits, and `decisions.tsv` | Opens a PR only when asked |
| verify | `blast-radius`, plus `interrogate` if design questions remain open | `verify.md`: code and tests for each scenario, and an overall PASS / FAIL | Doesn't archive on FAIL |

## License

MIT ([LICENSE](LICENSE)). The proposal, spec, design, and tasks templates in `openspec/schemas/pstackspec/templates/` are copied from OpenSpec's `spec-driven` schema (MIT, see `openspec/schemas/pstackspec/LICENSE.openspec`).

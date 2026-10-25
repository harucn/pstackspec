# pstackspec

OpenSpec の custom schema。組み込みの `spec-driven` に [pstack](https://github.com/cursor/plugins/tree/main/pstack) の skill を差し込む。[superspec](https://github.com/danielhanold/superspec) が Superpowers で行っていることを pstack で行う。

proposal・specs・design・tasks の書式は実行時に `openspec instructions <id> --schema spec-driven` から取るので、`spec-driven` の instruction を書き写していない。OpenSpec を更新すれば書式の変更にも追従する。

```text
sharpen ──→ proposal ──→ specs ──→ tasks ──→ apply ──→ verify ──→ archive
                │                     ↑
                └──→ design ──────────┘
```

| 段 | pstack の skill |
| --- | --- |
| sharpen | `sharpen`、`how` |
| design（条件付き） | `architect` の Phase A〜C、`principle-model-the-domain` |
| tasks | `principle-sequence-verifiable-units`、最後に `interrogate` |
| apply | `poteto-mode` の feature / refactoring playbook、`show-me-your-work` |
| verify | `blast-radius`、争点が残る場合だけ `interrogate` |

## 前提

- OpenSpec CLI 1.14 以降と、対象リポジトリで `openspec init --tools claude` 済みであること
- Claude Code の pstack プラグイン（`/plugin install pstack@cursor-plugins`）

## インストール

自分だけで使う（リポジトリに入れない）:

```bash
mkdir -p ~/.local/share/openspec/schemas
ln -s "$PWD/openspec/schemas/pstackspec" ~/.local/share/openspec/schemas/pstackspec
```

リポジトリに入れてチームで使う:

```bash
cp -r openspec/schemas/pstackspec <repo>/openspec/schemas/
```

確認:

```bash
openspec schema which pstackspec
```

## 使い方

既定の schema を変えずに、change ごとに選ぶ:

```
/opsx:new my-feature --schema pstackspec
/opsx:continue   # sharpen、proposal、specs、design、tasks の順に 1 つずつ
/opsx:apply
/opsx:continue   # verify
/opsx:archive
```

既定にする場合は `openspec/config.yaml` を `schema: pstackspec` にする。

## 注意

- user レベルで入れた場合、`pstackspec` で作った change は schema を持っていない人の環境では `openspec status` が解決できない。チームのリポジトリで使い続けるならリポジトリに入れる。
- pstack の skill は Skill tool では呼べないため、instruction は `~/.claude/plugins/installed_plugins.json` から pstack の場所を解決して `SKILL.md` を Read させている。

## License

`templates/` の proposal・spec・design・tasks は OpenSpec の `spec-driven` から複製した（MIT、`openspec/schemas/pstackspec/LICENSE.openspec`）。

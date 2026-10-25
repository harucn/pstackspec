# pstackspec

OpenSpec の custom schema。組み込みの `spec-driven` に [pstack](https://github.com/cursor/plugins/tree/main/pstack) の skill を差し込む。[superspec](https://github.com/danielhanold/superspec) が Superpowers で行っていることを pstack で行う。

proposal・specs・design・tasks の書式は実行時に `openspec instructions <id> --schema spec-driven` から取るので、`spec-driven` の instruction を書き写していない。OpenSpec を更新すれば書式の変更にも追従する。

```text
sharpen ──→ proposal ──→ specs ──→ tasks ──→ apply ──→ verify ──→ archive
                │                     ↑
                └──→ design ──────────┘
```

## 各段で起きること

| 段 | pstack の skill | 成果物 | 人の確認 |
| --- | --- | --- | --- |
| sharpen | `sharpen`、`how` | `sharpen.md`: 深めた要望、現状のコード（file:line）、完了の判定方法 | 書き換えた要望に合意するまで止まる |
| proposal | なし（spec-driven の書式） | `proposal.md` | |
| specs | なし（spec-driven の書式） | `specs/**/spec.md`。Scenario はテスト名になる粒度 | |
| design（条件付き） | `architect` の Phase A〜C、`principle-model-the-domain` | `design.md`: 使い方、型の sketch、却下した案 | |
| tasks | `principle-sequence-verifiable-units`、最後に `interrogate` | `tasks.md`: 1 task = 作る・検証・commit | |
| apply | `poteto-mode` の feature / refactoring playbook、`show-me-your-work` | コードと commit、`decisions.tsv` | PR は頼まれたときだけ作る |
| verify | `blast-radius`、争点が残る場合だけ `interrogate` | `verify.md`: Scenario ごとの対応コード、PASS / FAIL | FAIL なら archive しない |

apply では subagent（`pstack:poteto-agent`）がコードを書き、本体が差分を読んで検証コマンドを通してから checkbox を付ける。

## superspec との違い

| | superspec | pstackspec |
| --- | --- | --- |
| 最初の段 | brainstorm（`superpowers:brainstorming`） | sharpen（`sharpen` + `how`） |
| 実装計画 | `plan.md`（`superpowers:writing-plans`） | なし。tasks.md を検証可能な単位まで細かくする |
| apply | git worktree + subagent-driven-development | 同じ作業ツリーで subagent が実装 |
| 最後の段 | finalize（worktree の merge、push、PR コメント） | verify まで。push と PR は手動 |
| spec-driven の instruction | schema に書き写している | 実行時に読む |

## 前提

- OpenSpec CLI 1.14 以降。対象リポジトリで `openspec init --tools claude` 済みであること
- OpenSpec の workflow で `new`・`continue`・`verify` が有効なこと。既定の `core` profile には入っていないので、`openspec config profile` で追加して `openspec update` を実行する。`openspec config get workflows` で確認できる
- Claude Code の pstack プラグイン（`/plugin install pstack@cursor-plugins`）。`/pstack:setup-pstack` で subagent のモデルを設定しておく。未設定なら pstack の既定モデルを使う

## インストール

```bash
git clone https://github.com/harucn/pstackspec.git ~/Development/pstackspec
cd ~/Development/pstackspec
```

自分だけで使う（対象リポジトリに入れない）:

```bash
mkdir -p ~/.local/share/openspec/schemas
ln -s "$PWD/openspec/schemas/pstackspec" ~/.local/share/openspec/schemas/pstackspec
```

対象リポジトリに入れてチームで使う:

```bash
cp -r openspec/schemas/pstackspec <repo>/openspec/schemas/
```

対象リポジトリで確認:

```bash
openspec schema which pstackspec   # Source: user または project
```

## 使い方

既定の schema を変えずに、change ごとに選ぶ。1 段ずつ確認しながら進める場合:

```
/opsx:new my-feature --schema pstackspec
/opsx:continue   # sharpen → proposal → specs → design → tasks を 1 回に 1 つ
/opsx:apply
/opsx:continue   # verify
/opsx:archive
```

`/opsx:ff` や `/opsx:propose` を使うと sharpen から tasks までを 1 回で作る（sharpen の合意では止まる）。verify は含まれないので、apply の後に `/opsx:continue` で作る。

既定にする場合は対象リポジトリの `openspec/config.yaml` を `schema: pstackspec` にする。

## 注意

- user レベルで入れた場合、`pstackspec` で作った change は schema を持っていない人の環境で `Unknown schema 'pstackspec'` になる。チームのリポジトリで使い続けるならリポジトリに入れる。
- verify が FAIL のときに archive を止めるのは instruction の指示だけで、CLI は止めない。`/opsx:archive` は `verify.md` が無ければ警告して確認を求めるが、FAIL と書かれた `verify.md` があれば警告しない。
- apply の判断ログ `decisions.tsv` は change ディレクトリに置かれ、archive でも一緒に移る。commit しないなら `.gitignore` に `openspec/changes/**/decisions.tsv` を足す。
- pstack の skill は Skill tool では呼べないため、instruction は `~/.claude/plugins/installed_plugins.json` から pstack の場所を解決して `SKILL.md` を Read させている。Claude Code 以外の harness では動かない。

## 構成

```text
openspec/schemas/pstackspec/
├── schema.yaml        # artifact の依存関係と各段の instruction
├── LICENSE.openspec
└── templates/         # sharpen, proposal, spec, design, tasks, verify
```

## License

`templates/` の proposal・spec・design・tasks は OpenSpec の `spec-driven` から複製した（MIT、`openspec/schemas/pstackspec/LICENSE.openspec`）。

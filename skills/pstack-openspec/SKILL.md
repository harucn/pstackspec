---
name: pstack-openspec
description: "OpenSpec の change を pstack の進め方で回す。引数は stage（propose | apply | verify | archive）と change 名または要望。Use for /pstack-openspec."
disable-model-invocation: true
allowed-tools: Bash(openspec:*)
argument-hint: "<propose|apply|verify|archive> <change-name or request>"
---

# pstack-openspec

OpenSpec が手順と記録を持ち、pstack が各段の質を持つ。OpenSpec の手順は書き写さず、生成済みの skill を読んで従う。

## 準備（毎回）

1. pstack の場所を解決する: `~/.claude/plugins/installed_plugins.json` の `pstack@cursor-plugins` の `installPath`。以下 `<pstack>` と書く。バージョン番号を直書きしない。
2. `<pstack>/docs/harness/claude-code.md` を読む。pstack skill は Skill tool では呼べないので、`<pstack>/skills/<name>/SKILL.md` を Read して従う。
3. `~/.claude/rules/pstack-models.md` が無ければ pstack の既定モデルを使う。
4. `openspec list --json` で `root` を確認する。`null` なら書き込みをせず、`openspec init` するか、store を使うか、OpenSpec なしで進めるかをユーザーに聞いて止まる。
5. stage の OpenSpec skill `.claude/skills/<skill>/SKILL.md`（プロジェクト内）を読む。無ければ `openspec update` を提案して止まる。

| stage | OpenSpec skill | pstack の差し込み |
| --- | --- | --- |
| propose | `openspec-propose` | `sharpen` → `how` → `architect` → `interrogate` |
| apply | `openspec-apply-change` | `poteto-mode` の feature / refactoring playbook、`principle-sequence-verifiable-units`、`show-me-your-work` |
| verify | `openspec-verify-change` | `blast-radius`、`interrogate` |
| archive | `openspec-archive-change` | なし |

OpenSpec skill の guardrail（root 確認、planning boundary、checkbox の更新規則、pause 条件）は pstack 側の指示より優先する。

## propose

1. `sharpen` で要望を深める。書き換えた要望をユーザーに見せてから進む。
2. `how` で影響するサブシステムを読む。
3. `openspec-propose` に従って artifact を作る。各 artifact の中身は `openspec instructions <artifact> --change <name> --json` が正。
4. design.md は `architect` で型・シグネチャ・モジュール境界の案を出し、選んだ案と却下した案の理由を書く。
5. proposal / specs / design / tasks に `interrogate` をかけ、指摘を反映する。tasks は 1 task = 1 つの検証可能な単位にする（`principle-sequence-verifiable-units`）。
6. `openspec validate <name>` を通して止まる。同じ実行で apply に進まない。

## apply

1. `openspec-apply-change` の手順 1〜5 に従う。
2. 手順 6 の各 task を `poteto-mode` の feature playbook（構造だけの変更なら refactoring playbook）の手順 4〜5 で実装する。コードは subagent に書かせ、lead は検証と checkbox 更新を持つ。
3. checkbox は検証が通ってから lead が `sourcePath` と `line` で更新し、`openspec instructions apply --change <name> --json` で進捗を確かめる。task ごとに commit する。
4. 判断ごとに `show-me-your-work` の TSV に 1 行残す。
5. PR 作成は playbook の Opening a PR ではなく、ユーザーに頼まれたときだけ行う。

## verify

1. `openspec-verify-change` に従う。
2. 差分に `blast-radius` をかけ、安全と言える根拠を実際に動かして示す。
3. 設計に争点が残る場合だけ `interrogate` をかける。自動では修正しない。

## archive

`openspec-archive-change` に従う。verify で未解決の CRITICAL が残っていれば archive せず止まる。

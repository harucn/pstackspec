# pstack-openspec

OpenSpec の change（propose → apply → verify → archive）を pstack の skill で回す Claude Code 用 skill。

## 前提

- [OpenSpec](https://github.com/Fission-AI/OpenSpec) CLI と、対象リポジトリでの `openspec init --tools claude`
- Claude Code の pstack プラグイン（`/plugin install pstack@cursor-plugins`）

## インストール

```bash
ln -s "$PWD/skills/pstack-openspec" ~/.claude/skills/pstack-openspec
```

## 使い方

```
/pstack-openspec propose ユーザーがメールアドレスを変更できるようにする
/pstack-openspec apply <change-name>
/pstack-openspec verify <change-name>
/pstack-openspec archive <change-name>
```

propose は artifact を作って止まる。apply は別の実行で始める。

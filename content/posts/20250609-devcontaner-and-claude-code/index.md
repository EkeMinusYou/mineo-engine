---
title: "ターミナルで生活する人のための Dev Container CLI + Claude Code"
date: 2025-06-09T00:07:12+09:00
---

Claude Code の勢いがすさまじいですね。

ベストプラクティスっぽいものを考えたのでメモです。以下の要素がある人向けです。

- Claude Code を使いたい
- セキュリティ確保のため、devcontainer を使いたい
- でも、VSCode / JetBrain製品 は起動したくない
- 共通の devcontainer.json を利用したい

[Dev Container CLI](https://github.com/devcontainers/cli) を install しましょう。特に書かれていないですが、brew でも install できます。

```bash
brew install devcontainer
```

共通の `devcontainer.json` を用意します。`~/.config/devcontainer/claude/devcontainer.json` とかに配置するとよいでしょう。

ちなみに、私のはこんな感じです。
`.claude` を共有することで、devcontainerであることを気にすることなく、利用することができます。

```json
{
  "name": "Claude Code",
  "image": "mcr.microsoft.com/vscode/devcontainers/base:ubuntu-24.04",
  "features": {
    "ghcr.io/anthropics/devcontainer-features/claude-code:latest": {},
    "ghcr.io/devcontainers/features/node:1": {
      "version": "lts"
    },
    "ghcr.io/devcontainers/features/go:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "mounts": [
    "type=bind,source=${localEnv:HOME}/.gitconfig,target=/home/vscode/.gitconfig,readonly",
    "type=bind,source=${localEnv:HOME}/.claude,target=/home/vscode/.claude",
    "type=bind,source=${localEnv:HOME}/.claude/settings.json,target=/home/vscode/.claude/settings.json,readonly"
  ]
}
```

開発したいリポジトリに移動します。

まずは、コンテナを立ち上げます。(もちろん、Docker は使える状態である必要があります。)

```bash
devcontainer up --workspace-folder . --config ~/.config/devcontainer/claude/devcontainer.json
```

以下で、Claude Code を実行します。

```bash
devcontainer exec --workspace-folder . --config ~/.config/devcontainer/claude/devcontainer.json claude
```

注意点として、コンテナが残り続けてしまいますが、Dev Container CLI では削除できないので、docker コマンドを使う必要があります。

Claude Code だけを使うなら、VSCode の中で対話するよりミニマムでよいかもしれません。並列作業もしやすそうです。

私は、`.zshrc` に alias を登録しました。これで、`devclaude` だけで立ち上がるので便利です。

```
alias devclaude="devcontainer up --workspace-folder . --config ~/.config/devcontainer/claude/devcontainer.json && devcontainer exec --workspace-folder . --config ~/.config/devcontainer/claude/devcontainer.json claude"
```

Enjoy!

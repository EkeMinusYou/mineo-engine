---
title: "Git/GitHubの便利コマンドを紹介！"
date: 2024-05-11T16:59:47+09:00
tags: ["cli", "GitHub", "fzf"]
---


## はじめに

よく使っている便利コマンドをいくつか紹介します。

自分はzshのaliasに登録して、すぐ呼び出せるようにしています。

github cli とfzfの導入が必要です。

### デフォルトブランチに移動する 

リポジトリによって、デフォルトブランチがmainとmasterがバラバラだったりするので。

```shell
git switch $(gh repo view --json defaultBranchRef --jq .defaultBranchRef.name)
```

## fzfでブランチリストを出してswich

```shell
git branch | fzf --reverse --height 50% | xargs git switch
```

## デフォルトブランチからrebaseする

```shell
git pull --rebase origin $(gh repo view --json defaultBranchRef --jq .defaultBranchRef.name)
```

## fzfでstarしたリポジトリ一覧を出して、ブラウザで開く

気になるリポジトリはstarをつけますが、後になって探すのが、大変だったりするので、作りました。

```shell
gh api -X GET /user/starred --paginate --cache 24h | jq '.[].full_name' -r | fzf --reverse --height 50% | xargs gh repo view --web
```

## おわりに

以上、子ネタでした。

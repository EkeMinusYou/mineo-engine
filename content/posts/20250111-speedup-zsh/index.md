---
title: "zshの起動速度を20倍高速化した"
date: 2025-01-11T02:42:53+09:00
---


## はじめに
zshの起動速度が遅かったので高速化しました。

## 前提

以下を使っています。

- Mac
- sheldon(プラグインマネージャー)
- wezterm(ターミナルエミュレーター)

## 計測方法


以下で計測しました

```shell
(for i in $(seq 1 10); do time zsh -i -c exit; done)
```

profilingもしましたが、あまり参考にならなかったので、そんなに使いませんでした。

## なにもしていない状態

1.2s程度でした。やばいですね。

```shell
zsh -i -c exit  0.75s user 0.35s system 76% cpu 1.428 total
zsh -i -c exit  0.61s user 0.26s system 70% cpu 1.232 total
zsh -i -c exit  0.58s user 0.26s system 69% cpu 1.202 total
zsh -i -c exit  0.63s user 0.25s system 72% cpu 1.218 total
zsh -i -c exit  0.61s user 0.25s system 68% cpu 1.270 total
zsh -i -c exit  0.60s user 0.25s system 70% cpu 1.208 total
zsh -i -c exit  0.56s user 0.24s system 67% cpu 1.181 total
zsh -i -c exit  0.60s user 0.25s system 69% cpu 1.243 total
zsh -i -c exit  0.65s user 0.26s system 73% cpu 1.238 total
zsh -i -c exit  0.59s user 0.26s system 70% cpu 1.202 total
```

## やったこと

ひとすら`.zshrc`を書き換えていきます。

### `brew --prefix` を`$HOMEBREW_PREFIX`に置き変える
Macなのでhomebrewを使っていましたが、以下のようなbrew コマンドに時間がかかっていたようなので、環境変数を参照するようにしました。

```diff
- export EDITOR=$(brew --prefix)/bin/nvim
+ export EDITOR=$HOMEBREW_PREFIX/bin/nvim
```

200msほど速くなりました。

```shell
zsh -i -c exit  0.53s user 0.24s system 67% cpu 1.142 total
zsh -i -c exit  0.55s user 0.21s system 69% cpu 1.098 total
zsh -i -c exit  0.56s user 0.21s system 72% cpu 1.063 total
zsh -i -c exit  0.51s user 0.21s system 71% cpu 0.997 total
zsh -i -c exit  0.51s user 0.21s system 70% cpu 1.016 total
zsh -i -c exit  0.54s user 0.21s system 71% cpu 1.048 total
zsh -i -c exit  0.55s user 0.21s system 73% cpu 1.026 total
zsh -i -c exit  0.52s user 0.20s system 72% cpu 0.990 total
zsh -i -c exit  0.55s user 0.20s system 72% cpu 1.023 total
zsh -i -c exit  0.52s user 0.20s system 72% cpu 0.988 total
```

### `aqua root-dir` も環境変数に置き換える
一部のツールはaquaを使っていたので、aquaの設定も入っていました。こちらも、`aqua root-dir` コマンドに時間がかかっているようでした。

以下を参考にコマンドを使わない形式にしました。

https://aquaproj.github.io/docs/install/#2-set-the-environment-variable-path

```diff
- export PATH="$(aqua root-dir)/bin:$PATH"
+ export PATH="${AQUA_ROOT_DIR:-${XDG_DATA_HOME:-$HOME/.local/share}/aquaproj-aqua}/bin:$PATH"
```

少しだけ速くなったかも

```shell
zsh -i -c exit  0.57s user 0.38s system 55% cpu 1.719 total
zsh -i -c exit  0.42s user 0.18s system 65% cpu 0.913 total
zsh -i -c exit  0.41s user 0.18s system 66% cpu 0.891 total
zsh -i -c exit  0.44s user 0.19s system 68% cpu 0.906 total
zsh -i -c exit  0.42s user 0.18s system 67% cpu 0.904 total
zsh -i -c exit  0.42s user 0.18s system 58% cpu 1.032 total
zsh -i -c exit  0.43s user 0.19s system 66% cpu 0.934 total
zsh -i -c exit  0.41s user 0.18s system 64% cpu 0.911 total
zsh -i -c exit  0.43s user 0.18s system 68% cpu 0.899 total
zsh -i -c exit  0.42s user 0.18s system 66% cpu 0.905 total
```

### コマンド結果をeval/sourceしている箇所をファイルか読み込むようにする

各種コマンドなどでsourceを使っているところが、遅くなっている原因のようでした。以下のようなコードです。

```shell
source <(kubectl completion zsh)
```

これらのsourceやevalを試しに全てコメントアウトしたら、かなり速くなりました。

```shell
zsh -i -c exit  0.12s user 0.08s system 88% cpu 0.236 total
zsh -i -c exit  0.13s user 0.09s system 83% cpu 0.252 total
zsh -i -c exit  0.12s user 0.08s system 84% cpu 0.243 total
zsh -i -c exit  0.13s user 0.09s system 84% cpu 0.263 total
zsh -i -c exit  0.13s user 0.09s system 85% cpu 0.253 total
zsh -i -c exit  0.13s user 0.09s system 79% cpu 0.272 total
zsh -i -c exit  0.13s user 0.09s system 87% cpu 0.244 total
zsh -i -c exit  0.14s user 0.09s system 86% cpu 0.264 total
zsh -i -c exit  0.14s user 0.09s system 82% cpu 0.280 total
zsh -i -c exit  0.13s user 0.09s system 85% cpu 0.263 total
```

なので、これらをどうにかする方法を考えます。
source/evalを使っているのは補完目的です。常にコマンドを実行して、最新を取得する必要はありません。そのため、事前にファイルにコマンドの結果を出力しておいて、そちらをsourceして、コマンド出力結果の更新は非同期で実行すればよいと考えました。

※ 最終的には別のスクリプトで定期的にファイル出力を実行するようにしたので、`.zshrc`としてはsourceするだけとなりました。

```shell
local script_dir="$HOME/.zsh/local.script"
# load local scripts
for file in ${script_dir}/*; do
  [ -f "$file" ] && source "$file"
done
# setup local scripts for next time
function local_script_setup() {
  local command="$1"
  if [ -n "$command" ] && ! type "$command" &>/dev/null; then
    return
  fi
  local script_command="$2"
  local script_file="${script_dir}/_$command"
  if [ ! -f "$script_file" ]; then
    eval "source <($script_command)"
  fi
  (eval "$script_command > $script_file" &) > /dev/null 2>&1
}
local_script_setup "kubectl" "kubectl completion zsh"
local_script_setup "lovot" "lovot completion zsh"
local_script_setup "atlas" "atlas completion zsh"
local_script_setup "direnv" "direnv hook zsh"
local_script_setup "npm" "npm completion"
```

起動時は事前に保存したファイルをsourceしかつ、非同期でファイルを更新するようにできました。変更があれば、次回から適用されることになるので、そこだけ注意です。

あまり知見がないので、もっとよいやり方があるかもしれません。

これで300msまで速くなりました。

```shell
zsh -i -c exit  0.29s user 0.22s system 89% cpu 0.576 total
zsh -i -c exit  0.13s user 0.10s system 76% cpu 0.306 total
zsh -i -c exit  0.13s user 0.09s system 74% cpu 0.297 total
zsh -i -c exit  0.14s user 0.10s system 80% cpu 0.288 total
zsh -i -c exit  0.14s user 0.10s system 82% cpu 0.289 total
zsh -i -c exit  0.13s user 0.09s system 78% cpu 0.278 total
zsh -i -c exit  0.13s user 0.10s system 80% cpu 0.288 total
zsh -i -c exit  0.14s user 0.11s system 83% cpu 0.298 total
zsh -i -c exit  0.13s user 0.10s system 82% cpu 0.276 total
zsh -i -c exit  0.14s user 0.10s system 82% cpu 0.287 total
```


### sheldonのpluginでdeferしてみる

プラグインマネージャーとして、sheldonを利用して、プラグインをいくつか管理していましたが、特に遅延読み込みしていませんでした。

https://sheldon.cli.rs/Examples.html を参考に以下のように設定しました。


```toml
shell = "zsh"

[plugins]

[plugins.zsh-defer]
github = "romkatv/zsh-defer"

[templates]
defer = "{{ hooks?.pre | nl }}{% for file in files %}zsh-defer source \"{{ file }}\"\n{% endfor %}{{ hooks?.post | nl }}"

[plugins.fast-syntax-highlighting]
github = 'zdharma-continuum/fast-syntax-highlighting'
apply = ["defer"]

[plugins.zsh-completions]
github = 'zsh-users/zsh-completions'

[plugins.zsh-autocomplete]
github = 'marlonrichert/zsh-autocomplete'

[plugins.jq-zsh-plugin]
github = "reegnz/jq-zsh-plugin"
apply = ["defer"]

[plugins.zeno]
github = "yuki-yano/zeno.zsh"
apply = ["defer"]
```

その結果、200ms台が安定して出るようになりました。

```shell
zsh -i -c exit  0.11s user 0.09s system 81% cpu 0.244 total
zsh -i -c exit  0.11s user 0.09s system 87% cpu 0.230 total
zsh -i -c exit  0.11s user 0.09s system 79% cpu 0.258 total
zsh -i -c exit  0.10s user 0.09s system 84% cpu 0.224 total
zsh -i -c exit  0.10s user 0.08s system 85% cpu 0.206 total
zsh -i -c exit  0.09s user 0.08s system 82% cpu 0.210 total
zsh -i -c exit  0.11s user 0.09s system 84% cpu 0.241 total
zsh -i -c exit  0.10s user 0.09s system 80% cpu 0.235 total
zsh -i -c exit  0.10s user 0.09s system 80% cpu 0.234 total
zsh -i -c exit  0.11s user 0.08s system 79% cpu 0.237 total
```

### brewのshellenvも同様にファイルから読み込むように

これを

```shell
eval "$(/opt/homebrew/bin/brew shellenv)"
```

以下のように置き換えました。

```shell
local brew_path="$HOMEBREW_PREFIX/bin/brew"
local brew_script_path="${script_dir}/_brew"
if [ ! -f "$brew_script_path" ]; then
  eval "source <($brew_path shellenv)"
fi
(eval "$brew_path shellenv > $brew_script_path" &) > /dev/null 2>&1
```

結果、やや速くなったような...？

```shell
zsh -i -c exit  0.22s user 0.17s system 86% cpu 0.455 total
zsh -i -c exit  0.08s user 0.07s system 72% cpu 0.201 total
zsh -i -c exit  0.07s user 0.06s system 56% cpu 0.241 total
zsh -i -c exit  0.07s user 0.07s system 68% cpu 0.204 total
zsh -i -c exit  0.08s user 0.07s system 66% cpu 0.229 total
zsh -i -c exit  0.08s user 0.07s system 58% cpu 0.259 total
zsh -i -c exit  0.08s user 0.08s system 65% cpu 0.233 total
zsh -i -c exit  0.09s user 0.07s system 74% cpu 0.214 total
zsh -i -c exit  0.07s user 0.07s system 69% cpu 0.201 total
zsh -i -c exit  0.09s user 0.08s system 66% cpu 0.247 total
```

### sourceでファイルを読込んでいる箇所を遅延読み込みする

直接ファイルをsourceしている箇所が時間がかかってそうでした。sheldonでzsh-deferをインストールしたので、それを使うようにしました。

```shell
# autopair
zsh-defer source $HOMEBREW_PREFIX/share/zsh-autopair/autopair.zsh
# gcloud
zsh-defer -c "test -e $HOME/google-cloud-sdk/path.zsh.inc && source $HOME/google-cloud-sdk/path.zsh.inc || true"
zsh-defer -c "test -e $HOME/google-cloud-sdk/completion.zsh.inc && source $HOME/google-cloud-sdk/completion.zsh.inc || true"
# iterm2
zsh-defer -c "test -e $HOME/.iterm2_shell_integration.zsh && source $HOME/.iterm2_shell_integration.zsh || true"
```

優位に速くなりました。

```shell
zsh -i -c exit  0.21s user 0.18s system 76% cpu 0.513 total
zsh -i -c exit  0.08s user 0.08s system 92% cpu 0.178 total
zsh -i -c exit  0.08s user 0.09s system 89% cpu 0.188 total
zsh -i -c exit  0.09s user 0.08s system 91% cpu 0.179 total
zsh -i -c exit  0.08s user 0.08s system 90% cpu 0.177 total
zsh -i -c exit  0.08s user 0.07s system 87% cpu 0.180 total
zsh -i -c exit  0.09s user 0.09s system 91% cpu 0.190 total
zsh -i -c exit  0.09s user 0.08s system 91% cpu 0.182 total
zsh -i -c exit  0.09s user 0.08s system 91% cpu 0.188 total
zsh -i -c exit  0.09s user 0.08s system 86% cpu 0.194 total
```

### weztermをやめてalacrittyにする

このあたりで、手がなくなってきたので、weztermではなくalacrittyを試しに使ってみました。100msくらい速くなりました。こんなに変わるとは...

```shell
zsh -i -c exit  0.04s user 0.03s system 80% cpu 0.088 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.072 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.072 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.072 total
zsh -i -c exit  0.04s user 0.03s system 90% cpu 0.072 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.072 total
zsh -i -c exit  0.04s user 0.03s system 90% cpu 0.074 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.073 total
zsh -i -c exit  0.04s user 0.03s system 90% cpu 0.073 total
zsh -i -c exit  0.04s user 0.03s system 91% cpu 0.073 total
```

### source対象のファイル出力を別のスクリプトで行うようにした

前述したsource対象のファイル出力(brew shellenvの出力含む)ですが、非同期なのにやや時間がかかっているようなので、定期的に実行しているスクリプトで実行するようにしました。

その結果、.zshrcとしてはシンプルにこんな感じとなりました。

```shell
for file in $HOME/.zsh/scripts/*; do
  [ -f "$file" ] && source "$file"
done
```

10ms程度ですが、明らかに効果ありました。

```shell
zsh -i -c exit  0.04s user 0.03s system 79% cpu 0.086 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.067 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.066 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.067 total
zsh -i -c exit  0.04s user 0.03s system 92% cpu 0.071 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.067 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.066 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.066 total
zsh -i -c exit  0.04s user 0.02s system 93% cpu 0.066 total
zsh -i -c exit  0.04s user 0.02s system 92% cpu 0.066 total
```

## おわりに

- コマンド実行とsourceがたいていの原因だった
- 20倍速くなったぜ！
- alacritty最強！

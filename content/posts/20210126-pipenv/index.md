+++
title = "pipenvがnpmみたいに使えて便利そう"
date = 2021-01-26T00:00:00+09:00
tags = [
  "Python",
  "pipenv",
]
+++

## まえがき

最近久々にPythonを触ってみました。

Pythonといえばバージョン管理を考えるのが厄介だなという印象でした。


なにぶんmacosではデフォルトでPython2が入っているので、Python3を使うにはバージョン管理ソフトが必須となります。
バージョン管理のツールだとpyenvやらvirtualenvやら入り乱れていて、何がデファクトスタンダードだかよくわからないのが実情だったと思います。


今回あらためてPythonを書くにあたり調べたところ、どうやら [pipenv](https://github.com/pypa/pipenv) がもっとも有力なバージョン管理であり、さらにパッケージの管理までしてしてくれるとのことで便利そうだなと思い使ってみました。

結論としては、 `pipenv` はバージョン管理+パッケージ管理ができる素晴らしいツールですが、ややデメリットもあるかなというところです。

### 導入(mac)

READMEには書かれていないですが普通にbrewでinstallできます。

```bash
brew install pipenv
```

Pythonのディレクトリで以下を実施して仮想環境を初期化します。このときPythonのバージョンを指定して、そのバージョンで初期化することができます。

```bash
pipenv --Python 3.8
```

またこのときデフォルトではグローバルな場所に環境が構築されますが、環境変数を設定することでローカルな環境( `.venv` 配下)に構築することが可能です。

https://pipenv-ja.readthedocs.io/ja/translate-ja/advanced.html#pipenv.environments.PIPENV_VENV_IN_PROJECT

VSCodeでは `.venv` にPython環境があった方がインタプリターを指定しやすいので、なるべく設定した方が良いと思われ。

初期化すると以下のように `Pipfile` が生成されます。`Pipfile` がnpmでいうpackage.jsonに当たるファイルのようです。

```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]

[requires]
python_version = "3.8"
```

何やらnpmと同じような雰囲気を感じますね。

### pipenv環境に入るには

`pipenv shell` を用います。その後、pipenv環境で `python` コマンドなどが使えるようになります。
ちょっとした動作確認などnpmでいう `npx` と同じような目的で使われる機能ですね。

```bash
$ pipenv shell
Launching subshell in virtual environment...
 . /Users/mineo/.local/share/virtualenvs/test-wjEkysQk/bin/activate
$ python app.py
```

### パッケージ管理

#### packageを追加

```bash
pipenv install numpy
```

```bash
Installing numpy...
Adding numpy to Pipfile's [packages]...
✔ Installation Succeeded
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success!
Updated Pipfile.lock (456e4b)!
Installing dependencies from Pipfile.lock (456e4b)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

`Pipfile` が以下のようになり `numpy` が追加されました。また、 `Pipfile.lock` が生成されます。 npmでいうpackage-lock.jsonに当たるファイルのようです。

```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
numpy = "*"

[dev-packages]

[requires]
python_version = "3.8"
```

また、devでinstallする場合は `--dev` をつけるようです

#### packageを削除

```bash
pipenv uninstall numpy
```

```bash
Installing numpy...
Adding numpy to Pipfile's [packages]...
✔ Installation Succeeded
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success!
Updated Pipfile.lock (456e4b)!
Installing dependencies from Pipfile.lock (456e4b)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

#### requirements.txtに出力

何かと必要な `requirements.txt` にpackageを出力することもできます。

https://pipenv-ja.readthedocs.io/ja/translate-ja/advanced.html#generating-a-requirements-txt

```bash
pipenv lock -r > requirements.txt
```

### Pipfileから環境構築

pipenvの仮想環境が構築され、Pipfileのpackageがまとめてinstallされます。

```bash
pipenv install
```

devも含めてinstall

```bash
pipenv install --dev
```

#### 構築した環境を削除

以下でinstallしたpackageやPythonの仮想環境などを全て削除します。

```bash
pipenv --rm
```

npmだとnode_modulesを手動で消さないといけないので、この点はnpmより使い勝手が良いですね。

### scripts

pipenvでもnpmのように任意のscriptsを設定することができます。

```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[scripts]
start = "python app.py"

[packages]
numpy = "*"

[dev-packages]

[requires]
python_version = "3.8"
```

```bash
pipenv run start
```

ただし、ここで気をつけないといけないのはscriptをで使えない文字列があることです。例えば、以下のように `requirements.txt` に出力するscriptsを設定した場合、残念な結果になります。

```
[scripts]
exportPackage = "pipenv lock -r > requirements.txt"
```

```bash
$ pipenv run exportPackage
Usage: pipenv lock [OPTIONS]
Try 'pipenv lock -h' for help.

Error: Got unexpected extra arguments (> requirements.txt)
```

また、 `&&` も使えないため複数のscriptを組み合わせることもできません。私的にはこれが一番残念な点でした。
issueで議論されていましたが、どうもpipenvではPipfileではなく `invoke` というタスクランナーを使って解決するべきという結論のようです。

https://github.com/pypa/pipenv/issues/2038#issuecomment-505493265

## おわりに

scripts で `&&` が使えないなど少し残念な点はありましたが、バージョン管理を含めた環境構築ができるので、Pipenvはチーム開発に適したツールだと思います。全体的な使い勝手もnpmに近く直感的にわかりやすいです。

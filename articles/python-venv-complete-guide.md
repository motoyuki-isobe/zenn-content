---
title: "Python仮想環境(venv)の使い方を完全理解する"
emoji: "🐍"
type: "tech"
topics: ["Python", "初心者", "環境構築", "venv"]
published: true
---

## はじめに

Pythonでの開発を始めると、すぐにぶつかるのが「パッケージの依存関係」の問題です。プロジェクトAではDjango 4.2を、プロジェクトBではDjango 5.0を使いたい——こんな状況で活躍するのが **仮想環境（venv）** です。

この記事では、Pythonの標準機能である `venv` の仕組みから実践的な使い方まで、ハンズオン形式で解説します。記事のコマンドを順番に実行すれば、仮想環境の作成・有効化・パッケージ管理・削除まで一通り体験できます。

## 前提条件

- Python 3.3 以上がインストール済み（3.12以上推奨）
- ターミナル（macOS/Linux）またはコマンドプロンプト/PowerShell（Windows）の基本操作

バージョン確認:

```bash
python3 --version
# Python 3.12.x と表示されればOK
```

## そもそも仮想環境とは？

仮想環境は、**プロジェクトごとに独立したPython実行環境を作る仕組み**です。

仮想環境を使わない場合、`pip install` したパッケージはすべてグローバル（システム全体）にインストールされます。すると：

- プロジェクトAで入れたパッケージがプロジェクトBに影響する
- バージョン違いで動かなくなる
- 「自分のPCでは動くのに本番で動かない」が発生する

仮想環境を使えば、プロジェクトごとにパッケージが隔離されるため、これらの問題を完全に回避できます。

```
グローバル環境（使わない）    仮想環境（推奨）
┌──────────────┐        ┌──────────┐  ┌──────────┐
│ Django 4.2   │        │ Project A│  │ Project B│
│ Flask 3.0    │        │ Django4.2│  │ Django5.0│
│ requests 2.31│        │ Flask3.0 │  │ FastAPI  │
│ 全部混在...   │        └──────────┘  └──────────┘
└──────────────┘          独立！         独立！
```

## ステップ1: 仮想環境を作成する

まず、プロジェクト用のディレクトリを作り、その中に仮想環境を作成します。

```bash
# プロジェクトディレクトリを作成
mkdir my-python-project
cd my-python-project

# 仮想環境を作成（.venv という名前が慣例）
python3 -m venv .venv
```

`.venv` ディレクトリが作成されます。中身を確認してみましょう：

```bash
ls -la .venv/
# bin/     include/  lib/     pyvenv.cfg
```

| ディレクトリ | 役割 |
|------------|------|
| `bin/` (macOS/Linux) / `Scripts/` (Windows) | Python実行ファイルとactivateスクリプト |
| `lib/` | この環境専用のパッケージ置き場 |
| `pyvenv.cfg` | 仮想環境の設定ファイル |

:::message
`.venv` というディレクトリ名は慣例です。`venv`、`env`、`.env` なども使われますが、`.venv` が最も一般的で、多くのツール（VS Code等）がデフォルトで認識します。
:::

## ステップ2: 仮想環境を有効化する

作成しただけでは使えません。**有効化（activate）** する必要があります。

**macOS / Linux:**

```bash
source .venv/bin/activate
```

**Windows（PowerShell）:**

```powershell
.venv\Scripts\Activate.ps1
```

**Windows（コマンドプロンプト）:**

```cmd
.venv\Scripts\activate.bat
```

有効化されると、ターミナルのプロンプトが変わります：

```bash
# 有効化前
$

# 有効化後（先頭に (.venv) が付く）
(.venv) $
```

この状態で `python` や `pip` を実行すると、仮想環境内のものが使われます。確認してみましょう：

```bash
# どのPythonが使われているか確認
which python
# /Users/yourname/my-python-project/.venv/bin/python

# pipの場所も確認
which pip
# /Users/yourname/my-python-project/.venv/bin/pip
```

## ステップ3: パッケージをインストールする

仮想環境が有効な状態で、通常通り `pip install` します。

```bash
# パッケージをインストール
pip install requests flask

# インストール済みパッケージを確認
pip list
```

出力例：

```
Package    Version
---------- -------
blinker    1.9.0
click      8.1.7
Flask      3.1.0
...
requests   2.32.3
```

このパッケージは `.venv/lib/` 内にインストールされるため、他のプロジェクトには一切影響しません。

## ステップ4: requirements.txt で依存関係を管理する

チーム開発やデプロイ時に「どのパッケージが必要か」を共有するために、`requirements.txt` を作成します。

```bash
# 現在のパッケージ一覧をファイルに出力
pip freeze > requirements.txt
```

`requirements.txt` の中身：

```
blinker==1.9.0
click==8.1.7
Flask==3.1.0
itsdangerous==2.2.0
Jinja2==3.1.4
MarkupSafe==3.0.2
requests==2.32.3
urllib3==2.2.3
```

別の環境で同じパッケージを再現するには：

```bash
# 別の環境で仮想環境を作成・有効化した後
pip install -r requirements.txt
```

これで全く同じバージョンのパッケージが再現されます。

:::message alert
`requirements.txt` はGitにコミットしますが、`.venv/` ディレクトリ自体はコミットしません。`.gitignore` に追加しておきましょう。
:::

```bash
# .gitignore に追加
echo ".venv/" >> .gitignore
```

## ステップ5: 仮想環境を無効化・削除する

**無効化（一時的に抜ける）:**

```bash
deactivate
```

プロンプトから `(.venv)` が消え、グローバル環境に戻ります。

**削除（完全に消す）:**

仮想環境はただのディレクトリなので、削除するだけです。

```bash
# 無効化してから削除
deactivate
rm -rf .venv
```

再度必要になったら `python3 -m venv .venv` で作り直せます。`requirements.txt` があれば、パッケージも一発で復元できます。

## よくあるエラーと対処法

### `python3 -m venv` が動かない

```
Error: Command 'python3' not found
```

**対処法:** Pythonがインストールされていないか、パスが通っていません。

```bash
# macOS (Homebrew)
brew install python

# Ubuntu/Debian
sudo apt install python3 python3-venv
```

Ubuntu/Debianでは `python3-venv` パッケージが別途必要な点に注意してください。

### `activate` が見つからない

```
bash: .venv/bin/activate: No such file or directory
```

**対処法:** カレントディレクトリが正しいか確認してください。`.venv` はプロジェクトルートに作成されます。

```bash
# 現在のディレクトリを確認
pwd
ls -la .venv/bin/activate
```

### Windows PowerShellで実行ポリシーエラー

```
.venv\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled
```

**対処法:**

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## VS Code での仮想環境の使い方

VS Codeは `.venv` ディレクトリを自動で検出します。

1. プロジェクトフォルダをVS Codeで開く
2. `Ctrl+Shift+P`（macOSは `Cmd+Shift+P`）でコマンドパレットを開く
3. 「Python: Select Interpreter」を選択
4. `.venv` 内のPythonが候補に表示されるので選択

これで、VS Code内のターミナルやデバッガーが自動的に仮想環境を使うようになります。

## まとめ

| 操作 | コマンド |
|------|---------|
| 作成 | `python3 -m venv .venv` |
| 有効化 | `source .venv/bin/activate` |
| パッケージ追加 | `pip install パッケージ名` |
| 依存関係出力 | `pip freeze > requirements.txt` |
| 依存関係復元 | `pip install -r requirements.txt` |
| 無効化 | `deactivate` |
| 削除 | `rm -rf .venv` |

venvはPython開発の基本中の基本です。プロジェクトを始めるときは、最初に `python3 -m venv .venv` を実行する癖をつけましょう。

Pythonを含むWebエンジニアのスキルをより体系的に学びたい方は、[Webエンジニアへの学習ロードマップ全体像](https://techcareer.media/articles/web-engineer-roadmap)も参考にしてみてください。

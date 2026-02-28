いいですね、では完成版としてまとめます。
NOTE向けに、体験＋整理の構成で書きます。

---

# CMUXを入れようとしたら、CMUXが多すぎて混乱した話

こんにちはヤマグチです。最近はClaude CodeやCodexなんかを使ってアプリを作るのにハマっております。

Claude Codeのおかげで、ターミナルに接する機会が増えまして、たいぶ仲良くなってきました（笑）

そんなある日、cmuxというターミナルエミュレータがあるということを知りました。ペインを分割したり、ブラウザーを表示したり、Workspaceを切り替えたりできるそうです。これは便利！！

まずはbrewでインストール

```bash
brew tap manaflow-ai/cmux
brew install --cask cmux
```

確認。

```bash
cmux --version
```

結果：

```
zsh: command not found: cmux
```

え？入ったんじゃないの？

ここから混乱が始まった。

---

# 原因：CMUXは1種類ではない

実はcmuxには3種類ありました。

---

## ① Goライブラリ版 cmux

* gRPCとHTTPを同一ポートで動かすためのもの
* Goコードに組み込む
* CLIではない

これはサーバー開発用。

---

## ② CLI版 cmux

* コマンドラインツール
* ターミナル上で動く
* 自動化やスクリプトに組み込める
* GUIなし

使い方のイメージ：

```bash
cmux run task
cmux start
cmux attach
```

つまり：

> 既存のターミナルの中で動くツール

---

## ③ macOSアプリ版 cmux（今回入ったもの）

* GUIターミナルアプリ
* Ghosttyベース
* 縦タブUI
* AI通知機能あり
* `/Applications/cmux.app` に入る

つまり：

> 見た目付きのターミナルアプリ

---

# 参考：CLI版とアプリ版の違い

|         | アプリ版  | CLI版     |
| ------- | ----- | -------- |
| 起動方法    | クリック  | コマンド     |
| 画面      | GUIあり | テキスト     |
| スクリプト組込 | ×     | ◎        |
| 自動化     | △     | ◎        |
| 目的      | 作業環境  | ワークフロー制御 |

一番シンプルに言うと：

* アプリ版 → iTermに近い
* CLI版 → tmuxやdockerコマンドに近い

---

# なぜ `cmux` コマンドが使えなかったのか

Homebrewには2種類ある。

* Formula → CLIツール
* Cask → macOSアプリ

今回入ったのは Cask。

だから、

```bash
cmux --version
```

は存在しない。

正しい起動方法はこれ。

```bash
open -a cmux
```

または、アプリケーションフォルダからクリック。

---

# 今回の混乱ポイント

今回の問題は、**brewの **``tap`` と ``--cask``** という命令を理解せずに使ったこと**でした。

```bash
brew tap manaflow-ai/cmux
brew install --cask cmux
```

## 1) `brew tap <url>` は「追加の置き場を登録」する

一言でいうと：
Homebrew に「cmux 用の追加レポジトリ（タップ）」を登録して、cmux を brew install できるようにするコマンドです。もう少し具体的には、Homebrew に「manafow-ai/cmux」という Git リポジトリをタップとして追加します。

>実体は https://github.com/manaflow-ai/cmux　（またはそれに対応する Homebrew 用リポジトリ）をローカルにクローンして登録します。
>これにより、そのリポジトリ内の formula / cask（ここでは cmux）を brew install できるようになります。

このコマンド自体は cmux のインストールまでは行わず、「レポジトリの追加だけ」で、インストールには別途 `brew install`が必要です。

---

## 2) `brew install`の `--cask` は「macOSアプリ」をインストール

```bash
brew install --cask cmux
```

ここが最大のポイント。

`--cask` は、CLIコマンドではなく、macOSアプリ（.app）をインストールするという指定でした。

`--cask` で入れたものは、基本的に「コマンド」ではないので

```bash
cmux --version
```

が `command not found` になったわけです。

正しい起動法は、Finderからアイコンをクリックして起動することでした。

ターミナルから起動するには、以下が正解でした。

```bash
open -a cmux
```

---

# ついでに整理：`--cask` と `--formula` の違い

Homebrewには大きく2種類ある。

## `--formula`（＝CLIツール）

- ターミナルから実行するコマンドが入る
- 例：node / git / ffmpeg

```bash
brew install --formula node
node -v
```

## `--cask`（＝macOSアプリ）

- Applicationsフォルダにアプリが入る
- 例：Chrome / Slack / Docker Desktop

```bash
brew install --cask google-chrome
open -a "Google Chrome"
```

じゃあ、`brew install cmux` みたいに、`brew install` だけ打ったときはどうなるのかというと、**デフォルトは基本的に `--formula`（CLIツール）扱い**です。

---

# 今回の教訓

ターミナルは便利だけど、コマンドを知らずに使うと思わぬ落とし穴にハマりますね。

知らないコマンドは、面倒くさがらずにいちいち調べていきたいと思います。

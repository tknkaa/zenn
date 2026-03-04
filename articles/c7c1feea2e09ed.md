---
title: "Copilot CLIのセッションを簡単に再開できるCLIツールを作りました。"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "githubcopilot", "cli"]
published: true 
---
# はじめに
私は普段から Copilot CLI をよく使っているのですが、Copilot CLI のセッションを再開するにはセッションの ID を指定する必要があり、それだとセッションを再開するのが少し面倒だったので、もっと簡単にセッションを再開できる CLI ツールを作りました。
Copilot Session Picker ということで、csp という名前にしました。
# 使い方
cargo でインストールします。
```sh
cargo install --git https://github.com/tknkaa/csp
```
実行します。
```sh
csp
```
以下のように使えます。
![デモ動画](/images/csp.gif)
Copilot CLI は `~/.copilot/session-state` 以下に各セッションの情報を JSON で保存するので、そこから各セッションの一番最初のプロンプト、作業ディレクトリ、更新日時などを取得しています。セッションの要約を AI にやらせることも考えたのですが、要約させる適当なタイミングが思いつかなかったので、最初のプロンプトを表示するようにしています。
また、単に Copilot CLI から再開するだけだとそのセッションの作業ディレクトリに cd してくれないので、csp では cd してからセッションを再開するようにしています。セッションの選択は矢印キーでも hjkl でもできるようにしています。
# 終わりに
是非使ってみてください。また、あったらいい機能を思いついたら、Zenn でも GitHub でも是非教えてください。検索機能があったらいいな、と思っているので、気が向いたら追加します。
https://github.com/tknkaa/csp

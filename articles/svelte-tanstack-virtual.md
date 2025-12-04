---
title: "Svelte 5 で TanStack Virtual を使う"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Svelte", "アドベントカレンダー"]
published: false
---
:::message
ut.code(); Advent Calendar 5 日目です。
:::
# はじめに
普段開発している中で、Svelte 5 で Virtual Scroll を実装する場面があり、そのやり方がよく分からなかったので、とりあえず上手くいった方法を共有したいと思います。
# Virtual Scroll とは
Virtual Scroll とは、大量のリストがあったときに、リスト全体を DOM に展開するのではなく、現在のスクロール位置で実際に見えている項目（とその近辺の項目）だけをレンダリングする技術のことです。SNS のタイムラインやスプレッドシートが例として分かりやすいかと思います。
実装するときには、TanStack Virtual や 各フレームワークごとにライブラリ（Svelte なら svelte-virtual-list）を使うのが簡単です。今回は公式ドキュメントの充実さから、TanStack Virtual を採用しました。
# Svelte 5 + TanStack Virtual
# さいごに
多分もっといい方法があるはずなので、思いついた方は是非コメントで教えてくださると助かります。

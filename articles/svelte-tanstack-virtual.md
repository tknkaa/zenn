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
普段開発している中で、Svelte 5 で Virtual Scroll を実装する場面があり、そのやり方がよく分からなかったので、とりあえず上手くいった方法を共有したいと思います。具体的にはスプレッドシートを実装します。
# Virtual Scroll とは
Virtual Scroll とは、大量のリストがあったときに、リスト全体を DOM に展開するのではなく、現在のスクロール位置で実際に見えている項目（とその近辺の項目）だけをレンダリングする技術のことです。SNS のタイムラインやスプレッドシートが例として分かりやすいかと思います。
実装するときには、TanStack Virtual や 各フレームワークごとにライブラリ（Svelte なら svelte-virtual-list）を使うのが簡単です。今回は公式ドキュメントの充実さから、TanStack Virtual を採用しました。
# Svelte 5 + TanStack Virtual
まずは、公式ドキュメントの例を見てみます。この例では、まず HTMLDivElement を定義して、`$:` によってその変更を追い、bind されたタイミングで `rowVirtualizer` と `columnVirtualizer` を初期化しています。
```ts:+page.svelte
<script lang="ts">
  import { createVirtualizer } from '@tanstack/svelte-virtual'

  let virtualListEl: HTMLDivElement

  $: rowVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
    count: 10000,
    getScrollElement: () => virtualListEl,
    estimateSize: () => 35,
    overscan: 5,
  })

  $: columnVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
    horizontal: true,
    count: 10000,
    getScrollElement: () => virtualListEl,
    estimateSize: () => 100,
    overscan: 5,
  })
</script>

<div class="list scroll-container" bind:this={virtualListEl}>
  <div
    style="position: relative; height: {$rowVirtualizer.getTotalSize()}px; width: {$columnVirtualizer.getTotalSize()}px;"
  >
    {#each $rowVirtualizer.getVirtualItems() as row (row.index)}
      {#each $columnVirtualizer.getVirtualItems() as col (col.index)}
        <div
          class={col.index % 2
            ? row.index % 2 === 0
              ? 'list-item-odd'
              : 'list-item-even'
            : row.index % 2
              ? 'list-item-odd'
              : 'list-item-even'}
          style="position: absolute; top: 0; left: 0; width: {col.size}px; height: {row.size}px; transform: translateX({col.start}px) translateY({row.start}px);"
        >
          Cell {row.index}, {col.index}
        </div>
      {/each}
    {/each}
  </div>
</div>

<style>
  .scroll-container {
    height: 500px;
    width: 500px;
    overflow: auto;
  }
</style>
```
[公式ドキュメント]( https://tanstack.com/virtual/latest/docs/framework/svelte/examples/fixed?path=examples%2Fsvelte%2Ffixed%2Fsrc%2FGridVirtualizerFixed.svelte)より抜粋（Svelte のシンタックスハイライトが効かないので、 TypeScript として表示しています。）

これは Svelte 5 ではないので、 runes を使って書き直していきます。

Svelte ５ に移行するときには、 `$:` を　`$derived` または `$effect` で書き直すのですが、今回は `createVirtualizer` が イベントリスナーを登録しているので、 副作用として `$effect` で管理するのが適切かと思います。また、 `$effect` の中で更新するので、 `rowVirtualzer` と `columnVirtualizer` も `$state` で宣言します。 型は `createVirtualizer<HTMLDivElement, HTMLDivElement>` の返り値の型である、`Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>` となり、 `undefined` で初期化します。
```ts:+page.svelte
<script lang="ts">
    import { createVirtualizer } from "@tanstack/svelte-virtual";
    import type { Readable } from "svelte/store";
    import type { SvelteVirtualizer } from "@tanstack/svelte-virtual";

    let virtualListEl: HTMLDivElement;

    let rowVirtualizer:
        | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
        | undefined = $state();

    let columnVirtualizer:
        | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
        | undefined = $state();

    $effect(() => {
        rowVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
            count: 10000,
            getScrollElement: () => virtualListEl,
            estimateSize: () => 35,
            overscan: 5,
        });
        columnVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
            horizontal: true,
            count: 10000,
            getScrollElement: () => virtualListEl,
            estimateSize: () => 100,
            overscan: 5,
        });
    });
</script>

<div class="list scroll-container" bind:this={virtualListEl}>
    <div
        style="position: relative; height: {$rowVirtualizer?.getTotalSize()}px; width: {$columnVirtualizer?.getTotalSize()}px;"
    >
        {#each $rowVirtualizer?.getVirtualItems() as row (row.index)}
            {#each $columnVirtualizer?.getVirtualItems() as col (col.index)}
                <div
                    class={col.index % 2
                        ? row.index % 2 === 0
                            ? "list-item-odd"
                            : "list-item-even"
                        : row.index % 2
                          ? "list-item-odd"
                          : "list-item-even"}
                    style="position: absolute; top: 0; left: 0; width: {col.size}px; height: {row.size}px; transform: translateX({col.start}px) translateY({row.start}px);"
                >
                    Cell {row.index}, {col.index}
                </div>
            {/each}
        {/each}
    </div>
</div>

<style>
    .scroll-container {
        height: 500px;
        width: 500px;
        overflow: auto;
    }
</style>
```
# セルがレンダリングされたタイミングで、各セルの状態を初期化したい
今回はスプレッドシートを作っているので、セルごとにユーザー入力を受け取って、その入力をグローバルな状態として反映させたいです。ナイーブな方法としては、1000 * 1000 の二次元配列をリアクティブな値として定義して、それを virtual scroll によってレンダリングする方法が考えられますが、それだとリアクティブな値を定義するのに莫大な計算コストがかかってしまい、動作がとても重くなってしまいます。
そこで、レンダリングされたセルにだけ初期化する方法（遅延初期化）を実装してみたいと思います。
```ts:+page.svelte
<script lang="ts">
  import { createVirtualizer } from "@tanstack/svelte-virtual";
  import type { Readable } from "svelte/store";
  import type { SvelteVirtualizer } from "@tanstack/svelte-virtual";
  import Cell from "$lib/Cell.svelte";
  import type { CellType } from "$lib/Cell.svelte";

  let grid: CellType[][] = $state([]);
  function getCell(x: number, y: number): CellType {
    if (!grid[y]) {
      grid[y] = [];
    }
    let cell = grid[y][x];
    if (!cell) {
      cell = {
        x,
        y,
        value: "",
      };
      grid[y][x] = cell;
    }
    return cell;
  }
  function setCell(cell: CellType) {
    grid[cell.y][cell.x] = cell;
  }

  let virtualListEl: HTMLDivElement;

  let rowVirtualizer:
    | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
    | undefined = $state();

  let columnVirtualizer:
    | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
    | undefined = $state();

  $effect(() => {
    rowVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
      count: 10000,
      getScrollElement: () => virtualListEl,
      estimateSize: () => 35,
      overscan: 5,
    });
    columnVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
      horizontal: true,
      count: 10000,
      getScrollElement: () => virtualListEl,
      estimateSize: () => 100,
      overscan: 5,
    });
  });
</script>

<div class="list scroll-container" bind:this={virtualListEl}>
  <div
    style="position: relative; height: {$rowVirtualizer?.getTotalSize()}px; width: {$columnVirtualizer?.getTotalSize()}px;"
  >
    {#each $rowVirtualizer?.getVirtualItems() as row (row.index)}
      {#each $columnVirtualizer?.getVirtualItems() as col (col.index)}
        <div
          class={col.index % 2
            ? row.index % 2 === 0
              ? "list-item-odd"
              : "list-item-even"
            : row.index % 2
              ? "list-item-odd"
              : "list-item-even"}
          style="position: absolute; top: 0; left: 0; width: {col.size}px; height: {row.size}px; transform: translateX({col.start}px) translateY({row.start}px);"
        >
          <Cell
            bind:cell={
              () => getCell(row.index, col.index), (cell) => setCell(cell)
            }
          />
        </div>
      {/each}
    {/each}
  </div>
</div>

<style>
  .scroll-container {
    height: 500px;
    width: 500px;
    overflow: auto;
  }
</style>
```
```ts:Cell.svelte
<script lang="ts">
  export type CellType = {
    x: number;
    y: number;
    value: string;
  };

  interface Props {
    cell: CellType;
  }

  let { cell = $bindable() }: Props = $props();
</script>

<input bind:value={cell.value} />
```
こんな感じですかね。動作確認したレポジトリはこれです。

https://github.com/tknkaa/svelte-tanstack-virtual-sample

# さいごに
SvelteMap は試してみたのですが、上手く実装できなかったです。SvelteMap で実装する場合はキーはどうなるんですかね。AI と相談した感じ `${x}-${y}` とかになりそうな気がしていて、実際 `grid` を `Record<string, CellType>` で定義しても実装して動かすところまでできました。ただ、それだと毎回 `${x}-${y}` を x, y に直す必要があるので、二次元配列のほうが実装として分かりやすいのかな〜と思います。
多分もっといい方法があるはずなので、思いついた方は是非コメントで教えてくださると助かります。
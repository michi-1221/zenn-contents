---
title: "Ghostty をフォークして「透過したまま裏を操作できる」オーバーレイ端末を作った"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ghostty", "macos", "zig", "swift", "terminal"]
published: true
---

## はじめに

ターミナルエミュレータ [Ghostty](https://github.com/ghostty-org/ghostty) を背景透過にして裏のブラウザを覗く運用をしていましたが、「見えているのに直接操作できない」のがもどかしくなり、フォークしてそのまま裏の画面を触れるオーバーレイモードを実装しました。(他にもっと簡単な方法があったかもしれませんが....)

ワンキーで、ターミナルが

- **透過** して裏が透けて見え、
- **常に最前面** に浮かび、
- **クリックスルー** でマウス操作が裏にそのまま抜ける

という状態に切り替わります。もう一度押せば普通のターミナルに戻ります。


| 背景透過 | 通常（不透明） |
|:---:|:---:|
| ![背景透過した Ghostty](/images/ghostty-overlay/最初透過.png) | ![通常の不透明な Ghostty](/images/ghostty-overlay/最初非透過.png) |

:::message
macOS 専用です。配布用の設定が面倒だったので自分でビルドして使ってくださいm(_ _)m。
:::

## 何ができるのか

状態は ON / OFF の2つだけ。

| 状態 | 見た目 | 重なり | マウス |
| --- | --- | --- | --- |
| **OFF（通常）** | 不透明 | 普通 | クリックできる |
| **ON（オーバーレイ）** | 透過 | 最前面 | 裏に抜ける |

透けるのは背景だけなので、**文字は薄くなりません**。手順書やグラフをブラウザで見ながら、その上にコマンド出力を重ねて眺められます。

![オーバーレイモードで裏のブラウザを操作しているデモ GIF](/images/ghostty-overlay/サンプル.gif =600x)
*オーバーレイ ON。透過したまま、手前のターミナル越しに裏のブラウザをそのまま操作できる*

## Ghostty の構成

実装場所が2つに分かれるので、先に Ghostty の構成だけ。

- **Zig の共通コア（`src/`）** … 描画やキー入力など OS 共通の中身
- **OS ごとのガワ** … macOS 版は **Swift（`macos/`）** でウィンドウやメニューを担当

この2層が **C ABI** でつながっていて、Zig 側の「こうしてくれ」が C 経由で Swift に届きます。

やりたいのは「キーを押したら macOS のウィンドウを変える」だけなので、**Zig 側にキー操作を1つ足し、要となる処理は Swift 側でウィンドウをいじる** という素直な分担になりました。

## macOS ネイティブの機能をどう使ったか

やっていることは、macOS（AppKit）標準のウィンドウ設定を「いまオーバーレイ中か」を表す `active` 1つで切り替えているだけです。

- **透過** … `isOpaque`（透け具合は `background-opacity`）
- **最前面** … `window.level = .floating`
- **クリックスルー** … `ignoresMouseEvents`（マウス操作を裏に素通り）

```swift:macos/Sources/Features/Terminal/BaseTerminalController.swift
isBackgroundOpaque        = !active                     // 透過
window.level              = active ? .floating : .normal // 最前面
window.ignoresMouseEvents = active                      // クリックスルー
```

`active` を反転するだけで3つが一斉に切り替わります。あとは OFF に戻したとき、`NSApp.activate` と `makeKeyAndOrderFront` で Ghostty を前面に呼び戻し、すぐ入力できるようにフォーカスを返しています。

:::message
普段の挙動を壊さないよう、この機能は設定 `background-control = true` のときだけ動きます。何もしなければ本家とまったく同じです。
:::

## 使い方（設定）

`~/.config/ghostty/config` に3行足すだけ。

```ini
# ① オーバーレイ機能を有効化（無いと何も起きない）
background-control = true

# ② 透け具合。1未満であること
background-opacity = 0.85

# ③ ON / OFF のキー。global: が重要
keybind = global:cmd+backquote=toggle_background_control
```

保存して再起動したら、`cmd + backquote`（`` cmd + ` ``）で ON / OFF します。

### `global:` は必須

オーバーレイ中はクリックが裏に抜けてターミナルを押せないので、トグルキーには必ず **`global:`** を付けます。これで Ghostty が前面になくても、どのアプリからでもワンキーで戻せます（付け忘れると裏の操作中に戻れなくなります）。

### 戻れなくなったときの保険

万一キーが効かなくても、**Dock の Ghostty アイコンをクリックすれば前面に戻ります**。その状態でトグルキーを押せば OFF にできるので、操作不能になることはありません。

### 色付き背景も透かすと見やすい

diff やエディタのように、**明示的に背景色が塗られたセルは標準だと透けません**（Ghostty が透過するのは既定の背景だけなので）。次の1行を足すと、そうした背景も `background-opacity` に従って透けるようになり、オーバーレイ中の見やすさが上がります。

```ini
background-opacity-cells = true
```

例えば Claude Code をフルスクリーン表示で使うと、メッセージや diff の背景がオーバーレイを塞いでしまいますが、これを入れると一緒に透けてくれます。

## 文字が多くても意外と平気

ターミナルの文字で裏の画面が見えなくて不便なのでは、と思うかもしれませんが、使ってみると意外と平気です。

![文字が大量に出ていても裏のブラウザが透けて見え、操作できている様子](/images/ghostty-overlay/文字が多くても大丈夫.png)
*この記事を作成している画面*


## おわりに


Ghostty が **Zig コア＋ Swift のガワ** という構成なので役割分担も明快で、フォークの題材を探している人の参考になれば。

フォークはこちらです。

https://github.com/michi-1221/ghostty-overlay

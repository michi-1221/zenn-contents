---
title: "ターミナルで動くデジタル＆アナログ時計 tty-clock を作ってみた"
emoji: "🕒"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["cli", "terminal", "tui", "tools"]
published: true
---

## はじめに

`tty-clock` は、ターミナル上で動作する設定可能な **デジタル & アナログ時計** です。
テーマ対応（グラデーションテーマを含む）、キーボード操作、目にやさしい表示が特徴で、
macOS / Linux / Windows で動作します。

この記事では、実装の中身には触れず、**インストール方法**と**使い方**を中心に紹介します。

![tty-clock のデジタル時計表示](/images/terminal-clock/overview.png)
*ターミナルに表示されるデジタル時計（デフォルトの tokyo-night テーマ）*

## インストール

### npm で実行する（おすすめ）

npm ですぐに実行できます。

```bash
npx tty-clock@latest        # 今すぐ実行する
npm install -g tty-clock    # または `tty-clock` コマンドをインストールする
```

macOS / Linux / Windows（x64 / arm64）向けにビルド済みです。

### Go ツールチェーンで入れる

Go を使いたい場合は、ソースからインストールできます。

```bash
go install github.com/michi-1221/tty-clock@latest   # `tty-clock` をインストール
go run github.com/michi-1221/tty-clock@latest        # または一度だけ実行（インストールなし）
```

バージョンの確認はこちら。

```bash
tty-clock --version
```


## 起動と基本的な使い方

インストール後は、次のコマンドで起動します。

```bash
tty-clock
```

起動するとターミナルいっぱいに時計が表示されます。あとはキー操作で表示を切り替えられます。

### キー操作

| キー | 動作 |
| --- | ------ |
| `s` | 秒の表示 / 非表示 |
| `t` | テーマを順番に切り替える |
| `m` | デジタル ⇄ アナログ を切り替える |
| `?` | ヘルプ行の表示 / 非表示 |
| `r` | 設定ファイルを再読み込みする |
| `e` | 設定ファイルをエディタで開く |
| `q` · `ctrl+c` · `esc` | 終了する |

切り替えた内容は **その場で設定ファイルに保存される**ので、次回起動時も同じ状態で始まります。

![tty-clock のアナログ時計表示](/images/terminal-clock/analog.png)
*`m` キーで切り替わるアナログ時計*

## 設定

`tty-clock` は設定を **`~/.tty-clock/config.json`** に保存します。
初回起動時にデフォルト値で自動作成されるので、常に編集対象のファイルが存在します。

- `e` キーでエディタ（`$VISUAL` / `$EDITOR`、なければ OS のデフォルトアプリ）で開けます。保存して閉じると自動でリロードされます。
- 手動で編集して `r` キーを押すと、その場で再読み込みされます。

別のファイルを使いたい場合は、起動時に指定できます。

```bash
tty-clock --config /path/to/config.json
```

### 主な設定項目

| 項目 | デフォルト | 内容 |
| --- | --- | --- |
| `mode` | `"digital"` | `"digital"` または `"analog"`（`m` でも切り替え可） |
| `theme` | `"tokyo-night"` | カラーテーマ |
| `granularity` | `"seconds"` | 更新間隔：`"seconds"` または `"minutes"` |
| `showHelp` | `true` | 下部のヘルプ行を表示する |
| `format.hour24` | `true` | 24時間表示（`false` で12時間表示） |
| `format.showSeconds` | `true` | 秒を表示する |
| `format.showDate` | `true` | 時計の下に日付を表示する |
| `format.blinkColon` | `false` | コロン `:` を1秒ごとに点滅させる |

### 設定ファイルの例

```json
{
  "mode": "digital",
  "theme": "tokyo-night",
  "customTheme": { "accent": "#ff79c6" },
  "granularity": "seconds",
  "showHelp": true,
  "format": {
    "hour24": false,
    "showSeconds": true,
    "showDate": true,
    "blinkColon": true,
    "font": "block"
  }
}
```


## テーマ

`tokyo-night` がデフォルトです。`t` キーで全10種類（グラデーションテーマ含む）を順番に切り替えられます。

代表的なテーマ：

- `tokyo-night`（デフォルト）
- `dracula`
- `nord`
- `gruvbox`
- `catppuccin-mocha`
- `solarized-dark`
- `monochrome`

さらに、巨大な数字や文字盤をカラーランプで塗り分ける **グラデーションテーマ** もあります。

- `sunset`（gold → orange → red → purple）
- `aurora`（green → cyan → violet）
- `rainbow`（red → orange → yellow → green → blue → purple）

![tty-clock のグラデーションテーマ表示](/images/terminal-clock/gradient.png)
*数字や文字盤をカラーランプで塗り分けるグラデーションテーマ*

## 知っておくと便利なこと

- **色** — `NO_COLOR` に対応し、色非対応のターミナルでは自動で色を落とします。非UTF-8環境ではシンプルなASCIIフォントになります。
- **アナログモード**（`m`） — 丸いブライユ文字の文字盤です。ウィンドウが小さすぎる場合は自動でデジタル表示に切り替わり、余裕ができると戻ります。
- **12時間表示** — ゼロ埋め（例：`03:04:05`）で、AM/PM ラベルは付きません。
- **タイムゾーン** — システムのローカルタイムゾーンを使用します。`TZ=…` で上書きできます。


## おわりに

`tty-clock` は、ターミナルにちょっとした彩りと実用性を加えてくれるツールです。
`npx tty-clock@latest` ですぐ試せるので、気になった方はぜひ動かしてみてください。

ソースコードは GitHub で公開しています（MIT ライセンス）。

https://github.com/michi-1221/tty-clock

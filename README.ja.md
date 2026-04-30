# pi-obscura-skill

[简体中文](./README.zh-CN.md) | [English](./README.en.md) | [日本語](./README.ja.md)

[Obscura](https://github.com/h4ckf0r0day/obscura) を Pi で使うための軽量ヘッドレスブラウザ skill/package です。フル Chrome を起動せずに使えます。

Pi での skill 名: `obscura-cdp`

## 製品ポジション

- `pi-obscura-skill`: 軽量・低メモリ・デフォルト第一候補
- `pi-browser-hybrid-skill`: 先に互換性を判定し、危ないサイトは Chrome に切り替える

## このパッケージの目的

`chrome-cdp-skill` の Obscura 版ですが、Chrome 前提をそのまま移植したのではなく、Obscura の実際の挙動に合わせて作り直しています。

主な特徴:

- headless Chrome より低メモリ
- 必要時にローカル Obscura daemon を自動起動
- per-tab keepalive daemon が不要
- Markdown スナップショット、HTML 参照、JS eval、セレクタ click、フォーム入力、遷移、raw CDP を重視
- 現在の Obscura では `Page.captureScreenshot` が未実装なので、スクリーンショット前提の設計を避ける

## インストール

### GitHub に push する前のローカルインストール

```bash
pi install /Users/daidai/ai/pi-obscura-skill
```

### GitHub 公開後のインストール

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

### グローバルではなく、現在のプロジェクトにだけインストール

```bash
pi install -l git:github.com/daidai118/pi-obscura-skill
```

## 更新

このパッケージだけ更新:

```bash
pi update git:github.com/daidai118/pi-obscura-skill
```

固定されていない Pi パッケージをまとめて更新:

```bash
pi update --extensions
```

今後の更新を楽にしたいなら、インストール時に git ref を固定しないでください。

更新しやすい例:

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

固定されるので `pi update` では更新されない例:

```bash
pi install git:github.com/daidai118/pi-obscura-skill@v0.1.2
```

## プロジェクト文書

- [Changelog](./CHANGELOG.md)
- [Contributing guide](./CONTRIBUTING.md)
- [Roadmap](./ROADMAP.md)
- [Release checklist](./RELEASE_CHECKLIST.md)

## 要件

- Node.js 22+
- 自動ダウンロード可能な Obscura リリースバイナリ、または `OBSCURA_BIN` で指定したローカル Obscura

`PATH` に `obscura` があればそれを使います。なければ検証済みバージョンを自動ダウンロードします。

## サイト互換性セルフチェック

新しいサイトを自動化する前に、まず次を実行します:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs check --json https://example.com
```

状態の意味:

- `compatible` → Obscura をそのまま使う
- `risky` → Chrome fallback のほうが安全
- `incompatible` → hybrid / Chrome を使う

このチェックを追加した理由は、ローカル検証で Obscura が問題なく処理できるサイトと、部分的に壊れるサイトがはっきり分かれたためです。たとえば `https://100t.xiaomimimo.com/` では、スタイル未適用、レイアウト崩壊、インタラクティブな申請フロー未表示が確認されました。

## コマンド

すべてのコマンドは以下のスクリプトを使います:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs
```

### ライフサイクル

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs start
skills/obscura-cdp/scripts/obscura-cdp.mjs status
skills/obscura-cdp/scripts/obscura-cdp.mjs stop
```

### ページ管理

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs list
skills/obscura-cdp/scripts/obscura-cdp.mjs open https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs close <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs nav <target> https://example.com
```

### 参照

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs md   <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs snap <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs html <target> [selector]
skills/obscura-cdp/scripts/obscura-cdp.mjs eval <target> "document.title"
skills/obscura-cdp/scripts/obscura-cdp.mjs net  <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs evalraw <target> "DOM.getDocument" '{}'
```

`md` / `snap` は Obscura の `LP.getMarkdown` を使った Markdown スナップショットです。

### 操作

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs click <target> "button.submit"
skills/obscura-cdp/scripts/obscura-cdp.mjs fill  <target> "input[name=q]" "pi agent"
skills/obscura-cdp/scripts/obscura-cdp.mjs type  <target> "hello world"
skills/obscura-cdp/scripts/obscura-cdp.mjs loadall <target> ".load-more" [ms]
```

安定したセレクタがある場合は `type` より `fill` を推奨します。

## 環境変数

```bash
OBSCURA_BIN=/path/to/obscura
OBSCURA_PORT=9223
OBSCURA_STEALTH=1
OBSCURA_WORKERS=4
OBSCURA_PROXY=http://127.0.0.1:8080
OBSCURA_AUTO_INSTALL=0
OBSCURA_VERSION=v0.1.1
OBSCURA_CHECK_SETTLE_MS=1500
```

## 設計メモ

### なぜデフォルトポートは 9223？

Obscura 自体のデフォルトは `9222` ですが、`9222` は Chrome の remote debugging でもよく使われます。両方を併用しやすいように、このパッケージでは `9223` を使います。

### なぜスクリーンショット機能がない？

現行 Obscura では `Page.captureScreenshot` が未実装だからです。現時点では Markdown と HTML ベースの確認のほうが安定しています。

### なぜ per-tab daemon が不要？

Chrome 版ではデバッグ許可ダイアログ回避や tab セッション保持のため必要でした。Obscura は最初からローカル headless daemon なので、各コマンドで browser websocket に再接続し、target に再 attach するだけで十分です。

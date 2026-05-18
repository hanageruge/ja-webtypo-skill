# ja-typo-skill

日本語Webタイポグラフィを最適化するための、AIアシスタント向けスキル / プロンプト集です。

LP・コーポレートサイト・SaaS管理画面・ECサイト・会員サイト・採用サイト・メディア / ブログ・ダッシュボードなど、**日本語が使われるあらゆるWeb制作**で、見出し・本文・ボタン・カード・フォームの**改行と可読性**を改善します。

---

## このスキルが解決すること

- ヒーローコピーが「新しい体験を、/お届けします」のように変な位置で折り返される
- ボタンの日本語が2行に割れる
- カードタイトルが半端な場所で改行される
- `、` や `。` が行頭に来てしまう
- スマホで価格表やフォームがはみ出す
- 字間・行間が日本語に最適化されていない
- 日本語フォントのbold合成で文字が潰れる
- ブラウザ互換を無視した最新CSSで一部環境が崩れる

---

## 特徴

- **日本語タイポの基本**（フォントスタック・字間・行間・約物詰め）を網羅
- **CJK 改行制御**（`line-break: strict` / `word-break: auto-phrase` / `text-wrap: balance`）
- **ブラウザ互換性表**つき（auto-phrase は Chrome 119+ など、フォールバック必須項目を明示）
- **レスポンシブ検証**: 360 / 390 / 768 / 1280px の最小確認セット + 中間幅のドラッグ検証
- **Tailwind ユーティリティパターン**（`text-ja-heading` / `text-ja-button` など）
- **手動改行保護**（重要フレーズの nowrap span パターン）
- **修正後レポートのフォーマット**を規定

---

## ファイル構成

| ファイル | 用途 |
|---|---|
| `SKILL.md` | Claude Code 用のスキル本体（frontmatter 付き） |
| `prompt.md` | Cursor / ChatGPT / Codex / Copilot 等の他ツール用プロンプト |
| `README.md` | このファイル |
| `LICENSE` | MIT |

---

## インストール

### Claude Code

#### 方法 A: ユーザーグローバルに symlink（推奨）

```bash
git clone https://github.com/hanageruge/ja-typo-skill.git ~/ja-typo-skill
mkdir -p ~/.claude/skills
ln -s ~/ja-typo-skill ~/.claude/skills/ja-typo-skill
```

Claude Code を再起動すると、スキル一覧に `ja-typo-skill` が出ます。

#### 方法 B: プロジェクトローカルに配置

```bash
cd your-project
mkdir -p .claude/skills
git clone https://github.com/hanageruge/ja-typo-skill.git .claude/skills/ja-typo-skill
```

#### 方法 C: SKILL.md を単体で配置

```bash
mkdir -p ~/.claude/skills/ja-typo-skill
curl -o ~/.claude/skills/ja-typo-skill/SKILL.md \
  https://raw.githubusercontent.com/hanageruge/ja-typo-skill/main/SKILL.md
```

---

### Cursor

`.cursorrules` または `.cursor/rules/japanese-typography.mdc` に `prompt.md` の内容を貼り付け:

```bash
curl -o .cursorrules \
  https://raw.githubusercontent.com/hanageruge/ja-typo-skill/main/prompt.md
```

---

### GitHub Copilot

`.github/copilot-instructions.md` に追記:

```bash
mkdir -p .github
curl -o .github/copilot-instructions.md \
  https://raw.githubusercontent.com/hanageruge/ja-typo-skill/main/prompt.md
```

---

### Codex CLI

リポジトリのルートに `AGENTS.md` を作成、または既存の `AGENTS.md` に `prompt.md` の内容を追記してください。

---

### ChatGPT (Custom GPT / Projects)

1. `prompt.md` の中身をコピー
2. Custom GPT の **Instructions** に貼り付け
3. もしくは Projects の **System Prompt** に貼り付け

---

### Cline / Roo Code

ワークスペースルートに `.clinerules` を作成:

```bash
curl -o .clinerules \
  https://raw.githubusercontent.com/hanageruge/ja-typo-skill/main/prompt.md
```

---

### Gemini (Gems)

Gem の **Instructions** に `prompt.md` の内容を貼り付け。

---

## 使い方の例

スキルをインストール後、AIに次のように依頼します:

- 「このLPの日本語タイポを最適化して」
- 「ヒーロー見出しがスマホで変な位置で改行されるので直して」
- 「ボタンの折り返しを防ぐCSSを追加して」
- 「Tailwindで日本語ユーティリティを整備して」
- 「`word-break: break-all` を使ってる箇所を全部探して直して」
- 「新規Next.jsプロジェクトに日本語タイポの基本CSSを仕込んで」
- 「`<br>` でレイアウト調整してる箇所をCSSに置き換えて」

---

## 想定ユースケース

| シーン | 対応する利用例 |
|---|---|
| LP / セールスページ | ヒーローコピー、CTA、価格表 |
| コーポレートサイト | ナビ、部署名、ニュース一覧 |
| メディア / ブログ | 本文、引用、目次、リード文 |
| SaaS / 管理画面 | サイドバー、データテーブル、フォーム |
| EC サイト | 商品カード、レビュー、絞り込みフィルタ |
| 会員サイト | マイページメニュー、お知らせ、申込フォーム |
| イベント / セミナー LP | 概要、登壇者、申込ボタン |
| 採用サイト | 職種カード、社員インタビュー、選考フロー |
| ドキュメントサイト | サイドバー、APIリファレンス |

---

## 対象外の用途

このスキルは「日常的なWeb制作の日本語タイポ」に集中しています。次の用途には対応していません:

- フォント見本ページ（specimen）の最適化
- 縦書きレイアウト（`writing-mode: vertical-rl`）
- 印刷用 CSS（@media print の日本語禁則）
- ルビ（`<ruby>`）の細かい挙動
- 特定業種向けのレイアウトチェックリスト

---

## ライセンス

MIT License — 詳細は [LICENSE](./LICENSE) を参照。

商用利用・改変・再配布すべて可能です。

---

## 作者

[花上雄介 / hanaue.yusuke](https://github.com/hanageruge)

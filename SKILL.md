---
name: ja-webtypo-skill
description: Japanese web typography for HTML/CSS, Tailwind, React, Next.js, Vue, Svelte, Astro. Fixes awkward Japanese line breaks; improves readability of headings, body, buttons, cards, nav, forms. Applies proper letter-spacing, line-height, font stack, CJK punctuation handling, and responsive verification. Triggers when Japanese text wraps oddly, headings look cramped, particles strand at line ends, or a site needs production-grade Japanese typography defaults.
---

# Japanese Web Typography

## Goal

Make Japanese web pages readable before decorative. Apply stable CSS defaults first — font stack, letter-spacing, line-height, CJK line-break rules — then protect important phrases manually only when automatic wrapping still fails.

## One-Shot Instruction

When the user wants a copy-paste prompt for another tool, provide this:

```txt
日本語Webタイポグラフィの最適化を行ってください。

目的は、日本語のLP・Webページ上で、見出し・本文・リード文・ボタン・カード・ナビが読みやすく、改行が不自然にならないようにすることです。

以下を実施してください。

1. <html lang="ja"> が設定されているか確認
2. グローバルCSSに、日本語フォントスタック・letter-spacing・line-height・font-feature-settings・改行制御の基本指定を追加
3. 見出し・本文・リード文・カードタイトル・ボタン・ナビ・短いラベルに、要素ごとの改行制御を適用
4. word-break: break-all を原則削除または置換
5. 不要な <br> や &nbsp; による手動改行を見直し
6. スマホ幅 (360 / 390px) で、見出し・ボタン・カードタイトルが変な位置で改行されないように調整
7. カード・選択肢・フォーム行の中で、テキストが枠内に収まっているか確認
8. 行数が増えすぎる場合は、先にカラム幅・グリッド比率・コンテナ幅・左右余白を見直す
9. テーブルがある場合は、`table-layout: fixed` + `<colgroup>` + 各セルの `overflow-wrap: normal` を適用し、ヘッダーには `white-space: nowrap` を指定。ボディ継承の `overflow-wrap: anywhere` によるセル内の1文字孤立を確実に止める
10. CSSだけで解決できない長すぎる文言は短縮案も提示
11. 修正後は、変更したCSS・適用箇所・削除した危険な指定・残っている確認ポイントを報告
```

## Baseline CSS

Use `<html lang="ja">` in markup (not CSS).

```css
body {
  /* Japanese font stack */
  font-family:
    "Hiragino Sans", "Hiragino Kaku Gothic ProN",
    "Yu Gothic", "YuGothic", "Meiryo",
    "Noto Sans JP", system-ui, sans-serif;

  /* CJK-specific spacing */
  font-feature-settings: "palt" 1;   /* Proportional kerning for punctuation */
  letter-spacing: 0.02em;            /* Japanese standard */
  line-height: 1.75;                 /* Japanese standard (English is 1.5) */

  /* Mobile sizing */
  -webkit-text-size-adjust: 100%;    /* Prevent iOS auto-zoom */

  /* Line breaking */
  overflow-wrap: anywhere;
  word-break: normal;                /* Fallback */
  word-break: auto-phrase;           /* Chrome 119+ progressive enhancement */
  line-break: strict;
}

h1, h2, h3, h4 {
  overflow-wrap: normal;
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: balance;
  letter-spacing: 0.01em;            /* Tighter for headings */
  line-height: 1.4;
}

.lead, .nav-label {
  overflow-wrap: normal;
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: pretty;
}

.card-title {
  overflow-wrap: normal;
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: pretty;
}

.button {
  white-space: nowrap;
  word-break: keep-all;
  line-break: strict;
}

.choice, .card, [data-card] {
  overflow-wrap: anywhere;
  word-break: normal;
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: pretty;                 /* orphan guard where auto-phrase is unsupported */
}
```

## Decision Rules

1. **Use `line-break: strict`** for Japanese content. Punctuation, brackets, and small kana follow stricter CJK rules.
2. **Use `word-break: auto-phrase` with `word-break: normal` as fallback.** `auto-phrase` is a **Blink-engine feature** — desktop and Android Chrome / Edge 119+ only. It does **not** run on any iOS browser (iOS Chrome, Edge, and Firefox are all WebKit, not Blink) or on desktop Firefox (Gecko); those fall back to `normal`. Judge support by **rendering engine, not browser brand** — an iPhone "Chrome" is WebKit and will still break mid-word.
3. **Use `text-wrap: balance` only on hero/section headings.** Avoid on repeated card titles, FAQ titles, or ordinary lead paragraphs — use `text-wrap: pretty` or normal wrapping there. Repeated cards with `balance` leave odd blank space on the right. **Put `text-wrap: pretty` on card body / description text too, not just card titles.** `word-break: auto-phrase` is Chromium-only, so without `pretty` a card description orphans its trailing character (e.g. "す。" alone on the last line) on iOS Safari and Firefox. `text-wrap: pretty` (Safari 17.5+, Firefox 121+) restores orphan protection there.
4. **Keep `overflow-wrap: anywhere` on body text** to prevent URL/long-token overflow. Override with `normal` on headings.
5. **Use `white-space: nowrap` and `word-break: keep-all` only for compact CTA buttons.** Card-like buttons, option cards, and quiz choices must be allowed to wrap inside their border. Never apply globally to `button`.
6. **Set `letter-spacing: 0.02em` and `line-height: 1.75` for Japanese body.** Tighter values (0.01em / 1.4) for headings. WCAG requires line-height ≥ 1.5; Japanese standard is 1.7+.
7. **Set explicit `font-weight` for Japanese display fonts.** Browser synthetic bold (when no real bold weight file exists) crushes CJK glyphs. Headings get `font-weight: 700` by default — for single-weight display fonts, override to `400` or the actual supported weight.
8. **Check card interiors:** title, body, controls must fit within the visible border with comfortable padding. Watch for `margin-top: auto` accidentally bottom-aligning the title.
9. **When a heading gains extra lines,** first widen the text column, change grid ratios, reduce column gaps, or narrow the adjacent media — before inserting `<br>` or shrinking the type.
10. **Never use `word-break: break-all`** for normal Japanese body or headings — it splits inside words. Reserve for technical long-token containers only. Remove manual `<br>` / `&nbsp;` hacks that fight the layout.
11. **Override `overflow-wrap: anywhere` inside table cells.** Body-level `anywhere` inherits into `td` / `th` and produces single-character orphans like "コー / ド". Pair `overflow-wrap: normal` on cells with `table-layout: fixed` + `<colgroup>` widths. See the "Table Cells" section.
12. **Decide editorial strictness before writing CSS.** JLREQ defines three line-break strictness levels: newspaper (loose), magazine (mid), and general book (strict). The choice belongs to the editorial policy of the surface, not to the CSS. Narrow news columns may break loose; brand sites, dashboards, and long-form pages should stay strict. The CSS `line-break: loose / normal / strict` maps to this — make the policy call first, then write the values.
13. **Treat `<wbr>` as the inverse of `nowrap`.** `white-space: nowrap` says "never break here." `<wbr>` says "you may break here if needed." Use them together: `nowrap` protects quoted phrases and product names; `<wbr>` is a safety valve for long compound katakana, long URLs, and brand-name + suffix combinations. Unlike `<br>`, `<wbr>` is invisible until the line actually needs a break.
14. **Wrap inline English in `<span lang="en">` and let `hyphens: auto` handle it.** A long English word in Japanese body text (e.g., "extraordinary") can overflow narrow cards. Mark the English span with `lang="en"`, apply `:lang(en) { hyphens: auto }` globally, and optionally insert `&shy;` at preferred break points. This avoids `word-break: break-all` while still keeping the layout intact.
15. **Audit outer CSS selectors when extending a card template.** Pre-existing broad descendant selectors (`.card span`, `.hero p`) will catch new inner elements you add for line-break control. Narrow them to direct child (`.card > span`) or explicit class (`.card .eyebrow`) BEFORE adding nested spans for line control. This prevents accidental color, font-weight, or letter-spacing leakage to your new spans. The bug usually appears as "title turned red" or "weight got bold" right after adding `titleLines`.
16. **Never write literal HTML or Markdown markup in user-facing body data.** Strings like `"<br>"`, `"<ruby>"`, `` "`code`" `` are escaped by JSX/Astro interpolation and render as visible `<br>`, `` `code` `` characters in the page. In Japanese body at small sizes, the angle brackets and backticks appear as tiny marks above CJK glyphs and read as visual noise (resembling 圏点 or 濁点 marks). Either replace with plain Japanese ("改行タグ", "ふりがな"), or use semantic `<code>` markup styled with monospace + background.

## Browser Compatibility

| Property | Coverage | Notes |
|---|---|---|
| `line-break: strict` | All modern browsers | Safe |
| `word-break: auto-phrase` | **Blink only** — desktop/Android Chrome & Edge 119+ | NOT iOS (every iOS browser is WebKit) and NOT Firefox (Gecko). Judge by engine, not brand. Always write `word-break: normal;` fallback above |
| `text-wrap: balance` | Chrome 114+, Safari 17.5+, Firefox 121+ | Graceful degradation |
| `text-wrap: pretty` | Chrome 117+, Safari 17.5+ | Falls back to normal wrapping |
| `text-autospace` | Chrome 132+, Firefox 132+, Safari 18.4+ (Baseline 2025) | CJK / non-CJK auto-kerning. Wrap in `@supports` |
| `text-spacing-trim: trim-start` | Chrome 123+ only | Future-proof CJK punctuation trimming |
| `hanging-punctuation` | Safari only | Use with `first allow-end` for blockquotes |
| `font-feature-settings: "palt"` | All modern browsers | CJK proportional kerning |

Fallback declaration pattern:

```css
word-break: normal;        /* fallback */
word-break: auto-phrase;   /* progressive enhancement */
```

Optional progressive enhancements:

```css
@supports (text-spacing-trim: trim-start) {
  body { text-spacing-trim: trim-start; }
}

@supports (hanging-punctuation: first) {
  blockquote { hanging-punctuation: first allow-end; }
}
```

## Responsive Verification

Minimum check set: **360 / 390 / 768 / 1280** px. Add 1024 and 1440 for content-heavy sites.

Add a breakpoint only when the content actually fails — not because a named device size exists. Drag the viewport between checkpoints to catch mid-range failures (around 540, 640, 834, 1100px).

Acceptance criteria:
- No horizontal scrolling 360–1440px
- Body ≥ 15px on mobile, ≥ 16px on desktop
- Line-height ≥ 1.6 for body
- Tappable controls ≥ 44px tall
- Mobile side padding 16–24px

Typography starting points:

| Element | Mobile | Desktop |
|---|---:|---:|
| Body | 15–16px | 16–18px |
| Section heading | 24–32px | 36–48px |
| Hero heading | 32–40px | 48–64px |
| Button | 15–16px | 16–18px |

For hero headings, prefer `clamp()` but verify at 360px and 390px:

```css
.hero-title { font-size: clamp(28px, 6vw, 56px); }
```

Content width starting points:

| Surface | Max-width |
|---|---:|
| Prose | 720–880px |
| LP section inner | 1080–1200px |
| Card grid | 1120–1280px |
| Dashboard | 1280–1440px |

## Tailwind Pattern

Add utilities to a global layer instead of repeating arbitrary classes:

```css
@layer utilities {
  .text-ja-base {
    overflow-wrap: anywhere;
    word-break: normal;
    word-break: auto-phrase;
    line-break: strict;
    letter-spacing: 0.02em;
    line-height: 1.75;
  }
  .text-ja-heading {
    overflow-wrap: normal;
    word-break: auto-phrase;
    line-break: strict;
    text-wrap: balance;
    letter-spacing: 0.01em;
    line-height: 1.4;
  }
  .text-ja-lead,
  .text-ja-card-title {
    overflow-wrap: normal;
    word-break: auto-phrase;
    line-break: strict;
    text-wrap: pretty;
  }
  .text-ja-button,
  .text-ja-label {
    white-space: nowrap;
    word-break: keep-all;
    line-break: strict;
  }
  .text-ja-choice,
  .text-ja-card-body {
    overflow-wrap: anywhere;
    word-break: normal;
    word-break: auto-phrase;
    line-break: strict;
    text-wrap: pretty;
  }
}
```

Apply by semantic role:

- `text-ja-base` — page/body wrappers, prose containers
- `text-ja-heading` — h1–h4, hero copy
- `text-ja-lead` — lead paragraphs, section intros
- `text-ja-card-title` — card titles, FAQ summaries, feature titles
- `text-ja-button` — compact CTA buttons only
- `text-ja-label` — nav labels, tabs, badges, chips
- `text-ja-choice` — selectable cards, option tiles, quiz answers
- `text-ja-card-body` — card descriptions, helper text, form explanations

Do not apply `text-ja-button` to large option cards or cards with descriptions — they must wrap.

For tables, add table-specific utilities:

```css
@layer utilities {
  .text-ja-table-cell {
    overflow-wrap: normal;        /* breaks the inherited "anywhere" */
    word-break: auto-phrase;
    line-break: strict;
    text-wrap: pretty;
  }
  .text-ja-table-head {
    white-space: nowrap;
    word-break: keep-all;
    line-break: strict;
  }
}
```

Apply by semantic role:

- `text-ja-table-cell` — every `td` in a Japanese table
- `text-ja-table-head` — every `th` (also pair with explicit column widths)

## Table Cells

Tables are a frequent failure point because `overflow-wrap: anywhere` on `body` silently inherits into `td` / `th`. Cells then break mid-character, leaving single-character orphans on the last line ("コー / ド", "般" alone, etc.). Default `table-layout: auto` also lets a long-text column squeeze the short columns flat, so headers like "アプリ同梱" break into "アプリ同 / 梱".

The fix is a four-part contract — none of them alone is enough:

```css
.table-ja-wrap {
  overflow-x: auto;             /* horizontal scroll below min-width */
  border: 1px solid var(--rule);
}

.table-ja {
  width: 100%;
  min-width: 880px;             /* prevents cramping on tablet/mobile */
  table-layout: fixed;          /* respect <colgroup> widths */
  border-collapse: collapse;
}

.table-ja th,
.table-ja td {
  vertical-align: top;
  padding: 12px 14px;
  overflow-wrap: normal;        /* breaks body's inherited "anywhere" */
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: pretty;
}

/* Opt-in nowrap — apply ONLY when the header is confidently short
   (≤8 CJK chars or ≤15 Latin chars) AND fits the assigned column width.
   Blanket-nowrapping every th forces overflow when descriptive headers
   exceed the column. */
.table-ja th.is-compact {
  white-space: nowrap;
  word-break: keep-all;
  line-break: strict;
}
```

Pair with explicit per-column widths via `<colgroup>`:

```html
<div class="table-ja-wrap">
  <table class="table-ja">
    <colgroup>
      <col style="width: 14%">
      <col style="width: 17%">
      <col style="width: 16%">
      <col style="width: 13%">
      <col style="width: 11%">
      <col style="width: 29%">
    </colgroup>
    <thead>…</thead>
    <tbody>…</tbody>
  </table>
</div>
```

### Column width sizing

At ~0.94rem font-size, estimate ~17px per Japanese character and ~9px per Latin character. A 10-character mixed string ("Adobe埋め込みコード") needs roughly 150–170px of *content area* (column width minus ~28px horizontal padding).

| Cell content | Suggested column width |
|---|---:|
| Long description (notes, explanations) | 25–35% |
| Mixed Latin + 8–10 CJK chars (e.g., "Adobe埋め込みコード") | 15–17% |
| 短い日本語ラベル＋記号 (e.g., "○ 条件確認") | 11–14% |
| Single symbol cells (○ / × / △) | 9–11% |

Sum of widths must be 100% when using percentages.

### Diagnosing common cell pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| "コー / ド" — single-character orphan | `overflow-wrap: anywhere` inherited from `body` | Add `overflow-wrap: normal` to `.table-ja td` and `.table-ja th` |
| "アプリ同 / 梱" — short category header breaks mid-word | `th` allowed to wrap with no break-strategy | Add `.is-compact` class with `white-space: nowrap; word-break: keep-all` to that `th` |
| "Zenith (無料)" header overflows the column to the right | `th` blanket-nowrapped while descriptive header exceeds column width | Remove `nowrap` from that `th` — let it wrap at phrase / space boundaries with `auto-phrase` + `pretty` |
| "Adobe埋め込みコー / ド" after wrap fixes | Column narrower than the cell's longest string | Widen that column in `<colgroup>` or shorten cell text |
| Long-description column squeezing short columns flat | Default `table-layout: auto` | Add `table-layout: fixed` + `<colgroup>` |
| Cell text overflows the cell horizontally | `overflow-wrap: normal` + no break opportunity (rare for Japanese; common for long URLs) | Allow `overflow-wrap: anywhere` only in that specific cell, or shorten the content |

When a cell still wraps awkwardly after the four-part contract:

1. Verify the column width matches the content estimate above.
2. Audit each column for its *longest* cell — that's what the width must accommodate.
3. Shorten cell text to a synonym that fits (`"Adobe Fontsで提供される各フォント"` → `"Adobe提供フォント"`).
4. As a last resort, reduce cell `font-size` or raise `min-width` on the table.

Never solve this by adding `<br>` inside cells. The break should come from CSS so the table adapts to viewport changes.

## Manual Phrase Protection

CSS cannot infer phrase boundaries. For hero copy, campaign headlines, and important CTAs, protect key phrases with markup when automatic wrapping still fails. There are two complementary tools:

- **`white-space: nowrap` span** — "never break here"
- **`<wbr>`** — "you may break here if needed"

The core principle behind both: **don't make everything breakable. Increase the number of places where a break IS allowed, and protect the places where a break would hurt comprehension.**

### Protect what must not break

```jsx
<h1>
  シンプルに、
  <span className="nowrap-desktop">「もっと使いやすく」</span>
  進化させる。
</h1>
```

```css
.nowrap-desktop { white-space: nowrap; }
@media (max-width: 833px) {
  .nowrap-desktop { white-space: normal; }
}
```

Use for:
- Quoted phrases (`「〜」で`)
- Product / service names
- Short CTA phrases
- Noun + particle pairs that break poorly (`日本の/技術` を避ける)
- Numbers + units, prices, dates, durations

### Suggest where breaks are allowed

```jsx
<h1 lang="ja">
  デジタルエクスペリエンス<wbr />プラットフォームを導入
</h1>
```

`<wbr>` is invisible until the line actually needs a break — unlike `<br>`, which forces a line break unconditionally. Use it inside:

- Long compound katakana ("デジタルエクスペリエンスプラットフォーム")
- Brand name + Japanese suffix ("文字選びの標本室の<wbr>サービスサイト")
- Long URLs displayed in text
- Long English-Japanese hybrid technical terms

### Inline English

```jsx
<p lang="ja">
  予算は <span lang="en">extra&shy;ordinary</span> なほど膨れ上がった
</p>
```

```css
:lang(en) { hyphens: auto; }
```

Marking inline English with `lang="en"` enables proper hyphenation for that span. `&shy;` (soft hyphen) is an optional explicit break hint that only renders as a hyphen when the line actually breaks there.

Verify protected and suggested-break phrases still fit at the narrowest supported width. If they do not, shorten the copy or reduce font size — do not force overflow.

## Data-Controlled Line Breaks for Card Templates

When repeated cards (grid items, FAQ lists, hero callouts) display titles from data, CSS auto-wrap with `auto-phrase` + `pretty` can still fail at:

- Narrow card widths (5-up, 6-up grids)
- Browsers without `auto-phrase` support — WebKit (all iOS browsers, desktop Safari) and Gecko (Firefox)
- Strings that exceed the per-card character budget

The fallback: store explicit break points in the data, render as block-level child elements.

### Data shape

```ts
type Card = {
  title: string;             // fallback for SSR and when no editorial breaks set
  titleLines?: string[];     // optional explicit line breaks
};
```

### Render

```astro
<strong>
  {
    card.titleLines
      ? card.titleLines.map((line) => <span class="title-line">{line}</span>)
      : card.title
  }
</strong>
```

### Per-line CSS

```css
.title-line {
  display: block;
  overflow-wrap: normal;
  word-break: auto-phrase;
  line-break: strict;
  text-wrap: pretty;
}
```

Do NOT add `word-break: keep-all` to `.title-line`. If a line still exceeds the container, it should gracefully wrap with `auto-phrase` + `pretty` rather than overflow the card border. The principle holds: protect the intended break with the block-level span, but keep a fallback inside each line.

### When to use

- Hero taglines in grid cards (each card has its own short tagline)
- Font specimens with curated short sample text
- Editorial headlines where break position is part of the message
- Marketing CTAs where the line break creates visual rhythm

Not for body paragraphs, long lists, or user-generated content — those should use CSS auto-wrap.

## Card Density Budget

Before choosing a hero font-size for a card grid, calculate the per-line character budget. Skipping this step is why a `1.55rem` title looks fine in design tools and breaks at "い。" alone in production.

### Formula

```
content_width = (container_width − gap × (columns − 1)) / columns − card_padding × 2
budget_chars  = content_width / font_size_px
```

`font_size_px` is `rem × 16`. For Japanese full-width characters, assume 1 char ≈ font-size in pixels. Letter-spacing adds ~2% per pair, usually negligible at this granularity.

### Worked example (5-up grid)

- Container: 1280px (capped main width)
- Gap: 18px, columns: 5
- Card padding: 22px each side
- Content width: (1280 − 72) / 5 − 44 = **197px**

| Font size | Budget chars |
|---|---:|
| 1.55rem (24.8px) | ~7 |
| 1.4rem (22.4px) | ~8 |
| 1.3rem (20.8px) | ~9 |
| 1.15rem (18.4px) | ~10 |

If the longest line in `titleLines` exceeds the budget, choose one:

1. **Reduce font-size with `clamp()`** so it scales at narrower viewports.
   ```css
   .card-title { font-size: clamp(1.2rem, 1.7vw, 1.5rem); }
   ```
2. **Split into more `titleLines` parts.** A 10-char string at 1.3rem (~9 char budget) can split into 4 + 6.
3. **Reduce column count.** 5-up → 4-up gives ~50% more content width per card.
4. **Shorten the copy.** Cheapest fix, but constrains brand voice.

### Don't fight the math

A 1.55rem hero font in a 5-up grid produces ~7-character budget. Any title line longer than 7 characters WILL wrap. On browsers without `auto-phrase`, it will produce single-character orphans. Pick font-size to match the longest line you actually have, then validate at the design's narrowest desktop viewport (typically 1280px or 1024px).

## Coverage Tiers: CSS baseline → BudouX

Phrase-aware Japanese line breaking has two tiers. Apply Tier 1 always; escalate to Tier 2 as a deliberate, user-facing decision.

### Tier 1 — CSS baseline (always)

The baseline CSS above (`word-break: auto-phrase`, `line-break: strict`, `text-wrap`, `palt`). Result by **rendering engine**:

- **Blink** (desktop & Android Chrome / Edge): `auto-phrase` works → fully phrase-aware breaking. Done.
- **WebKit** (every iOS browser, desktop Safari) and **Gecko** (Firefox): `auto-phrase` is ignored. `text-wrap: balance / pretty` still prevents orphaned trailing characters, but lines can still break **mid-word** — "制作物" → "制作/物", "ライセンス" → "ラ/イセンス". Tier 1 does not fix this.

Because every iOS browser is WebKit, the Tier-1 residual affects **all iPhone visitors** — on a Japan-facing site that is roughly half of mobile traffic, not an edge case.

### Tier 2 — BudouX (escalate when the residual matters)

BudouX is a lightweight (~20KB) phrase-segmentation library by Google — the same model behind `auto-phrase`. It inserts `<wbr>` break opportunities at phrase boundaries, so phrase-aware breaking works on **every engine**, WebKit and Gecko included.

Escalating to Tier 2 is a **judgment call — always surface it to the user.** State the cost and the benefit and let them decide; never silently skip it, never silently add the dependency.

- **Cost:** a build-time dependency; the HTML gains `<wbr>` elements; one more layer to account for when debugging a break.
- **Benefit:** correct phrase breaking on iPhone and Firefox, not only Blink.

**Apply BudouX at build time** (SSG) or server-side — it bakes `<wbr>` into the HTML with zero runtime JS and zero runtime cost. Avoid the `<budoux-ja>` runtime web component on static sites; reserve it for text that only exists client-side.

```js
// Build-time (Astro / Vite / Node). Construct the parser once, reuse it.
import { loadDefaultJapaneseParser } from "budoux";
const parser = loadDefaultJapaneseParser();
// HTML-escape each segment, then join with <wbr>. Render the result via the
// framework's HTML injection (set:html / dangerouslySetInnerHTML).
const html = parser.parse(text).map(escapeHtml).join("<wbr>");
```

Output **`<wbr>`** — a real, semantically inert HTML element. Never emit zero-width-space characters (they contaminate copy-paste and search), and never wrap every phrase in a `<span>` (that is what creates accessibility / SEO noise). Never store the `<wbr>` HTML back in your data layer; wrap at the render layer only (see Rule 16).

### Where to apply BudouX

USE it on short, curated **display** copy:
- hero / section headings, card titles, CTA text
- short lead paragraphs, short card / feature descriptions
- any important copy that breaks badly on mobile

Do NOT use it on:
- long-form body / blog article text, Q&A long comments
- form input values, URLs, email addresses, phone numbers, product codes / IDs
- price tables, admin-panel input fields
- user-generated content

**Proper nouns** — clinic names, personal names, product / service / medical names (e.g. "フォームソティックス・メディカル") — are where auto-segmentation most often guesses wrong. Do not trust BudouX (or `auto-phrase`) there: protect them by hand with `white-space: nowrap`, manual `<wbr>`, or `<br>` (see "Manual Phrase Protection"), and keep hand-authored breaks authoritative where they mix with BudouX output.

## Layout Checks

After typography changes, verify:

- Single-character particles (を / が / の / に) not stranded at line ends/starts
- Punctuation (、。) not at line head
- Hero heading not unexpectedly tall
- No horizontal scroll 360–1440px
- Body text ≥ 15px on mobile
- Tap targets ≥ 44px
- Text fits inside card borders without clipping
- Card content not unintentionally bottom-aligned by `margin-top: auto`
- Form rows: label column not consuming too much width, input field not cramped
- Two-column LP sections: text column not too narrow vs. adjacent image
- Button text not awkwardly two-line (unless intentional card-button)
- Card titles not splitting particles, quoted phrases, product names, or numbers + units
- Card descriptions: trailing character not orphaned on the last line — check iOS Safari / Firefox, where `auto-phrase` is unavailable and only `text-wrap: pretty` guards the orphan
- Font weight not synthesized — display fonts show their actual weight, not browser-faked bold
- Table cells: no single-character orphans on the last line, headers not breaking mid-word, longest cell in each column fits the assigned `<colgroup>` width
- Tables narrower than viewport: horizontal scroll appears below `min-width`, layout does not break out of its container
- Card grids: longest planned `titleLines` line fits the calculated content_width / font_size_px budget at the narrowest desktop viewport
- Card title color / weight: matches the design after adding nested `.title-line` spans — verify no descendant selector leakage from older `.card span` / `.hero p` rules
- Body data audit: no literal `<br>`, `<ruby>`, backticks, or other HTML/Markdown tokens in user-facing strings — those render as visible characters and create noise above CJK glyphs

For repeated list cards with 1–2 lines of Japanese, prefer vertical centering:

```css
.list-item {
  display: flex;
  align-items: center;
}
```

## Final Report

After editing, report:

- CSS rules added or changed
- Components / selectors where typography classes or rules were applied
- Dangerous rules removed or replaced (especially `word-break: break-all`)
- Manual `<br>` / `&nbsp;` changes
- Viewports checked, manual drag-check ranges covered
- Any breakpoint added and the content failure that justified it
- Remaining copy that may need shortening
- **Tier-1 residual:** state explicitly that on WebKit (all iOS browsers, Safari) and Gecko (Firefox), `auto-phrase` does not run and mid-word breaks remain — raise the Tier-2 / BudouX decision for the user instead of claiming Japanese breaking is fully "fixed" when only Blink is covered

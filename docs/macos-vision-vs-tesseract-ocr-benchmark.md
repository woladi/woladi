# 📊 macos-vision vs Tesseract — A 50-File OCR-to-Markdown Benchmark

> 🍎 Apple Vision vs 📜 Tesseract on 50 PDFs, both routed through the same local LLM formatter, **both feeding the formatter real layout coordinates**. Result: on this corpus, Tesseract has **lower CER (19.2% vs 26.5%)**, Apple Vision has **higher LLM-judged structural quality (20% pass vs 12%)**, and the engines come out closer than any single small sample would suggest.

---

## 🎯 What We're Measuring (And Why This Run Differs From the Last One)

This is the second version of this benchmark. The first version, posted earlier this week with **10 PDFs and a naïve Tesseract setup**, concluded that Apple Vision dominated 8 / 10 to 0 / 10. That conclusion turned out to be premature.

The original setup gave Apple Vision a structural advantage that wasn't really about OCR — it was about what each engine handed to the downstream formatter:

- **Apple Vision** delivered text **grouped into paragraphs with bounding-box y-coordinates**, which the formatter used to assign headings and reading order.
- **Tesseract**, in v1, delivered a **flat line stream split on blank lines**. The formatter had to guess the structure and frequently guessed wrong.

Tesseract is not a flat-line OCR engine. Its TSV (or hOCR / ALTO) output exposes word-level bounding boxes plus a four-level hierarchy: page → block → paragraph → line → word. **For an apples-to-apples comparison, the Tesseract side has to use this hierarchy.** This benchmark does that: every word's coordinates flow into the same `ParagraphGroup` structure that `VisionScribe` builds for Apple Vision, and the formatter receives byte-identical `<ocr_source>` input shape from both engines. The only thing that varies is which engine produced the underlying text and coordinates.

The dataset was also bumped from 10 to **50 single-page PDFs** to dampen the run-to-run variance of the LLM judge.

---

## ⚙️ The Pipeline (v2)

```
                         ┌──────────────────────────────┐
        📄 PDF ─────────►│  🍎 Apple Vision OCR           │──► blocks + paragraph IDs + (x,y) ──┐
                         │  (macos-vision/VisionScribe)   │                                       │
                         └──────────────────────────────┘                                       │
                                                                                                  ▼
                                                                                ┌─────────────────────────────┐
                                                                                │  🦙 Ollama (mistral-nemo)     │
                                                                                │  ParagraphGroup → buildUser   │──► 📝 Markdown
                                                                                │  same SYSTEM_PROMPT, temp=0   │       │
                                                                                └─────────────────────────────┘       │
                                                                                                  ▲                    │
                         ┌──────────────────────────────┐                                       │                    │
        📄 PDF ─────────►│  📜 Tesseract 5.5.2            │──► TSV: words + bbox + par_num ───────┘                    │
                         │  pdftoppm 300 DPI, --psm 1 tsv │                                                            │
                         └──────────────────────────────┘                                                            │
                                                                                                                       ▼
                                                                                                          🧑‍⚖️ Claude Haiku 4.5 judge
                                                                                                              CER + 1–10 structure score
```

Both branches converge at the same Ollama call with the same system prompt and the same `buildUserContent()` wrapping. The only asymmetry remaining is the underlying OCR — what each engine "saw" on the page and how it segmented the result. That asymmetry is the thing being measured.

---

## 🧪 Methodology

| | |
|---|---|
| **Dataset** | 50 single-page PDFs from [opendataloader-bench](https://github.com/opendataloader-project/opendataloader-bench): `01030000000001.pdf` … `01030000000050.pdf`. Academic / archival pages — body text, headings, footnotes, figure captions, page numbers. |
| **Ground truth** | Hand-curated Markdown shipped with the benchmark, one file per PDF. |
| **Vision OCR** | `macos-vision@1.3.2` (latest on npm) via `VisionScribe.toMarkdown()`. The Apple Vision Framework returns text blocks with bounding boxes and reading-order metadata; macos-vision groups them into `ParagraphGroup` objects (`paragraphId`, `y`, `lines[]`). |
| **Tesseract OCR** | `tesseract 5.5.2` (Leptonica 1.87.0) with `--psm 1 tsv` on PNGs rasterized at 300 DPI by `pdftoppm` from Poppler 26.04.0. The TSV output is parsed into the same `ParagraphGroup` shape: words are grouped by `(block_num, par_num)`, paragraph y-coordinates are computed as `avg(top) / page_height` and normalized to [0, 1]. |
| **Formatter** | Ollama 0.23.3 running `mistral-nemo:latest` (digest `e7e06d107c6c`, 7.1 GB Q4_0 GGUF), temperature 0, top_p 1. Both engines call `chat(ollamaOpts, SYSTEM_PROMPT, buildUserContent(paragraphs))` — same `SYSTEM_PROMPT` and `buildUserContent` imported live from the macos-vision repo. |
| **CER** | Levenshtein(prediction, ground truth) ÷ len(ground truth), clamped to [0, 1]. Lower = better. |
| **LLM judge** | Claude `claude-haiku-4-5-20251001` via `@anthropic-ai/sdk`. Scores 1–10 on text accuracy + structure + completeness. Pass threshold: ≥ 8. Same prompt as `eval/metrics.ts`. Each prediction judged once (see Caveats). |
| **Runtime** | Node 24.14.1, `tsx` for live import of TypeScript modules from the macos-vision source tree, macOS 25.4 (Darwin) on Apple Silicon. |

The Vision-side numbers come from running `VisionScribe.toMarkdown()` over the 50 PDFs in a single sweep, not from the macos-vision repo's previously-cached predictions. Both engines' predictions were generated fresh in this benchmark run, so any Ollama drift between sessions is absorbed identically on both sides.

---

## 🏆 Headline Results

| Metric | 🍎 macos-vision | 📜 Tesseract | Winner |
|---|---:|---:|:---:|
| **Avg CER** (lower better) | 26.5% | **19.2%** | 📜 by 7.3 pt |
| **Median CER** | 19.3% | **13.4%** | 📜 by 5.9 pt |
| **Avg LLM score** (1–10) | **4.90** | 4.74 | 🍎 by 0.16 |
| **Median LLM score** | 4 | 4 | tied |
| **Pass rate** (LLM ≥ 8) | **10 / 50 (20%)** | 6 / 50 (12%) | 🍎 |
| **Head-to-head LLM wins** | **21** | 14 | 🍎 |
| **Head-to-head CER wins** | 13 | **24** | 📜 |
| **Files with CER > 50%** | 13 | **6** | 📜 |
| **Mean latency / file** | **25.3 s** | 29.3 s | 🍎 |
| **Empty outputs** | 0 | 0 | tied |

This is a very different table from the v1 result. Three things stand out.

**Tesseract reads characters more reliably across this corpus.** Lower mean CER, lower median CER, more head-to-head wins on CER, and fewer catastrophic OCR failures (CER > 50%). On 24 of the 50 files Tesseract returned closer-to-ground-truth text than Apple Vision did.

**Apple Vision still has the structural edge — but it's a small one.** Average LLM score is essentially tied (4.90 vs 4.74), but the pass-rate distinction (20% vs 12%) shows that when Apple Vision *does* work, it's more likely to produce a document the judge will accept outright. Tesseract bunches around scores of 3–6; Apple Vision is more bimodal — it nails it or fails it.

**Apple Vision is faster.** Native API access is real. The Apple Vision OCR step is essentially zero-overhead from Node, while Tesseract requires raster + OCR + TSV parse before the same Ollama call.

---

## 📈 Distributions

### LLM score distribution

```
score:  2   3   4   5   6   7   8   9  10
─────────────────────────────────────────
🍎      6  16   8   0   7   3   1   8   1
📜      5  16   7   2  10   4   1   5   0
```

- Both engines cluster around 3 — most files have at least one structural defect that the judge dings them for.
- Apple Vision has 9 files at score ≥ 9 vs Tesseract's 5 — Apple Vision is more likely to produce a near-perfect result.
- Neither engine has any 10s on Tesseract; both have very few 5s.

### CER distribution

```
bucket:   <5%   5–10%  10–20%  20–30%  30–50%  >50%
────────────────────────────────────────────────────
🍎         16     7      3       3       8      13
📜         12     9     16       1       6       6
```

The CER histograms tell the bimodality story:

- **Apple Vision** is a barbell — 16 files at < 5% CER (near-perfect reading) and 13 files at > 50% CER (catastrophic recognition failure). When it works it's pristine; when it fails the whole page is mangled.
- **Tesseract** is centered around 10–20% CER. Fewer perfect runs, but also fewer disasters. It's the more boring, more uniform engine on this dataset.

---

## 🔍 Notable Cases

### Case 1 — Where Apple Vision dominates: prose with a clear title (file 24)

**Ground truth** is straightforward narrative — title "At Home in Exile", then plain body paragraphs about a trip to Wrocław.

**🍎 macos-vision** (LLM 9, CER 2.1%):
```markdown
# At Home in Exile

We received the clear impression from grim customs officials and money-changers at the border that we had entered a part of the world still not entirely recovered from post-War economic depression. […]

Anthea's navigation skills took us promptly to the clean and pleasant Tumski Hotel on the Sand Island near the oldest part of Wroclaw. […]
```

**📜 Tesseract** (LLM 3, CER 4.7%):
```markdown
# At Home in Exile

## We received the clear impression...

- ...from grim customs officials and money-changers at the border…
- Roadside stands sold plaster garden statues, especially gnomes…

## Anthea's navigation skills took us promptly...

- ...to the clean and pleasant Tumski Hotel on the Sand Island…
```

Both engines read the page cleanly — CER is 2.1% for Vision, 4.7% for Tesseract. But the formatter, fed the Tesseract paragraph blocks, decided every paragraph's opening clause was a heading and the rest of the paragraph was a bulleted continuation. The judge's verdict: *"the prediction fundamentally misrepresents the structure by converting a prose narrative into an artificial heading-and-bullet-point format that doesn't exist in the original."* This is a layout-inference failure even with bounding boxes present — Tesseract's paragraph grouping put first lines at one y and continuations at another, and the LLM ran with that signal.

### Case 2 — Where Tesseract dominates: stylized title typography (file 12)

**Ground truth** is a textbook page about Aladdin pantomime prints, with two figure captions and body prose between them.

**🍎 macos-vision** (LLM 2, **CER 76.4%**):
```markdown
# 96 MACDONALD

## Taking Maddin mile Henderful Jamp?

FIGURE 5.1 Mr. Bologna Jun-r as Kalim Azack in Aladdin,
or FIGURE 5.2 Mr. Grimaldi as Kazrac (the Chinese slave) in
The Wonderful Lamp. Aladdin, or The Wonderful Lamp.
```

**📜 Tesseract** (LLM 6, CER 17.6%):
```markdown
# Figure 5.1 Mr. Bologna Jun-r as Kalim Azack in Aladdin, or The Wonderful Lamp.

theatrical prints, which are informed by intercul-turation and illustrate the Orientalized look of the tale's theatrical life: […]

# Figure 5.2 Mr. Grimaldi as Kazrac (the Chinese slave) in Aladdin, or The Wonderful Lamp.
```

This is the failure mode that drives Apple Vision's bimodal CER. The stylized "Aladdin, or The Wonderful Lamp" title — large fancy display type — defeated the Apple Vision recognizer, which read it as **"Taking Maddin mile Henderful Jamp?"**. With a 76% CER on the page, no downstream pipeline can recover. Tesseract's traditional pattern-matching handled the stylized type fine, kept the figure captions intact, and produced a document the judge gave a 6.

A similar pattern shows up on files **08** (Vision CER 56.8%, Tesseract 38.6%), **13** (74.9% vs 37.9%), **37** (56.9% vs 53.6%), and **42** (51.6% vs 15.7%). When Apple Vision misreads the page at the recognition level, Tesseract's more uniform performance becomes a meaningful advantage.

### Case 3 — Where both struggle: a table-of-contents page (file 16)

**Ground truth** is a flat list of chapter titles with page numbers.

Both engines restructured the list as a hierarchy with headings and bullet points; both lost the page numbers. Apple Vision LLM 4, Tesseract LLM 6. The judge: *"page numbers are entirely missing in the prediction"* (Vision) and *"loses page numbers entirely, omits numbered items"* (Tesseract). The Tesseract version was judged slightly more readable, but neither passes.

This is a class of document (TOCs, indexes, tabular content) where the formatter's strong prose bias works against both engines uniformly.

---

## 💡 Why The Gap Shrank

Three things explain the difference between this 50-file fair-play run and the original 10-file unfair-play run.

**Coordinates are necessary, not sufficient.** v1 measured "Vision + coordinates vs Tesseract + no coordinates." That isn't an OCR comparison; it's a comparison of how much spatial information the formatter receives. Giving both engines the same shape of structured input — paragraph IDs and y-hints — closes that gap directly. Tesseract pass rate went up (0 → 12%), Vision pass rate fell (80 → 20%), and a 1.84× LLM-score gap collapsed to 1.03×.

**Vision's recognition has a brittle long tail.** Apple Vision is excellent on body text and clean typography but has visible weaknesses on stylized titles, italics, and unusual layouts (a quarter of the corpus shows CER > 50%, which means the recognizer essentially gave up). Tesseract's pattern matching is less sensitive to display typography and produces a more uniform error profile. CER-the-metric reflects this.

**LLM-judge scores have meaningful variance.** Single-run scores on the same prediction can differ by ±2 points depending on which sentence the judge zeroes in on. A 10-file sample with such variance can swing aggregate pass rate by 30 percentage points. The 50-file aggregates here are much more stable, but the headline numbers should be read as point estimates with a meaningful confidence band, not exact measurements.

The honest read on the data: **on this corpus, with this formatter, with this judge, the engines are within noise on overall quality. Apple Vision is faster, slightly more likely to produce a passing document, and produces more "perfect" outputs when it works. Tesseract is more consistent, reads characters more reliably, and has fewer catastrophic failures.** Either is defensible depending on what you optimize for.

---

## ⚠️ Caveats

- **Judge non-determinism.** The macos-vision `llmJudge()` call doesn't set `temperature: 0` on the Claude API. Empirically, re-judging the same prediction can shift the score by 1–2 points; on a small sample (≤ 10 files) this can move aggregates substantially. Fifty files is enough for stable rankings but not for precise point estimates. A repeat run on the same predictions would produce similar — not identical — aggregates.
- **The formatter is a co-author.** Both engines share `mistral-nemo` as the downstream Markdown formatter. The formatter's behavior under structurally ambiguous input (e.g., "first long block is a heading") affects both branches and is responsible for some of the gap on prose-heavy pages like file 24.
- **Tesseract is single-pass.** No per-document tuning of `--psm`, no language detection, no preprocessing (binarize / deskew). A production deployment of either engine would do more.
- **CER is character-level only.** It does not penalize structural failures, reading-order errors, or layout misalignment. That's why CER and LLM-score disagree so often in this dataset.
- **Sample is academic / archival pages.** The opendataloader-bench corpus skews toward books, journal articles, and reproduced historical material. Engines may rank differently on receipts, screenshots, handwritten text, or modern forms.
- **One judge model.** Claude Haiku 4.5 was chosen because it's the macos-vision eval default. A different judge (GPT-4o, a larger Claude, a fine-tuned model) could yield different absolute scores. Relative rankings between Vision and Tesseract on the same files would likely be stable.
- **v1 vs v2 honesty.** The v1 numbers in this article's earlier version were not wrong on their own terms — they correctly described a particular pipeline configuration. They were misleading as a Vision-vs-Tesseract comparison because the Tesseract side was missing half its capabilities. This v2 is what the comparison looks like when both engines run with their full layout features.

---

## ⏱️ Latency

| | 🍎 Apple Vision | 📜 Tesseract |
|---|---:|---:|
| **Mean per file** | 25.3 s | 29.3 s |
| **Median** | 26.7 s | 30.4 s |
| **p95** | 39.7 s | 45.0 s |
| **Cost driver** | Ollama format (~95%) | Ollama format (~85%) + rasterize/OCR/parse (~15%) |

Apple Vision's native OCR is essentially free from the caller's perspective — the bottleneck is the same Ollama call on both sides. Tesseract adds a real but small overhead from the pdftoppm raster step (~0.7 s) and TSV parsing. On a per-page basis the gap is about four seconds. At scale (10,000 pages), that's ~11 hours of difference — meaningful but not dramatic. Both pipelines are dominated by the LLM formatter, and any optimization effort is best spent there.

---

## 🔁 Reproducibility

The Vision-side numbers can be reproduced with vanilla [`macos-vision`](https://github.com/woladi/macos-vision):

```bash
cd macos-vision
npm install
npm run eval:setup
npm run eval -- --limit 50
npm run eval:report
```

For Tesseract, the sketch is:

```bash
brew install tesseract poppler          # tesseract 5.x + pdftoppm
ollama pull mistral-nemo                # same formatter macos-vision uses
export ANTHROPIC_API_KEY=sk-ant-...     # claude-haiku-4-5 as judge

for stem in 01030000000001 … 01030000000050; do
  pdftoppm -r 300 -png "bench/pdfs/$stem.pdf" "images/$stem"
  tesseract "images/$stem-1.png" "tsv/$stem" -l eng --psm 1 tsv
  # parse TSV → ParagraphGroup → buildUserContent → ollama chat
done
```

The TSV-to-`ParagraphGroup` step is the critical part: words are grouped by `(block_num, par_num)`, lines by `line_num`, and y-coordinates normalized by `page_height` to match the `[0, 1]` shape that `buildUserContent` expects. After that, the call signature into Ollama is identical to what `VisionScribe.toMarkdown()` does internally.

Full benchmark harness, including the eval set, ground truth, CER, LLM-judge, and report runner, is shipped with the `eval/` directory in [woladi/macos-vision](https://github.com/woladi/macos-vision/tree/main/eval).

---

## 📦 Appendix — Selected Per-File Results

### Top 5 Apple Vision wins by LLM delta

| File | 🍎 LLM / CER | 📜 LLM / CER | ΔLLM |
|---|---|---|---:|
| 01030000000024 | 9 / 2.1% | 3 / 4.7% | **+6** |
| 01030000000010 | 6 / 35.9% | 3 / 28.1% | +3 |
| 01030000000020 | 9 / 1.6% | 6 / 2.4% | +3 |
| 01030000000025 | 6 / 2.5% | 3 / 1.8% | +3 |
| 01030000000023 | 9 / 1.4% | 7 / 1.2% | +2 |

### Top 5 Tesseract wins by LLM delta

| File | 🍎 LLM / CER | 📜 LLM / CER | ΔLLM |
|---|---|---|---:|
| 01030000000008 | 2 / 56.8% | 6 / 38.6% | **−4** |
| 01030000000012 | 2 / 76.4% | 6 / 17.6% | **−4** |
| 01030000000013 | 3 / 74.9% | 7 / 37.9% | **−4** |
| 01030000000037 | 3 / 56.9% | 6 / 53.6% | −3 |
| 01030000000042 | 3 / 51.6% | 6 / 15.7% | −3 |

### Aggregates

```
                    🍎 macos-vision    📜 Tesseract
─────────────────────────────────────────────────
mean CER                26.5%               19.2%
median CER              19.3%               13.4%
mean LLM score          4.90                4.74
median LLM score        4                   4
pass rate (≥ 8)         10/50 (20%)         6/50 (12%)
head-to-head CER        13 wins             24 wins  (13 ties)
head-to-head LLM        21 wins             14 wins  (15 ties)
catastrophic (CER>50%)  13 files            6 files
mean latency            25.3 s              29.3 s
```

Full per-file JSON (50 rows each) is available in the scratch directory used to produce this article:

```
/private/tmp/ocr-bench/reports/report-vision.json
/private/tmp/ocr-bench/reports/report-tesseract.json
```

These contain `{file, cer, llmScore, llmReason, passed}` per file, in the same shape as macos-vision's `eval/reports/`.

---

✅ Setup notes for anyone reproducing: the Tesseract branch in this benchmark imports `chat`, `ping`, `SYSTEM_PROMPT`, `buildUserContent`, `computeCER`, and `llmJudge` directly from the macos-vision source tree — no code is duplicated, so the formatter prompt and the scoring math are byte-identical to what `npm run eval` uses. No modifications were made to the macos-vision or woladi repos to produce these numbers; the benchmark scripts and intermediate predictions are intentionally kept in a scratch directory outside both repos.

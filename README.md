# Adrian Wołczuk

**Staff Front-end Engineer · Design Systems · Privacy-First AI Infrastructure**

I design enterprise-scale front-end architectures and build local-first infrastructure for AI agents.
15+ years of professional experience — most recently as Front-end Tech Lead at CGI; currently focused on the MCP layer for privacy-preserving document pipelines.

Available for consulting on design systems, AI-augmented DX, and privacy-first architecture.

→ awolczuk [at] gmail [dot] com · [LinkedIn](https://www.linkedin.com/in/adrianwolczuk) · Warsaw · Remote

---

## 🏆 Selected Work

### Enterprise PWA · CGI
Architected a modern insurance PWA achieving a **96% Google Page Speed score**:
- Component library with full Storybook documentation and testing
- Storyblok integration enabling visual editing independent of dev cycles
- AI-augmented delivery pipeline using Figma Code Connect to sync design tokens with production code

### Production SaaS · Solo build
Full-stack subscription web app shipped to production. Stack highlights:
- **Streaming LLM responses** — Vercel AI SDK + Claude, token-by-token rendering
- **Custom auth bridge** — Clerk JWT decoded inside Supabase RLS policies (non-standard integration)
- **Heavy compute in serverless** — C library compiled to WASM, singleton-cached across requests
- **Subscription billing** — Stripe Checkout with dynamic pricing and promo codes

---

## 🛠 Open Source — Privacy-First AI Ecosystem

One philosophy across all repos: **AI should work for you, not harvest your data.** Local inference.

### The privacy proxy
**[pseudonym-mcp](https://github.com/woladi/pseudonym-mcp)** — replaces PII with reversible tokens (`[PERSON:1]`, `[PESEL:1]`, `[CREDIT_CARD:1]`) before your prompt reaches the cloud, then restores original values in the response. GDPR-compliant pseudonymisation, fully offline, works with Claude, GPT-4, Gemini and any MCP-compatible client.

```bash
claude mcp add pseudonym-mcp -- npx pseudonym-mcp
```

### The vision stack
Node.js Apple Vision ecosystem for local OCR & image analysis on macOS — same engine, different surfaces:

- **[macos-vision](https://github.com/woladi/macos-vision)** — the foundation. Apple Vision OCR for Node.js, native and fast.
- **[macos-vision-mcp](https://github.com/woladi/macos-vision-mcp)** — MCP server for Claude Code & Claude Desktop. Pre-extracts text before your AI sees it, cutting token usage by ~97% on real documents.
- **[sortai](https://github.com/woladi/sortai)** — macOS CLI that scans a folder, OCRs every PDF and image with Apple Vision, and writes native **Finder tags** (`#Faktura`, `#Umowa`, `#Bank`…) and **Finder comments** directly into file metadata — so your documents become searchable in Spotlight and browsable by tag in Finder, with no extra database. Runs fully offline via local Ollama by default; cloud LLMs optional. When cloud mode is enabled, integrates `pseudonym-mcp` to mask PII before anything leaves the machine.

```bash
claude mcp add macos-vision-mcp -- npx macos-vision-mcp
```

---

## ✍️ Writing

- [Your Obsidian Vault as a Private Second Brain — Powered by Local AI](https://woladi.github.io/woladi/obsidian-privacy-pipeline) — how to use `macos-vision-mcp` + `pseudonym-mcp` to query your vault with cloud AI without exposing personal data.
- [Your Messaging Apps as a Private Document AI — Powered by OpenClaw](https://woladi.github.io/woladi/openclaw-privacy-pipeline) — using `macos-vision-mcp` + `pseudonym-mcp` inside OpenClaw to OCR and anonymise documents sent over WhatsApp, Telegram, or Slack before they reach any cloud LLM.
- [Apple Vision vs Tesseract — A 50-File OCR-to-Markdown Benchmark](https://woladi.github.io/woladi/macos-vision-vs-tesseract-ocr-benchmark) — head-to-head OCR comparison on 50 PDFs with identical LLM formatter input; Tesseract wins on CER, Apple Vision wins on structural quality.
- [Privacy Tiers for Document AI — Three Pipeline Configurations](https://woladi.github.io/woladi/document-ai-privacy-tiers) — fully local vs local OCR + cloud LLM vs local OCR + pseudonymisation + cloud LLM: what each configuration protects, where the trade-offs are, and how to choose.
- [LLM as a Bridge Between Qualitative and Quantitative Research](https://woladi.github.io/woladi/llm-qual-quant-bridging) — using LLM-as-coder and LLM-as-judge to classify interviews and project free-form text onto validated scales (PHQ-9, BDI-II, SF-36), with bias, non-determinism, and IRB caveats.

---

## 🌍 Beyond the Code

Outside software I serve on the management board of a large residential community in Warsaw, co-managing a 7-figure budget and vendor contracts.

---

## 💻 Tech Stack

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-F05138?style=flat-square&logo=swift&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![Storybook](https://img.shields.io/badge/Storybook-FF4785?style=flat-square&logo=storybook&logoColor=white)
![Figma](https://img.shields.io/badge/Figma-F24E1E?style=flat-square&logo=figma&logoColor=white)
![Claude](https://img.shields.io/badge/Claude_API-D97757?style=flat-square&logo=anthropic&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=flat-square&logo=stripe&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=flat-square&logo=clerk&logoColor=white)
![Storyblok](https://img.shields.io/badge/Storyblok-09B3AF?style=flat-square&logo=storyblok&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=flat-square&logo=ollama&logoColor=white)

---

<h3 align="center">📊 Engineering Metrics</h3>

<p align="center">
  <img src="https://github-readme-streak-stats.herokuapp.com/?user=woladi&theme=tokyonight&hide_border=true&date_format=j%20M%5B%20Y%5D" height="160" />
</p>

# Adrian Wołczuk

**Staff Product Engineer · Agentic AI Workflows · Privacy-First AI Infrastructure · Design Systems**

I ship production systems end-to-end — from front-end architecture and design systems to local-first AI infrastructure.
Currently on contract with a major tech company, building a new platform where **agentic coding is the daily workflow** — orchestrating coding agents at serious token budgets, owning features end-to-end across web and native surfaces.
15+ years of professional experience (CGI, KPMG, Acxiom). Solo-built a subscription SaaS and a privacy-first AI tool ecosystem.

Open to **founding / staff engineering roles**, **technical co-founder** opportunities, and **selective consulting** — Q4 2026 availability.

→ awolczuk [at] gmail [dot] com · [LinkedIn](https://www.linkedin.com/in/adrianwolczuk) · Warsaw · Remote

---

## 🏆 Selected Work

### Enterprise Agentic Platform · 2026

Building a greenfield platform for a major tech company with agentic coding as the core delivery model:

- Orchestrating coding agents end-to-end — architecture, agent direction, review, and quality gates on agent output
- Token budgets of hundreds of dollars per engineer per day — production-scale agentic development, not experimentation
- Full features shipped across web and native surfaces, from design-system components to complete integrations

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
Early-stage with an active roadmap — the direction is a deterministic sanitization layer for any agent workflow touching sensitive data.

```bash
claude mcp add pseudonym-mcp -- npx pseudonym-mcp
```

### The vision stack
Node.js Apple Vision ecosystem for local OCR & image analysis on macOS — same engine, different surfaces:

- **[macos-vision](https://github.com/woladi/macos-vision)** — the foundation. Apple Vision OCR for Node.js, native and fast.
- **[macos-vision-mcp](https://github.com/woladi/macos-vision-mcp)** — MCP server for Claude Code & Claude Desktop. Pre-extracts text before your AI sees it, cutting token usage by ~97% on real documents. Also does on-device UI testing: screenshots, accessibility trees, and element lookup — nothing leaves the machine.
- **macos-vision-mcp v2** *(in progress)* — built on the newest Apple Vision capabilities: live web-page operation for agents, fully on-device. Combined with `pseudonym-mcp`, this raises the ceiling on what an agent can safely automate on the web.
- **[sortai](https://github.com/woladi/sortai)** — macOS CLI that OCRs every PDF and image in a folder with Apple Vision and writes native **Finder tags** (`#Faktura`, `#Umowa`, `#Bank`…) and **comments** into file metadata — instantly searchable in Spotlight, no database. Offline via Ollama by default; cloud LLMs optional, with `pseudonym-mcp` masking PII first.

```bash
claude mcp add macos-vision-mcp -- npx macos-vision-mcp
```

### The legal verifier
**[sententim](https://github.com/woladi/sententim)** — deterministic MCP server that verifies whether a Polish court judgment signature actually exists before you cite it. LLMs hallucinate legal case numbers; this tool makes them stop. Zero LLM at runtime — purely SQLite + FTS5 lookups, sub-10ms, fully offline. Returns `FOUND`, `NOT_FOUND`, or `AMBIGUOUS` against a local database built from SAOS public data.

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
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)
![Claude API](https://img.shields.io/badge/Claude_API_(MCP)-D97757?style=flat-square&logo=anthropic&logoColor=white)

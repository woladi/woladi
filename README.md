# Hey, I'm Adrian 👋

**Front-end Tech Lead · React & Typescript · genAI/DX**

10+ years building enterprise front-end systems at CGI. I specialize in closing the gap between design and engineering — through scalable component libraries, AI-augmented workflows, and developer experience that actually ships.

Open to consulting on design system architecture or AI-augmented DX pipelines. → [Let's talk](mailto:awolczuk@gmail.com)

## 🚀 What I Do

- **Design Systems** → Scalable UI libraries with Storybook, design tokens, and cross-team governance
- **Headless CMS** → Storyblok architecture that empowers content teams without dev bottlenecks
- **AI-Driven Engineering** → MCP servers, Figma Code Connect, streaming LLM apps in production

## 🏆 Key Achievement

Architected a modern insurance PWA at CGI achieving a **95% Google Page Speed score**:
- Component library with full Storybook documentation and testing
- Storyblok integration enabling visual editing independent of dev cycles
- AI-augmented delivery pipeline using MCP + Figma Code Connect to sync design tokens with production code

## 🔧 Side Project — Production AI App

Solo-built and shipped a full-stack subscription web app currently running in production. Technical highlights:

- **Streaming LLM responses** → Vercel AI SDK + Claude, token-by-token rendering to the client
- **Custom auth bridge** → Clerk JWT decoded inside Supabase RLS policies (non-standard integration)
- **Heavy compute in serverless** → C library compiled to WASM, singleton-cached across requests
- **Subscription billing** → Stripe Checkout with dynamic pricing and promo codes

## 🛠 Open Source — Privacy-First AI Tooling

All projects share a common philosophy: **AI should work for you, not harvest your data.**
Local inference. No cloud APIs. No subscriptions.

### [pseudonym-mcp](https://github.com/woladi/pseudonym-mcp)
Local privacy proxy for LLMs — replaces PII with reversible tokens (`[PERSON:1]`, `[PESEL:1]`, `[CREDIT_CARD:1]`) before your prompt ever reaches the cloud, then restores original values in the response. GDPR-compliant pseudonymisation, fully offline, works with Claude, GPT-4, Gemini and any MCP-compatible client.

```bash
claude mcp add pseudonym-mcp -- npx pseudonym-mcp
```

### [macos-vision-mcp](https://github.com/woladi/macos-vision-mcp)
MCP server for Claude Code & Claude Desktop — local OCR and image analysis via Apple Vision. Files never leave your Mac: no cloud API, no API keys, no network requests. Pre-extracts text before your AI ever sees it, cutting token usage by ~97% on real documents.

```bash
claude mcp add macos-vision-mcp -- npx macos-vision-mcp
```

### [macos-vision-md](https://github.com/woladi/macos-vision-md)
Convert images and PDFs to structured Markdown using Apple Vision OCR + a local Ollama model — fully offline, no cloud APIs, no subscriptions.

```bash
npm install macos-vision-md
```

### [macos-vision](https://github.com/woladi/macos-vision)
Apple Vision OCR & image analysis for Node.js — native, fast, offline, no API keys required.

```bash
npm install macos-vision
```

### [AiSort](https://github.com/woladi/AiSort) *(early stage)*
Automatically tags and annotates files using `macos-vision-mcp` and local LLMs — no cloud, no subscriptions. Files never leave your machine. Feed it a better model and it can also suggest smarter folder structures for your documents.

## ✍️ Featured Article

[Your Obsidian Vault as a Private Second Brain — Powered by Local AI](https://woladi.github.io/woladi/obsidian-privacy-pipeline) — how to use `macos-vision-mcp` + `pseudonym-mcp` to query your vault with cloud AI without exposing personal data.

## 💻 Tech Stack

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Claude](https://img.shields.io/badge/Claude_API-D97757?style=flat-square&logo=anthropic&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=flat-square&logo=stripe&logoColor=white)
![Storybook](https://img.shields.io/badge/Storybook-FF4785?style=flat-square&logo=storybook&logoColor=white)
![Storyblok](https://img.shields.io/badge/Storyblok-09B3AF?style=flat-square&logo=storyblok&logoColor=white)
![Figma](https://img.shields.io/badge/Figma-F24E1E?style=flat-square&logo=figma&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-F05138?style=flat-square&logo=swift&logoColor=white)

## 📫 Let's Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/adrianwolczuk)
[![Email](https://img.shields.io/badge/Email-EA4335?style=flat-square&logo=gmail&logoColor=white)](mailto:awolczuk@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/woladi)

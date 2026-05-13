# Hey, I'm Adrian 👋

**Front-end Tech Lead · React & TypeScript · genAI/DX**

10+ years building enterprise front-end systems at CGI. I close the gap between design and engineering — through scalable component libraries, AI-augmented workflows, and developer experience that actually ships.

### 💼 Available for consulting

- **Design Systems** — component libraries, design tokens, Storybook, cross-team governance
- **AI-augmented DX** — MCP servers, Figma Code Connect, streaming LLM apps in production
- **Privacy-first AI** — local inference pipelines, GDPR-compliant LLM integrations

→ **[awolczuk@gmail.com](mailto:awolczuk@gmail.com)** · [LinkedIn](https://www.linkedin.com/in/adrianwolczuk) · [GitHub](https://github.com/woladi)

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
Node.js Apple Vision Middleware four-layer ecosystem for local OCR & image analysis on macOS — same engine, different surfaces:

- **[macos-vision](https://github.com/woladi/macos-vision)** — the foundation. Apple Vision OCR for Node.js, native and fast.
- **[macos-vision-mcp](https://github.com/woladi/macos-vision-mcp)** — MCP server for Claude Code & Claude Desktop. Pre-extracts text before your AI sees it, cutting token usage by ~97% on real documents.
- **[AiSort](https://github.com/woladi/AiSort)** *(early stage)* — auto-tags and annotates files using the stack above. Files never leave your machine.

```bash
claude mcp add macos-vision-mcp -- npx macos-vision-mcp
```

---
## ✍️ Writing

- [Your Obsidian Vault as a Private Second Brain — Powered by Local AI](https://woladi.github.io/woladi/obsidian-privacy-pipeline) — how to use `macos-vision-mcp` + `pseudonym-mcp` to query your vault with cloud AI without exposing personal data.
- [Your Messaging Apps as a Private Document AI — Powered by OpenClaw](https://woladi.github.io/woladi/openclaw-privacy-pipeline) — using `macos-vision-mcp` + `pseudonym-mcp` inside OpenClaw to OCR and anonymise documents sent over WhatsApp, Telegram, or Slack before they reach any cloud LLM.
---

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

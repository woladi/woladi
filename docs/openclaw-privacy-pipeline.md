# 📱 Your Messaging Apps as a Private Document AI — Powered by OpenClaw

> 📱 Send a PDF on WhatsApp. 🍎 OCR it locally. 🕵️ Mask PII. 🤖 Ask Claude. 🔒 Zero personal data reaches the cloud.

---

## 😤 The Problem With AI on Messaging

People send sensitive documents over WhatsApp, Telegram, and Slack every day — medical reports, bank statements, NDAs, lease agreements, payslips. When you forward them to an AI assistant, the document hits a cloud server before any reasoning happens.

Most setups look like this:

```
📱 You → (document with real names, SSNs, card numbers) → Cloud Gateway → LLM API
```

The gap is at the gateway. Even if you trust the LLM provider, your raw document crosses multiple hops — infrastructure you don't control, logs you can't audit.

**✅ This guide closes that gap using three components that all run on your Mac.**

---

## 🛠️ The Stack

| Component | What it does | Key technology |
| --- | --- | --- |
| [OpenClaw](https://github.com/openclaw/openclaw) | Receives messages from WhatsApp / Telegram / Slack, routes them to a local agent | Local-first gateway — self-hosted |
| [`macos-vision-mcp`](https://github.com/woladi/macos-vision-mcp) | Extracts text from images and PDFs sent via messaging | Apple Vision framework — fully offline |
| [`pseudonym-mcp`](https://github.com/woladi/pseudonym-mcp) | Replaces PII with reversible tokens before anything reaches the cloud | Regex NER + local Ollama — fully offline |

OpenClaw acts as the local control plane. Your messages arrive there first. The two MCP servers plug in as tools available to its agent — no data leaves your Mac until PII is already masked.

---

## ⚙️ How It Works

```
📱 WhatsApp / Telegram / Slack
          │
          ▼
  🏠 OpenClaw  (local gateway)
          │
          ├── plain text ──────────► pseudonym-mcp
          │                              mask_text()
          │                                  │
          └── file / image ──► macos-vision-mcp
                                   extract_text()
                                        │
                                        └──► pseudonym-mcp
                                               mask_text()
                                                    │
                                                    ▼
                                         [PERSON:1], [SSN:1],
                                         [CREDIT_CARD:1], [PESEL:1]...
                                                    │
                                                    ▼
                                          ☁️ Cloud LLM API
                                     (Claude / GPT-4 / Gemini)
                                                    │
                                            response with
                                             tokens only
                                                    │
                                                    ▼
                                          🔓 pseudonym-mcp
                                            unmask_text()
                                                    │
                                                    ▼
                                        ✅ Real names restored
                                                    │
                                                    ▼
                                       📱 Reply in your app
```

The LLM reasons about structure, obligations, and meaning — never about real identities. The unmask step happens locally before the reply is sent back.

---

## 🚀 Setup (10 minutes)

### Step 1 — Install and onboard OpenClaw

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

The wizard guides you through connecting a channel (WhatsApp, Telegram, Slack…) and choosing a cloud model (Claude, GPT-4, Gemini).

---

### Step 2 — Register the privacy MCP servers

```bash
openclaw mcp set macos-vision-mcp '{"command": "npx", "args": ["macos-vision-mcp"]}'
openclaw mcp set pseudonym-mcp '{"command": "npx", "args": ["pseudonym-mcp", "--engines", "hybrid"]}'
```

Verify both are registered:

```bash
openclaw mcp list
```

**Alternative — edit `~/.openclaw/openclaw.json` directly:**

```json
{
  "mcp": {
    "servers": {
      "macos-vision-mcp": {
        "command": "npx",
        "args": ["macos-vision-mcp"]
      },
      "pseudonym-mcp": {
        "command": "npx",
        "args": ["pseudonym-mcp", "--engines", "hybrid"]
      }
    }
  }
}
```

---

### Step 3 — Pull an Ollama model *(optional, for name and organisation masking)*

```bash
ollama pull llama3
```

> 💡 Skip this if you only need regex-based masking — SSN, credit cards, PESEL, IBAN, phone, email are covered without Ollama.

---

## 💡 Real-World Use Cases

### 📋 1. Summarise a contract sent via WhatsApp

Your lawyer forwards an NDA over WhatsApp. Instead of opening a web-based AI tool and pasting the content:

```
Forward me the PDF and ask:
Extract text from the attached file, mask all PII, then summarise
the key obligations, deadlines, and termination clauses.
Restore real names in the final answer.
```

**What happens:**
- 🍎 Apple Vision reads the PDF locally — no cloud OCR
- 🕵️ pseudonym-mcp replaces names, company names, and IBAN numbers with tokens
- 🤖 The LLM summarises obligations without ever seeing real identities
- ✅ Names are restored in the reply that arrives back in WhatsApp

---

### 🏥 2. Medical report via Telegram

Your doctor sends a scan of a cardiology report:

```
Summarise this report in plain language and suggest questions
I should prepare for my next appointment.
```

Your doctor's name, your SSN, the diagnosis — all tokenised locally. The cloud provider never processes Protected Health Information. No BAA required. ✅

---

### 💳 3. Bank statement categorisation via Slack

You drop a screenshot of your bank statement into a private Slack channel connected to OpenClaw:

```
Extract the transactions from this image, mask all card numbers
and account holders, then group them by category (food, transport,
subscriptions) and give me a monthly total.
```

Card numbers pass through as `[CREDIT_CARD:1]`. Account names pass as `[PERSON:1]`. The LLM categorises patterns — not your financial identity.

---

### 📂 4. Multi-file analysis with a persistent session

OpenClaw routes a session across multiple messages. You can chain masking across files and keep the token mapping consistent:

```
# Message 1
Extract and mask the text from invoice_jan.pdf — remember the session.

# Message 2
Do the same for invoice_feb.pdf using the same session.

# Message 3
Which supplier charged the most across both months?
Restore names in the answer.
```

> 💡 `[PERSON:1]` and `[ORG:1]` remain stable across all messages in the session — the LLM can reason about patterns and relationships without ever knowing real identities.

---

### ✍️ 5. Handwritten notes via any channel

You photograph a handwritten meeting note and send it:

```
Transcribe this note and extract the action items with owners and deadlines.
```

Apple Vision handles handwriting recognition natively. Owners are masked before the LLM sees them, then restored in the action-item list.

---

## ⚡ One-Click Privacy — Built-in Prompt Templates

`pseudonym-mcp` ships two prompt templates that chain the full pipeline automatically. They work the same way inside OpenClaw as in Claude Desktop.

### `pseudonymize_task` — inline text

```
/pseudonymize_task text="Meeting with Jan Kowalski (PESEL: 90010112318). Contract: 45 000 zł." task="Extract action items"
```

1. PII masked locally → `[PERSON:1]`, `[PESEL:1]`
2. LLM processes anonymised text
3. Originals restored in the response

Optional `lang` argument: `en` (default) or `pl`.

### `privacy_scan_file` — file or image path

> Requires **macos-vision-mcp** alongside pseudonym-mcp.

```
/privacy_scan_file filePath="/path/to/document.pdf" task="Summarise key obligations"
```

1. macos-vision-mcp extracts text locally via Apple Vision
2. pseudonym-mcp masks all PII before any cloud call
3. LLM processes anonymised content
4. Originals restored before the reply is shown

---

## 🛡️ What Gets Protected

### 🇺🇸 English (`--lang en`, default)

| Token | Covers |
| --- | --- |
| `[PERSON:1]` | 👤 Full names (via Ollama NER) |
| `[ORG:1]` | 🏢 Organisation names (via Ollama NER) |
| `[SSN:1]` | 🪪 Social Security Numbers — with area-number validation |
| `[CREDIT_CARD:1]` | 💳 13–19 digit card numbers — with Luhn checksum |
| `[EMAIL:1]` | 📧 Email addresses |
| `[PHONE:1]` | 📱 US phone formats |

> 🌍 **Polish users:** `--lang pl` adds PESEL, Polish IBAN, and Polish phone formats.

---

## 🔒 Privacy Guarantees

- **📡 No telemetry.** Neither MCP package makes any network request except to your local Ollama instance.
- **💾 In-memory only.** Token mappings are never written to disk. Sessions expire when OpenClaw restarts — no PII lingers.
- **✈️ Offline OCR.** Apple Vision runs entirely on-device via Neural Engine. No image or document is ever uploaded.
- **📦 Zero production dependencies.** pseudonym-mcp has no third-party runtime dependencies — a deliberate security decision for a tool handling sensitive data.
- **🔁 Idempotent tokens.** The same value always maps to the same token within a session, preserving coherence across multi-message conversations.

---

## ⚖️ Regulatory Alignment

| Regulation | Who it affects | How the pipeline helps |
| --- | --- | --- |
| **🇺🇸 HIPAA** | Healthcare providers, patients | PHI never reaches a non-BAA cloud provider |
| **💳 PCI DSS 3.4** | Anyone handling card data | Card numbers masked before LLM transit |
| **🇪🇺 GDPR Art. 44** | EU users & businesses | No personal data transferred cross-border |
| **🏢 SOC 2** | SaaS & enterprise | Demonstrates PII leaves no trust boundary |

> ⚠️ **Note:** Pseudonymisation does not equal anonymisation — the data remains personal in your local system. However, it substantially reduces risk and demonstrates compliance with accountability principles.

---

## ⚠️ Disclaimer

This pipeline is a **risk-reduction tool, not a guarantee of zero data exposure.**

Pseudonymisation is a compromise by design: it replaces identifiable values with tokens, but the surrounding context — sentence structure, topic, document type, dates, amounts — is still sent to the cloud. A sufficiently determined party with access to LLM logs and additional context could potentially re-identify individuals from that surrounding content alone.

No tool, including `macos-vision-mcp` and `pseudonym-mcp`, can provide a 100% guarantee that personal data will never leak or be inferred. Edge cases exist:

- NER models miss names they haven't seen before or names written unconventionally
- Regex patterns don't cover every format of every country's ID numbers
- Contextual clues in the text (job title + city + employer) may be enough to identify someone even without explicit PII
- Bugs in any software layer — OCR, masking, transport — can cause unexpected behaviour

**Use this pipeline as one layer of a broader privacy strategy, not as a substitute for legal advice, a BAA, or a formal data protection assessment.** If you're handling data subject to strict regulatory requirements (HIPAA, GDPR Article 9 special categories, classified information), consult a qualified professional before relying on any automated pseudonymisation tool.

---

## 📦 Quick Start Reference

```bash
# 1. Install OpenClaw
npm install -g openclaw@latest
openclaw onboard --install-daemon

# 2. Add MCP servers
openclaw mcp set macos-vision-mcp '{"command": "npx", "args": ["macos-vision-mcp"]}'
openclaw mcp set pseudonym-mcp '{"command": "npx", "args": ["pseudonym-mcp", "--engines", "hybrid"]}'

# 3. Optional: full NER
ollama pull llama3

# 4. Verify
openclaw mcp list
```

Or as `~/.openclaw/openclaw.json`:

```json
{
  "mcp": {
    "servers": {
      "macos-vision-mcp": {
        "command": "npx",
        "args": ["macos-vision-mcp"]
      },
      "pseudonym-mcp": {
        "command": "npx",
        "args": ["pseudonym-mcp", "--engines", "hybrid"]
      }
    }
  }
}
```

---

## 🔗 Links

- 🕵️ **pseudonym-mcp** — [npm](https://www.npmjs.com/package/pseudonym-mcp) · [GitHub](https://github.com/woladi/pseudonym-mcp)
- 📸 **macos-vision-mcp** — [GitHub](https://github.com/woladi/macos-vision-mcp)
- 🤖 **OpenClaw** — [GitHub](https://github.com/openclaw/openclaw)
- 📄 License: MIT — Adrian Wolczuk

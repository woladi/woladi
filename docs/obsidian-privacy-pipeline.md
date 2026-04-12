# 🧠 Your Obsidian Vault as a Private Second Brain — Powered by Local AI

> 📄 Scan PDFs. ✍️ Transcribe handwriting. 🤖 Ask Claude anything. 🔒 Zero personal data leaves your Mac.

---

## 😤 The Problem Nobody Has Solved

Obsidian users have built elaborate second brains — journals, medical notes, financial records, scanned contracts, handwritten meeting notes. The vault becomes deeply personal. Then AI arrives, and the promise is irresistible: _"Ask your entire knowledge base a question."_

But there's a catch.

Every plugin that connects your vault to a cloud LLM (Claude, GPT-4, Gemini) sends your raw notes upstream. Social Security Numbers, credit card numbers, medical diagnoses, client names — all of it lands on a third-party server. For anyone who takes their vault seriously, this is a dealbreaker.

The alternatives are bleak: run a weak local LLM and accept lower quality, or redact manually before every query. Neither is sustainable.

**✅ This guide introduces a third path.**

---

## 🛠️ The Stack

Two open-source MCP servers, both by the same author, designed to work together:

| Package                                                          | What it does                                              | Key technology                           |
| ---------------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------- |
| [`macos-vision-mcp`](https://github.com/woladi/macos-vision-mcp) | 📸 Extracts text from PDFs, images, and handwritten notes | Apple Vision framework — fully offline   |
| [`pseudonym-mcp`](https://www.npmjs.com/package/pseudonym-mcp)   | 🕵️ Replaces PII with opaque tokens before any cloud call  | Regex NER + local Ollama — fully offline |

Both run on your Mac. Neither sends data anywhere. Together, they form a complete local privacy pipeline between your vault and any cloud LLM.

---

## ⚙️ How It Works

```
🗂️ Your Obsidian Vault
        │
        ├── 📝 Markdown notes (.md)
        │         │
        │         └──► pseudonym-mcp
        │                mask_text()
        │                   │
        └── 📄 Scanned files (.pdf, .jpg, .png)
                  │
                  └──► macos-vision-mcp
                         extract_text()
                              │
                              └──► pseudonym-mcp
                                     mask_text()
                                          │
                                          ▼
                                  [PERSON:1], [SSN:1],
                                  [CREDIT_CARD:1], [EMAIL:1]...
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
                            ✅ You see real names & data
```

At no point does any personal data leave your machine. The cloud LLM reasons about structure and content — never about identities.

---

## 🚀 Setup (5 minutes)

**Step 1 — Add both servers to Claude Desktop:**

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "macos-vision-mcp": {
      "command": "npx",
      "args": ["-y", "macos-vision-mcp"]
    },
    "pseudonym-mcp": {
      "command": "npx",
      "args": ["-y", "pseudonym-mcp", "--engines", "hybrid"]
    }
  }
}
```

Restart Claude Desktop. Both tool sets appear automatically. ✨

**Step 2 — Pull an Ollama model** _(optional, for full NER including names and organisations):_

```sh
ollama pull llama3
```

> 💡 Skip this if you only need regex-based masking (SSN, credit cards, email, phone).

**Step 3 — For Claude Code:**

```sh
claude mcp add macos-vision-mcp -- npx -y macos-vision-mcp
claude mcp add pseudonym-mcp -- npx -y pseudonym-mcp --engines hybrid
```

---

## 💡 Real-World Use Cases

### 📋 1. Summarise a scanned contract

You have a PDF scan of a lease agreement in your vault:

```
vault/legal/lease_agreement_2026.pdf
```

In Claude Desktop or Claude Code:

```
Extract text from vault/legal/lease_agreement_2026.pdf using macos-vision-mcp,
then mask all PII with pseudonym-mcp (save the session_id),
then summarise the key obligations, deadlines, and termination conditions.
Finally, restore the response using the session_id.
```

**🔍 What happens:**

- 🍎 Apple Vision reads the PDF locally — no cloud OCR
- 🕵️ pseudonym-mcp replaces names, SSNs, and card numbers with tokens
- 🤖 Claude summarises structure and obligations without seeing real data
- ✅ Original names are restored in the final answer

---

### ✍️ 2. Transcribe handwritten notes

You photograph a page from your notebook and drop it into the vault:

```
vault/journal/2026-04-12.jpg
```

```
Transcribe my handwritten note at vault/journal/2026-04-12.jpg
and save it as vault/journal/2026-04-12.md
```

Apple Vision handles handwriting recognition natively. The resulting Markdown note is fully searchable inside Obsidian. 🔍

---

### 💰 3. Categorise monthly expenses

You keep iPhone camera scans of receipts in your vault:

```
vault/finance/receipts/april/
```

```
Extract text from all images in vault/finance/receipts/april/,
mask PII with pseudonym-mcp (single session for all files),
then create a categorised expense summary for April 2026
and save it as vault/finance/2026-04-summary.md
```

Credit card numbers, account holders, and names are tokenised before Claude sees them. 🔐

---

### 🔭 4. Query your entire vault with a persistent session

This is the most powerful use case — a session that spans multiple notes:

```
# Step 1: mask vault notes, save the session_id
Use mask_text on all notes in vault/work/ — remember the session_id

# Step 2: ask anything
Which clients did I meet most frequently in Q1 2026?
What were the main topics across my meetings with [PERSON:1]?

# Step 3: restore when done
Use unmask_text with the saved session_id on the response
```

> 💡 `[PERSON:1]` always refers to the same person across all notes in the session — Claude can reason about relationships and patterns without ever knowing real identities.

---

### 🏥 5. Medical notes without HIPAA risk

```
vault/health/2026-03-cardiology-visit.md
```

```
Mask this note with pseudonym-mcp, then explain the diagnosis in plain language
and suggest questions I should ask at my next appointment.
```

Your doctor's name, SSN, and diagnosis are tokenised locally. The cloud provider never processes your Protected Health Information (PHI). No Business Associate Agreement (BAA) required. ✅

---

## ⚡ One-Click Privacy — Built-in Prompt Templates

Instead of typing the full pipeline every time, pseudonym-mcp ships two built-in prompt templates that chain masking, the LLM task, and unmasking automatically.

### `pseudonymize_task` — inline text

```
/pseudonymize_task text="Meeting with Jan Kowalski (PESEL: 90010112318). Contract: 45 000 zł." task="Extract action items"
```

What happens:
1. pseudonym-mcp masks PII locally → `[PERSON:1]`, `[PESEL:1]`
2. Claude processes the anonymized text
3. pseudonym-mcp restores originals in the response

Optional `lang` argument: `en` (default) or `pl`.

### `privacy_scan_file` — file or PDF

> Requires **macos-vision-mcp** to be installed alongside pseudonym-mcp.

```
/privacy_scan_file filePath="/Users/me/vault/contracts/nda.pdf" task="Summarize obligations and deadlines"
```

What happens:
1. macos-vision-mcp extracts text from the file locally via Apple Vision
2. pseudonym-mcp masks all PII before sending anything to the cloud
3. Claude processes the anonymized content
4. pseudonym-mcp restores originals before the response is shown

Optional arguments: `task` (default: _summarize the key points_), `lang` (`en` or `pl`).

---

## 🛡️ What Gets Protected

### 🇺🇸 English PII (`--lang en`, default)

| Token             | Covers                                                   |
| ----------------- | -------------------------------------------------------- |
| `[PERSON:1]`      | 👤 Full names (via Ollama NER)                           |
| `[SSN:1]`         | 🪪 Social Security Numbers — with area-number validation |
| `[CREDIT_CARD:1]` | 💳 13–19 digit card numbers — with Luhn checksum         |
| `[EMAIL:1]`       | 📧 Email addresses                                       |
| `[PHONE:1]`       | 📱 US phone formats                                      |
| `[ORG:1]`         | 🏢 Organisation names (via Ollama NER)                   |

> 🌍 **International users:** `--lang pl` adds support for PESEL (national ID), Polish IBAN, and Polish phone formats.

---

## 🔒 Privacy Guarantees

- **📡 No telemetry.** Neither package makes any network request except to your local Ollama instance.
- **💾 In-memory by default.** Token mappings are never written to disk. Sessions expire when Claude Desktop restarts — this is intentional. No PII lingers.
- **✈️ Offline OCR.** Apple Vision runs entirely on-device using Neural Engine acceleration. No image or document is ever uploaded.
- **📦 Zero production dependencies.** pseudonym-mcp has no third-party runtime dependencies — a deliberate security decision for a tool handling sensitive data.
- **🔁 Idempotent tokens.** The same value always maps to the same token within a session, preserving semantic coherence across multi-note queries.

---

## ⚖️ Regulatory Alignment

| Regulation          | Who it affects                    | How the pipeline helps                                                     |
| ------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| **🇺🇸 HIPAA**        | Healthcare providers, patients    | PHI never reaches a non-BAA cloud provider                                 |
| **💳 PCI DSS 3.4**  | Anyone storing card data          | Card numbers masked before LLM transit                                     |
| **🇺🇸 CCPA / CPRA**  | California residents & businesses | Minimises personal data sent to third-party processors                     |
| **🏢 SOC 2**        | SaaS & enterprise                 | Demonstrates PII leaves no trust boundary                                  |
| **🇪🇺 GDPR Art. 44** | EU users & businesses             | If no personal data is transferred, cross-border restrictions do not apply |

> ⚠️ **Note:** Pseudonymisation does not equal anonymisation — the data remains personal data in your local system. However, it substantially reduces risk and demonstrates compliance with accountability principles.

---

## 🌿 Why This Matters for Obsidian Users Specifically

Obsidian's core philosophy is _local-first_: your data lives on your device, in plain text, under your control. Every file is yours.

Cloud AI plugins break this contract the moment they send vault content upstream. `macos-vision-mcp` + `pseudonym-mcp` extend the local-first principle to AI: ☁️ cloud model quality, 🏠 local data sovereignty.

**Your second brain stays yours. 🧠🔒**

---

## 📦 Quick Start Reference

```json
// Claude Desktop — ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "macos-vision-mcp": {
      "command": "npx",
      "args": ["-y", "macos-vision-mcp"]
    },
    "pseudonym-mcp": {
      "command": "npx",
      "args": ["-y", "pseudonym-mcp", "--engines", "hybrid"]
    }
  }
}
```

```sh
# Claude Code
claude mcp add macos-vision-mcp -- npx -y macos-vision-mcp
claude mcp add pseudonym-mcp -- npx -y pseudonym-mcp --engines hybrid

# Optional: full NER (names + organisations)
ollama pull llama3
```

---

## 🔗 Links

- 🕵️ **pseudonym-mcp** — [npm](https://www.npmjs.com/package/pseudonym-mcp) · [GitHub](https://github.com/woladi/pseudonym-mcp)
- 📸 **macos-vision-mcp** — [GitHub](https://github.com/woladi/macos-vision-mcp)
- 📄 License: MIT — Adrian Wolczuk

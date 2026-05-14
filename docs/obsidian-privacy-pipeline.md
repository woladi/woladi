# 🧠 Your Obsidian Vault Meets Cloud AI — With a Local Privacy Layer

> 📄 Scan PDFs. ✍️ Transcribe handwriting. 🤖 Ask Claude anything. 🔒 Add a local pseudonymisation layer between your vault and the cloud.

---

## 😤 The Problem

Obsidian users build elaborate second brains — journals, medical notes, financial records, scanned contracts, handwritten meeting notes. The vault becomes deeply personal. Then AI arrives, and the promise is irresistible: _"Ask your entire knowledge base a question."_

But there's a catch.

Most plugins that connect your vault to a cloud LLM (Claude, GPT-4, Gemini) send your raw notes upstream. Names, account numbers, medical diagnoses, client identifiers — all of it lands on a third-party server in cleartext. For anyone who takes their vault seriously, sending raw notes by default isn't a comfortable choice.

The usual alternatives are bleak: run a weaker local LLM and accept lower quality, or redact manually before every query. Neither is sustainable.

**✅ This guide introduces a third path: a local pseudonymisation layer between your vault and any cloud LLM.**

It's not a silver bullet. It's a defense-in-depth measure — one more layer between your raw notes and a third-party server. Used thoughtfully, it meaningfully reduces what cleartext PII you ship to the cloud.

---

## ⚠️ What This Is — And What It Isn't

Before the setup, the honest framing:

- **This is pseudonymisation, not anonymisation.** Tokenised data is still personal data under GDPR (Recital 26 is explicit on this) and still PHI under HIPAA. The mapping between `[PERSON:1]` and a real person exists in memory on your machine while the session lives. Re-identification is possible through context — relationships, dates, locations, unusual phrasing.
- **Detection is best-effort, not exhaustive.** Regex catches structured PII (SSN, IBAN, card numbers) reliably. NER catches many names and organisations but misses nicknames, initials, indirect references ("my neighbour on the 3rd floor"), and contextual identifiers ("the meeting at the place we went last summer").
- **The cloud LLM still sees the structure of your life.** It sees that `[PERSON:1]` meets `[PERSON:2]` weekly, that you have a recurring medical issue, that you're in dispute with `[ORG:1]`. That structural information can itself be sensitive.
- **No legal advice here.** Compliance with HIPAA, GDPR, PCI-DSS, etc., depends on your full stack, your role (controller/processor), your contracts, and your jurisdiction. This tooling can be part of a compliance posture; it does not by itself make you compliant with anything.

If you've internalised those caveats — read on. This stack is genuinely useful as a profilaktic layer for personal vaults and research workflows.

---

## 🛠️ The Stack

Two open-source MCP servers, both by the same author, designed to work together:

| Package                                                          | What it does                                              | Key technology                           |
| ---------------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------- |
| [`macos-vision-mcp`](https://github.com/woladi/macos-vision-mcp) | 📸 Extracts text from PDFs, images, and handwritten notes | Apple Vision framework — on-device       |
| [`pseudonym-mcp`](https://www.npmjs.com/package/pseudonym-mcp)   | 🕵️ Replaces detected PII with opaque tokens before any cloud call  | Regex + local Ollama NER          |

Both run on your Mac. The OCR step is fully on-device. The masking step is local; the cloud LLM call is, of course, not — that's the whole point. Together, the pair gives you a local pipeline that strips most direct identifiers before anything reaches a third party.

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
                         extract_text()        (on-device)
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

Detected direct identifiers are replaced with tokens before the cloud call. Structure, relationships, and any PII that detection missed still travel upstream — so treat this as a meaningful reduction in cleartext exposure, not as a hermetic seal.

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

**Step 2 — Pull an Ollama model** _(recommended, for NER on names and organisations):_

```sh
ollama pull llama3
```

> 💡 Without Ollama, masking falls back to regex only — you'll catch structured identifiers (SSN, cards, email, phone, IBAN, PESEL) but not free-form names.

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

- 🍎 Apple Vision reads the PDF on-device — no cloud OCR step
- 🕵️ pseudonym-mcp tokenises detected names, IDs, and account numbers
- 🤖 Claude works on the masked text
- ✅ Tokens are restored in the final answer

Reasonable caveat: the structure of the contract (parties, dates, amounts) still reaches the cloud. Tokenisation hides _who_ — not _what kind of deal_.

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

Apple Vision handles handwriting recognition natively, on-device. The resulting Markdown note is fully searchable inside Obsidian. 🔍

If you then plan to send that note to a cloud LLM, run it through `mask_text` first.

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

Card numbers and account holder names are tokenised before Claude sees them. Merchant names, amounts, and dates are not — they're the substance of the task. Worth thinking about whether your merchant pattern itself is something you're comfortable sharing. 🔐

---

### 🔭 4. Query your entire vault with a persistent session

The most powerful use case — a session that spans multiple notes:

```
# Step 1: mask vault notes, save the session_id
Use mask_text on all notes in vault/work/ — remember the session_id

# Step 2: ask anything
Which clients did I meet most frequently in Q1 2026?
What were the main topics across my meetings with [PERSON:1]?

# Step 3: restore when done
Use unmask_text with the saved session_id on the response
```

> 💡 `[PERSON:1]` always refers to the same person across all notes in the session — Claude can reason about relationships and patterns without seeing the underlying name. The trade-off: that very consistency makes the masked corpus potentially re-identifiable to anyone with side knowledge of your work. Use sessions deliberately.

---

### 🏥 5. Medical notes — a more careful framing

```
vault/health/2026-03-cardiology-visit.md
```

```
Mask this note with pseudonym-mcp, then explain the diagnosis in plain language
and suggest questions I should ask at my next appointment.
```

The provider's name and any structured identifiers get tokenised. The diagnosis, symptoms, medications, and clinical narrative do not — those are exactly what you want the model to reason about.

**Important honesty here:** if you're a HIPAA-covered entity or business associate, this stack does **not** remove your BAA obligations with whichever cloud LLM you use. Pseudonymised PHI is still PHI. For personal use on your own health notes, this is a reasonable privacy posture — for professional clinical workflows, talk to your compliance team and read your vendor's BAA. ⚠️

---

## ⚡ One-Click Privacy — Built-in Prompt Templates

Instead of typing the full pipeline every time, pseudonym-mcp ships two built-in prompt templates that chain masking, the LLM task, and unmasking automatically.

### `pseudonymize_task` — inline text

```
/pseudonymize_task text="Meeting with Jan Kowalski (PESEL: 90010112318). Contract: 45 000 zł." task="Extract action items"
```

What happens:
1. pseudonym-mcp masks detected PII locally → `[PERSON:1]`, `[PESEL:1]`
2. Claude processes the anonymised text
3. pseudonym-mcp restores originals in the response

Optional `lang` argument: `en` (default) or `pl`.

### `privacy_scan_file` — file or PDF

> Requires **macos-vision-mcp** to be installed alongside pseudonym-mcp.

```
/privacy_scan_file filePath="/Users/me/vault/contracts/nda.pdf" task="Summarize obligations and deadlines"
```

What happens:
1. macos-vision-mcp extracts text from the file on-device via Apple Vision
2. pseudonym-mcp masks detected PII before sending anything to the cloud
3. Claude processes the masked content
4. pseudonym-mcp restores originals before the response is shown

Optional arguments: `task` (default: _summarize the key points_), `lang` (`en` or `pl`).

---

## 🛡️ What Gets Detected

These are the patterns the masker looks for. Detection is best-effort — not a guarantee.

### 🇺🇸 English PII (`--lang en`, default)

| Token             | Covers                                                   | How |
| ----------------- | -------------------------------------------------------- | --- |
| `[PERSON:1]`      | 👤 Full names                                            | Ollama NER |
| `[SSN:1]`         | 🪪 Social Security Numbers — with area-number validation | Regex + validator |
| `[CREDIT_CARD:1]` | 💳 13–19 digit card numbers — with Luhn checksum         | Regex + validator |
| `[EMAIL:1]`       | 📧 Email addresses                                       | Regex |
| `[PHONE:1]`       | 📱 US phone formats                                      | Regex |
| `[ORG:1]`         | 🏢 Organisation names                                    | Ollama NER |

> 🌍 **International users:** `--lang pl` adds support for PESEL (national ID), Polish IBAN, and Polish phone formats.

**Known gaps worth knowing about:**
- Nicknames, initials, and partial names ("J.K.", "the doc")
- Indirect references ("my landlord", "the place near the bridge")
- Free-form addresses that aren't in a structured format
- Document numbers and case numbers not yet in the regex set
- Anything language-specific outside `en` and `pl`

If your threat model demands exhaustive detection, this stack is not enough on its own — pair it with manual review or stricter local-only models for the highest-sensitivity material.

---

## 🔒 What This Stack Actually Guarantees

Calibrated claims, not marketing ones:

- **📡 No telemetry from the tools themselves.** Neither package phones home. The only outbound calls are: (a) to your local Ollama instance, and (b) the cloud LLM call _you_ initiate.
- **💾 In-memory token mappings.** Mappings are not written to disk by default. Sessions expire when Claude Desktop restarts — intentional, so PII doesn't linger.
- **✈️ On-device OCR.** Apple Vision runs locally using Neural Engine acceleration. The image bytes don't leave your Mac during the OCR step.
- **📦 Minimal dependency footprint.** pseudonym-mcp has no third-party runtime dependencies — a deliberate decision for a tool handling sensitive data.
- **🔁 Idempotent tokens within a session.** The same value maps to the same token, preserving semantic coherence across multi-note queries (with the re-identification trade-off noted above).

What this stack **does not** guarantee:
- That all PII in your text is detected
- That tokenised text is unlinkable to real people
- That the cloud provider can't learn sensitive things from structure, timing, or content
- Compliance with any specific regulation — that's a system-level property, not a tool-level one

---

## ⚖️ Where This Fits in a Compliance Posture

Not "this makes you compliant." More like "this is a defensible technical control to point to."

| Regulation          | Where this helps                                                           | Where it doesn't                                                                 |
| ------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **🇺🇸 HIPAA**        | Reduces cleartext PHI in cloud transit; supports minimum-necessary principle | Pseudonymised PHI is still PHI; BAAs and full safeguards remain required        |
| **💳 PCI DSS**      | Card numbers masked before LLM transit (Luhn-validated detection)          | Doesn't replace network segmentation, logging, or scope-reduction obligations    |
| **🇺🇸 CCPA / CPRA**  | Demonstrates data minimisation toward third-party processors               | Doesn't change controller/processor obligations or consumer rights              |
| **🏢 SOC 2**        | Evidence of a technical control limiting PII exposure                      | One control among many; auditors will want the full picture                     |
| **🇪🇺 GDPR**         | Pseudonymisation is explicitly encouraged (Art. 25, Art. 32)               | Recital 26: pseudonymised data is still personal data; Art. 44 transfers still apply |

> ⚠️ **GDPR specifically:** pseudonymisation is recognised as a risk-reduction measure but does not exempt you from lawfulness, transparency, or transfer rules. Treat this as a control, not an exemption.

---

## 🌿 Why This Matters for Obsidian Users Specifically

Obsidian's core philosophy is _local-first_: your data lives on your device, in plain text, under your control. Every file is yours.

Most cloud AI plugins stretch that contract — they were designed when "send the whole note" was the path of least resistance. `macos-vision-mcp` + `pseudonym-mcp` are an attempt to bring a local-first sensibility to the cloud-LLM call itself: get the model quality, ship less raw PII upstream than you otherwise would.

It's not a perfect solution. It's a profilaktic layer worth building into a research or second-brain workflow. **Your second brain stays more yours than it would without it. 🧠🔒**

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

# Recommended: NER for names + organisations
ollama pull llama3
```

---

## 🔗 Links

- 🕵️ **pseudonym-mcp** — [npm](https://www.npmjs.com/package/pseudonym-mcp) · [GitHub](https://github.com/woladi/pseudonym-mcp)
- 📸 **macos-vision-mcp** — [GitHub](https://github.com/woladi/macos-vision-mcp)
- 📄 License: MIT — Adrian Wolczuk

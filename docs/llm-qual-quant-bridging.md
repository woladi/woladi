# LLM as a Bridge Between Qualitative and Quantitative Research in Psychology and Sociology

## 1. The Problem

Qualitative research — in-depth interviews, focus groups, ethnography, diary studies — produces rich material that is expensive to process and difficult to correlate with quantitative data (questionnaires, administrative records, panels). The classic path runs through manual coding in NVivo / Atlas.ti / MAXQDA, aggregation into indicators, and only then correlation with numbers. The bottleneck is time and inter-coder variance — Krippendorff's α often barely exceeds 0.7.

LLMs change this equation on two sides at once: coding becomes 100× cheaper and 1000× faster, but a new problem appears — the model as "coder" has to be validated, because it carries its own theoretical and cultural biases inherited from the training corpus.

## 2. LLM as a Classifier of Qualitative Data

The simplest application: *codebook → prompt*. A classic coding manual (categories, definitions, positive and negative examples) becomes the system prompt. Each transcript fragment comes back from the model as structured JSON:

```json
{
  "fragment_id": "P07-min12-15",
  "categories": ["emotional_support", "social_isolation"],
  "intensity": 0.7,
  "confidence": 0.85,
  "evidence_quote": "..."
}
```

Three techniques boost reliability:
- **Multiple judges** — the same fragment coded by Claude, GPT-4, and Gemini; agreement = confidence indicator.
- **Sample validation** — 10–15% of material coded in parallel by a human and the LLM, reporting Krippendorff's α for the human-model pair.
- **Active learning** — fragments with low consensus are routed to a human; the rest stay LLM-only.

## 3. LLM as a Transformer: Qual → Quant

Categorical classification is just the start. The stronger layer: **translating free-form text into quantitative scales**. Example: a patient speaks 200 words about their wellbeing in an interview. The LLM receives a prompt with a scale definition (PHQ-9, BDI-II) and returns a predicted score per item — with justification grounded in specific quotes. The result: a `respondent × scale item` matrix that can be compared 1:1 with the questionnaire filled in by the same respondent.

This opens three paths:
- **Cross-method validation** — does the LLM extract from the interview the same scores the respondent declared in the survey?
- **Latent constructs** — surfacing hidden constructs ("agency", "cultural capital") where the respondent does not name them explicitly.
- **Vector embeddings** — each interview as a vector in 1536-dimensional space; clustering yields a qualitative typology correlated with demographic groups.

## 4. Correlation with Quantitative Data — Three Levels

**Level A — Same respondent (within-subject).** Interview and questionnaire from the same participant. The LLM transforms the interview onto the same scale. Spearman's ρ between the two versions answers whether the person **says** the same thing they **declare**. Divergence reveals social desirability bias or semantic problems in the scale.

**Level B — Population aggregation.** LLM categories joined with ESS, EVS, or national social-diagnosis data. Question: does the frequency of "economic anxiety" in interviews correlate with regional unemployment? Classic triangulation, except the qualitative side is now scalably coded.

**Level C — Cross-prediction.** A model that predicts the qualitative outcome from quantitative data (and vice versa). Prediction error = a measure of the "independent information" carried by qualitative data — what the survey did not capture.

## 5. Pipeline

```
Interview audio
   ↓ Whisper (local)
Transcript
   ↓ pseudonymisation (PII → tokens)
Pseudonymised text
   ↓ LLM 1: category classifier
   ↓ LLM 2: scale extractor
JSON {respondent_id, categories[], scale_predictions{}}
   ↓ validation: 10% manual coding
Data matrix (CSV / parquet)
   ↓ R / Python / Stata
Correlations, regressions, SEM
```

## 6. Limitations

- **LLM bias is real.** A model trained mostly on English text understands "dignity" differently than a Polish ethnographer would. Cross-cultural research requires separate validation per language.
- **Constructs are born in the prompt.** What the model "finds" is a function of the definition we handed it. Pure grounded theory still needs a human.
- **Non-determinism.** The same fragment coded twice can yield different results. Required: `temperature = 0` plus multiple runs with explicit agreement reporting.
- **Ethics and privacy.** Interviews contain sensitive data. Pseudonymisation before sending to a cloud LLM (or local models in the `mistral-nemo` class) is mandatory; IRB approval covering AI processing too.
- **LLMs don't replace theory.** The classifier is only as good as the codebook; the codebook is only as good as the theory that produced it.

## 7. Three Concrete Applications

1. **Healthcare satisfaction** — patient interviews + SF-36. The LLM extracts SF-36 predictions from the interview; we correlate them with the actually-completed questionnaire. Divergence reveals areas the standard scale does not cover.
2. **Social capital in Poland** — narratives from 200 interviews coded along Putnam's dimensions (bonding / bridging / linking), correlated with the *Diagnoza Społeczna* survey at the voivodeship level.
3. **Cross-cultural psychology** — the same codebook applied to interviews in PL / EN / DE / JP, with explicit measurement of LLM variance across languages as a proxy for *measurement invariance*.

---

**Conclusion.** The LLM does not replace the qualitative researcher — it moves the bottleneck from coding to validation. Mixed-methods research becomes scalable as long as the researcher treats the LLM as seriously as any other measurement instrument: with control-sample validation, explicit variance documentation, and model-bias reporting.

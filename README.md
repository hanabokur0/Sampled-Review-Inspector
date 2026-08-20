# Sampled Review Inspector

**Adaptive document review by sampling first and expanding only when needed.**

Sampled Review Inspector is a lightweight, local-first proof of concept for reviewing long documents without sending the entire text to an API.

It extracts a few reproducible samples, generates a structured review prompt, and converts an LLM's JSON response into a simple gate:

**PASS / REVIEW / HOLD / REJECT**

> sample → inspect → anomaly → expand

## Why

Long-document review is often wasteful: every paragraph receives the same attention even though only a small part may contain the problem.

This project borrows the logic of sampling inspection from quality assurance:

1. Sample a few sections.
2. Inspect those sections.
3. If no meaningful anomaly appears, stop.
4. If an anomaly appears, expand around that location.

The goal is **not** to prove whether AI wrote a document. AI-writing signals are treated as one weak signal among several.

## What v0.1.0 does

- Runs entirely in one `index.html`
- No API key
- No server
- No upload
- No network requests
- Accepts pasted text or a local `.txt` / `.md` file
- Extracts three samples:
  - beginning
  - seeded random section
  - ending
- Default sample size: 3,000 characters each
- Stores the random seed and character offsets for reproducibility
- Generates a review prompt for ChatGPT, Claude, Gemini, a local model, or another LLM
- Accepts a JSON review result pasted back into the page
- Displays findings and a recommended gate
- Suggests an expansion range when deeper review is needed

## Quick start

1. Download or clone this repository.
2. Open `index.html` in a browser.
3. Paste a document or load a `.txt` / `.md` file.
4. Click **Sample document**.
5. Copy the generated review prompt into an LLM.
6. Ask the LLM to return JSON only.
7. Paste the JSON result into **Review JSON**.
8. Click **Apply review**.

No build step is required.

## Review flow

```text
Document
   ↓
Sampling
   ↓
Prompt generation
   ↓
Manual LLM review
   ↓
Structured JSON
   ↓
Gate
   ├─ PASS
   ├─ REVIEW
   ├─ HOLD
   └─ REJECT
        ↓
   Expand only when needed
```

## Inspection axes

The generated prompt asks the reviewer to inspect each sample for:

- logic
- evidence
- consistency
- redundancy
- specificity
- AI-writing signals

AI-writing suspicion **must not** cause rejection by itself.

## Gate semantics

### PASS
No material problem was found in the sampled text.

### REVIEW
A human should inspect one or more findings.

Examples:

- unsupported generalization
- weak evidence
- suspiciously uniform writing
- possible contradiction

### HOLD
The sampled text is insufficient to decide.

Examples:

- missing source material
- citation cannot be verified
- required context is outside the sample

**Unknown ≠ Failed.**

### REJECT
Use only when a serious issue is directly supported by the inspected material.

Examples:

- confirmed fabricated citation
- explicit numerical contradiction
- conclusion reverses the reported result

## Example review JSON

```json
{
  "document_status": "REVIEW",
  "summary": "The random sample contains an unsupported causal claim and requires source verification.",
  "samples": [
    {
      "sample_id": "sample-beginning",
      "recommended_status": "PASS",
      "findings": []
    },
    {
      "sample_id": "sample-random",
      "recommended_status": "REVIEW",
      "findings": [
        {
          "type": "unsupported_generalization",
          "severity": "medium",
          "quote": "This always causes a measurable increase in productivity.",
          "reason": "The claim is causal and universal, but the sample provides no supporting evidence.",
          "needs_expansion": true
        }
      ]
    },
    {
      "sample_id": "sample-ending",
      "recommended_status": "PASS",
      "findings": []
    }
  ],
  "ai_writing_signal": {
    "level": "low",
    "note": "Reference signal only; not a rejection criterion."
  },
  "next_action": {
    "type": "expand",
    "sample_id": "sample-random",
    "characters_before": 3000,
    "characters_after": 3000,
    "reason": "Inspect supporting context around the causal claim."
  }
}
```

See `examples/sample_review.json` for a complete example.

## Design principle

```text
generator != inspector != gate
```

The tool separates three concerns:

- **Generator**: whoever or whatever wrote the document
- **Inspector**: the LLM that observes possible issues
- **Gate**: the rule that maps findings to PASS / REVIEW / HOLD / REJECT

This separation makes it possible to change review criteria without changing the document generator.

## Reproducibility

The seeded random sample records:

- seed
- start offset
- end offset
- sample size

This allows a later reviewer to inspect the exact same slice.

## Privacy

v0.1.0 performs no network requests.

The document remains in the browser unless **you manually paste the generated prompt into an external LLM**. Before doing that, consider whether the sampled text contains confidential, personal, regulated, or unpublished information.

## Non-goals

This project does not claim to provide:

- reliable AI-authorship detection
- complete plagiarism detection
- complete academic misconduct detection
- truth verification of an entire paper
- replacement for expert peer review

The purpose is narrower:

> Find where deeper human or machine review is worth spending attention.

## Roadmap

### v0.1 — Manual LLM review
- local sampling
- seeded random sample
- prompt generation
- JSON import
- gate display

### v0.2 — Adaptive expansion
- one-click extraction around flagged samples
- repeated inspection rounds
- review history

### v0.3 — Citation inspection
- reference matching
- DOI / URL verification adapters
- citation-to-claim checks

### v0.4 — Cross-section consistency
- compare definitions
- compare numbers
- compare named entities
- compare conclusions

### v0.5 — Multiple inspectors
- logic inspector
- citation inspector
- consistency inspector
- statistical inspector
- AI-writing-signal inspector

## Repository structure

```text
sampled-review-inspector/
├─ README.md
├─ index.html
├─ LICENSE
├─ RELEASE_NOTES_v0.1.0.md
└─ examples/
   ├─ sample_document.txt
   └─ sample_review.json
```

## License

MIT License. See `LICENSE`.

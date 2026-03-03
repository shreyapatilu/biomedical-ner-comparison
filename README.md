# Biomedical NER: HuggingFace vs GPT-4

A hands-on comparison of two fundamentally different approaches to biomedical Named Entity Recognition (NER) on the same scientific paper — a fine-tuned clinical transformer vs GPT-4 zero-shot prompting.

---

## Overview

This project extracts biomedical entities from a tissue engineering research paper ([arXiv:2110.03526](https://arxiv.org/abs/2110.03526)) using two distinct NLP strategies and compares their outputs across a normalized entity schema.

| Model | Approach | Entities Extracted |
|---|---|---|
| `d4data/biomedical-ner-all` (HuggingFace) | Fine-tuned on clinical corpora | 288 |
| GPT-4 (OpenAI) | Zero-shot prompting | 427 |
| **Agreement (same entity + type)** | — | **17 (3.1%)** |

The 3.1% agreement rate is the central finding: these models were not built for the same task, and their disagreement reveals that more clearly than their individual outputs do.

---

## Pipeline

```
PDF (arXiv)
    │
    ▼
pdf2image → PIL Images
    │
    ▼
pytesseract OCR → Raw Text
    │
    ▼
Text Cleaning + NLTK Sentence Tokenization (~200 sentences)
    │
    ├──────────────────────────────────────┐
    ▼                                      ▼
HuggingFace NER                        GPT-4 Zero-shot
d4data/biomedical-ner-all              gpt-4 (temperature=0)
(runs locally, free)                   (API calls, paid)
    │                                      │
    ▼                                      ▼
288 entities                           427 entities
15 clinical types                      11 research types
    │                                      │
    └──────────────┬───────────────────────┘
                   ▼
          Normalization to common schema
          (Disease, Gene, Chemical, Anatomy,
           Symptom, Procedure, Lab_value, Other)
                   │
                   ▼
          Agreement analysis
          Overlap: 17 entities (3.1%)
```

---

## Key Results

### Entity type distribution after normalization

| Type | HuggingFace | GPT-4 | Difference |
|---|---|---|---|
| Procedure | 90 | 1 | -89 |
| Other | 119 | 287 | +168 |
| Gene/Protein | 0 | 69 | +69 |
| Chemical | 2 | 19 | +17 |
| Disease | 26 | 39 | +13 |
| Symptom | 20 | 0 | -20 |
| Lab_value | 10 | 0 | -10 |
| Anatomy | 21 | 12 | -9 |

### Interpretation

**HuggingFace** was fine-tuned on clinical documentation. It excels at procedural language, symptoms, and lab values — the kind of entities found in EHRs and clinical notes. It found zero gene/protein entities because those aren't what clinical NER models are trained to see.

**GPT-4** was trained on broad text including scientific literature. It picks up molecular biology terminology (genes, proteins, chemicals) that the clinical model misses entirely. But 287 entities landed in "Other" — the prompt taxonomy needs refinement for production use.

**Same text. Same paper. Different models saw different things.** Architecture decisions don't just affect accuracy — they determine what kinds of entities your pipeline is capable of finding.

---

## Requirements

```
python >= 3.9
pytesseract
pdf2image
transformers
torch
openai
nltk
pandas
```

System dependency: `tesseract-ocr` (install via `apt-get` on Linux/Colab)

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/shreyapatilu/biomedical-ner-comparison.git
cd biomedical-ner-comparison
```

### 2. Install dependencies

```bash
pip install pytesseract pdf2image transformers torch openai nltk pandas
```

On Ubuntu/Colab, also install the Tesseract binary:

```bash
sudo apt-get install tesseract-ocr
```

### 3. Set your OpenAI API key

Do not hard-code your API key. Use an environment variable:

```bash
export OPENAI_API_KEY="your-key-here"
```

In the notebook, replace `"YOUR_API_KEY_HERE"` with:

```python
import os
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
```

### 4. Open the notebook

Run locally with Jupyter:

```bash
jupyter notebook biomedical_NER_comparison_github.ipynb
```

Or open directly in [Google Colab](https://colab.research.google.com/) by uploading the `.ipynb` file.

---

## Project Structure

```
biomedical-ner-comparison/
├── biomedical_NER_comparison_github .ipynb   # Main notebook
└── README.md
```

---

## Practical Recommendations

| Use case | Recommended approach |
|---|---|
| Clinical NLP, adverse event detection, EHR processing | Fine-tuned clinical model (HuggingFace) |
| Drug discovery, literature mining, molecular biology | GPT-4 with validated prompts |
| Production systems requiring both coverage types | Hybrid pipeline |

---

## Background

This project was part of a personal self-learning series exploring NLP in healthcare. The full write-up is available on ([LinkedIn](https://www.linkedin.com/in/shreyapatilu-datascientist/)).





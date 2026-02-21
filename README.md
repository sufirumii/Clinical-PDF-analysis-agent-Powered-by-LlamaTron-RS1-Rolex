# Clinical PDF analysis — Powered by LlamaTron RS1 Rolex

Clinical PDF analysis agent with Agentic PubMed RAG, built on a fine-tuned Llama-3.2-1B medical reasoning model.

**GitHub:** [sufirumii/Clinical-PDF-analysis-agent-Powered-by-LlamaTron-RS1-Rolex](https://github.com/sufirumii/Clinical-PDF-analysis-agent-Powered-by-LlamaTron-RS1-Rolex)
**Model:** [Rumiii/LlamaTron-RS1-Rolex](https://huggingface.co/Rumiii/LlamaTron-RS1-Rolex)
**Paper:** [ReasonMed arXiv:2506.09513](https://arxiv.org/abs/2506.09513)

---

## Demo

<img width="1920" height="964" alt="ClinIQ Interface" src="https://github.com/user-attachments/assets/ce76ad14-b5c8-4b5a-9e79-7a13c43333c4" />

<img width="1156" height="1020" alt="Clinical Analysis Report" src="https://github.com/user-attachments/assets/87f2b237-b252-49db-8f37-9fc906a643e2" />

<img width="1117" height="1010" alt="Agentic PubMed RAG Output" src="https://github.com/user-attachments/assets/d57256d2-04eb-443f-ba2a-460e58ddbf33" />

---

## Overview

ClinIQ is a clinical AI research agent that combines a domain-specific fine-tuned language model with an agentic retrieval pipeline. Upload any clinical PDF report and ClinIQ extracts findings, applies Chain-of-Thought clinical reasoning, decides which findings require supporting literature, fetches real PubMed abstracts, and generates a structured analysis report — all in a single pipeline. A follow-up Q&A interface allows multi-turn questioning grounded in the uploaded report context.

This project is built entirely on LlamaTron RS1 Rolex, a custom fine-tuned model trained on ReasonMed — the largest publicly available medical reasoning dataset as of 2025.

---

## Model — LlamaTron RS1 Rolex

| Property | Detail |
|---|---|
| Base model | meta-llama/Llama-3.2-1B-Instruct |
| Fine-tuning method | LoRA (rank=8, alpha=16) |
| Training data | ReasonMed 370K (arXiv:2506.09513) |
| Format | GGUF FP16 |
| Hardware | NVIDIA H100 via JarvisLabs.ai |
| HuggingFace | [Rumiii/LlamaTron-RS1-Rolex](https://huggingface.co/Rumiii/LlamaTron-RS1-Rolex) |

LlamaTron RS1 Rolex exhibits observable Chain-of-Thought reasoning on clinical questions, trained across 3 epochs on 370K multi-agent generated medical reasoning examples.

---

## Features

### Clinical PDF Analysis
Uploads are parsed via PyMuPDF. Extracted text is passed directly to LlamaTron for analysis without any preprocessing loss.

### Chain-of-Thought Reasoning
LlamaTron RS1 Rolex analyzes each finding with step-by-step clinical reasoning, assigns a severity rating (CRITICAL / HIGH / MODERATE / LOW), and provides a clinical significance summary.

### Agentic PubMed RAG
A rule-based agentic decision layer evaluates each finding and decides whether PubMed evidence retrieval is warranted. Retrieval is not triggered blindly for every finding — the agent assesses clinical significance before querying the PubMed Entrez API. Relevant abstracts are fetched and cited directly alongside findings in the report.

### Follow-up Q&A
After the initial analysis, a multi-turn Q&A interface allows further questioning grounded in the uploaded report context. LlamaTron answers follow-up questions using the original PDF content as context.

### Structured Report Output
The final report includes an executive summary table with severity counts, detailed per-finding sections with reasoning and citations, and a disclaimer.

---

## Tech Stack

| Component | Technology |
|---|---|
| Language model | LlamaTron RS1 Rolex (GGUF FP16) |
| Inference engine | llama-cpp-python |
| PDF parsing | PyMuPDF (fitz) |
| Evidence retrieval | PubMed Entrez API (no key required) |
| Interface | Gradio |
| Tunneling (Kaggle) | localtunnel |

---

## Installation

```bash
pip install llama-cpp-python gradio pymupdf requests
npm install -g localtunnel
```

---

## Usage

### Cell 1 — Install
```python
!pip install llama-cpp-python gradio pymupdf requests -q
!npm install -g localtunnel -q
```

### Cell 2 — Run the full app code
Paste the contents of `cliniq_app.py` (everything before the launch block).

### Cell 3 — Launch with localtunnel
```python
import subprocess, time, os
import gradio as gr

gr.close_all()
os.environ["GRADIO_UPLOAD_PROGRESS"] = "false"

demo = build_ui()
demo.launch(
    server_name="0.0.0.0",
    server_port=7860,
    share=False,
    show_error=True,
    quiet=True,
)

time.sleep(3)

proc = subprocess.Popen(
    ["lt", "--port", "7860"],
    stdout=subprocess.PIPE, stderr=subprocess.PIPE
)

for line in proc.stdout:
    url = line.decode().strip()
    if "https://" in url:
        ip = __import__('requests').get('https://loca.lt/mytunnelpassword').text.strip()
        print(f"ClinIQ LIVE at: {url}")
        print(f"Password: {ip}")
        break
```

Alternatively, set `share=True` in the launch block to use Gradio's built-in public URL when network allows it.

---

## Pipeline Architecture

```
Clinical PDF
     |
     v
PyMuPDF Text Extraction
     |
     v
LlamaTron RS1 Rolex — CoT Analysis + Severity Rating
     |
     v
Agentic Decision Layer — Should PubMed be queried for this finding?
     |
  YES |                    NO |
     v                       v
PubMed Entrez API        Skip retrieval
Fetch abstracts
     |
     v
Structured Report + PubMed Citations
     |
     v
Follow-up Q&A (multi-turn, grounded in report context)
```

---

## Limitations

- 1B parameter model — reasoning depth lags behind 7B-70B scale models
- ReasonMed training is MCQ-oriented, which affects free-form report analysis output style
- Not suitable for scanned PDFs (no OCR support)
- PubMed retrieval depends on network availability
- No persistent memory between sessions

---

## Planned Improvements

- DPO / ORPO alignment on clinical report data
- Fine-tuning on larger base models (Llama-3.2-3B, Meditron-7B)
- Formal evaluation on MedQA, PubMedQA, MMLU-Clinical
- OCR support for scanned clinical documents
- Deployment on Hugging Face Spaces for permanent public access

---

## References

- ReasonMed Dataset: [arXiv:2506.09513](https://arxiv.org/abs/2506.09513)
- LlamaTron RS1 Rolex: [Rumiii/LlamaTron-RS1-Rolex](https://huggingface.co/Rumiii/LlamaTron-RS1-Rolex)
- Base model: [meta-llama/Llama-3.2-1B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct)
- PubMed Entrez API: [NCBI E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25497/)

---

## Disclaimer

ClinIQ is a research and educational prototype. It is not a medical device, diagnostic tool, or substitute for professional clinical judgment. All outputs must be reviewed by a qualified healthcare professional before any clinical use.

---

## License

Code: MIT
Model weights: Meta Llama 3.2 Community License + ReasonMed dataset terms

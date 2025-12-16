# OCR Benchmarks in VLMEvalKit

This document provides comprehensive documentation for all OCR-related benchmarks supported in VLMEvalKit.

## Overview

| Benchmark | Type | Sub-datasets | Primary Metrics | Source File |
|-----------|------|--------------|-----------------|-------------|
| **OCRBench** | VQA | 10 categories | Category Accuracy | `image_vqa.py` |
| **OCRBench_v2** | VQA | Enhanced | Accuracy | `utils/ocrbrnch_v2_eval.py` |
| **CC-OCR** | VQA | 40+ | Macro F1, Accuracy | `image_ccocr.py` |
| **OceanOCR** | QA | 5 types | BLEU, METEOR, F1 | `oceanocr.py` |
| **olmOCRBench** | QA | Single | Custom metrics | `olmOCRBench/` |
| **OmniDocBench** | QA | Multiple | Edit Distance, TEDS | `OmniDocBench/` |
| **OCR_Reasoning** | VQA | Single | Accuracy + LLM | `utils/ocr_reasoning.py` |

---

## 1. OCRBench

### Description
Standard OCR benchmark covering 10 categories of text recognition and understanding tasks.

### Categories
```python
OCRBench_score = {
    'Regular Text Recognition': 0,        # Standard printed text
    'Irregular Text Recognition': 0,      # Curved/rotated text
    'Artistic Text Recognition': 0,       # Stylized text
    'Handwriting Recognition': 0,         # Handwritten text
    'Digit String Recognition': 0,        # Number sequences
    'Non-Semantic Text Recognition': 0,   # Random character strings
    'Scene Text-centric VQA': 0,          # QA about scene text
    'Doc-oriented VQA': 0,                # Document QA
    'Key Information Extraction': 0,      # KIE tasks
    'Handwritten Mathematical Expression Recognition': 0  # Math formulas
}
```

### Evaluation Method
- **Matching**: Case-insensitive substring matching
- **Math Formulas**: Whitespace-normalized matching
- **Final Score**: Sum of all categories (max 1000) / 10 = normalized score

### Usage
```bash
python run.py --data OCRBench --model MODEL_NAME
```

### Output
- `{model}_OCRBench_score.json`: Detailed category scores

---

## 2. CC-OCR (Comprehensive Chinese OCR)

### Description
Large-scale OCR benchmark from Qwen team with 40+ sub-datasets covering multiple scenarios.

### Sub-dataset Categories

#### Document Parsing (DocParsing)
| Dataset | Language | Type | Samples |
|---------|----------|------|---------|
| DocPhotoChn | Chinese | Document Photo | 75 |
| DocPhotoEng | English | Document Photo | 75 |
| DocScanChn | Chinese | Document Scan | 75 |
| DocScanEng | English | Document Scan | 75 |
| TablePhotoChn | Chinese | Table Photo | 75 |
| TablePhotoEng | English | Table Photo | 75 |
| TableScanChn | Chinese | Table Scan | 75 |
| TableScanEng | English | Table Scan | 75 |
| MolecularHandwriting | - | Molecular Formula | 100 |
| FormulaHandwriting | - | Math Formula | 100 |

#### Key Information Extraction (KIE)
| Dataset | Type | Samples |
|---------|------|---------|
| Sroie2019Word | Receipt | 347 |
| Cord | Receipt | 100 |
| EphoieScut | Chinese Form | 311 |
| Poie | Product | 250 |
| ColdSibr | Open Category | 400 |
| ColdCell | Open Category | 600 |

#### Multi-Language OCR
Supports: Arabic, French, German, Italian, Japanese, Korean, Portuguese, Russian, Spanish, Vietnamese (150 samples each)

#### Multi-Scene OCR
- Document Text: CORD, FUNSD, IAM, Chinese Doc, Chinese Handwriting
- Scene Text: Hieragent, IC15, InverseText, TotalText, Chinese Scene
- UGC Text: UGC Laion, Chinese Dense, Chinese Vertical

### Evaluation Metrics
- **lan_ocr**: Multi-language OCR (Macro F1)
- **scene_ocr**: Scene text (Macro F1)
- **kie**: Key Information Extraction (Accuracy)
- **doc_parsing**: Document parsing (Score)
- **total**: Average of all categories

### Usage
```bash
# Full benchmark
python run.py --data CCOCR --model MODEL_NAME

# Specific sub-dataset
python run.py --data CCOCR_DocParsing_DocPhotoChn --model MODEL_NAME
```

---

## 3. OceanOCR

### Description
Multi-scene OCR benchmark covering document, scene, and handwritten text.

### Evaluation Types
| Type | Description |
|------|-------------|
| document_en | English document OCR |
| document_zh | Chinese document OCR |
| scene_text_rec | Scene text recognition |
| handwritten_en | English handwriting |
| handwritten_zh | Chinese handwriting |

### Metrics
- **BLEU**: Bilingual Evaluation Understudy
- **METEOR**: Metric for Evaluation of Translation with Explicit Ordering
- **F-measure**: Precision/Recall harmonic mean
- **Precision/Recall**: Token-level metrics
- **Edit Distance**: Normalized Levenshtein distance

### System Prompt
```python
system_prompt = "Can you pull all textual information from the image?"
```

### Usage
```bash
python run.py --data OceanOCRBench --model MODEL_NAME
```

---

## 4. olmOCRBench

### Description
Document-to-Markdown conversion benchmark.

### Task
Convert document images to structured Markdown with:
- Text blocks
- Mathematical expressions (LaTeX)
- Tables (Markdown format)

### System Prompt
```python
system_prompt = (
    "Please provide a natural, plain text representation of the document, "
    "formatted in Markdown. Skip any headers and footers. "
    "For ALL mathematical expressions, use LaTeX notation with "
    "\\( and \\) for inline equations and \\[ and \\] for display equations. "
    "Convert any tables into Markdown format."
)
```

### Usage
```bash
python run.py --data olmOCRBench --model MODEL_NAME
```

---

## 5. OmniDocBench

### Description
Comprehensive document understanding benchmark with fine-grained evaluation.

### Evaluation Components

#### End-to-End Evaluator
| Element | Metrics |
|---------|---------|
| text_block | Edit_dist, BLEU, METEOR |
| display_formula | Edit_dist, CDM |
| table | TEDS, Edit_dist |
| reading_order | Edit_dist |

#### Table Evaluator
- TEDS (Tree Edit Distance-based Similarity)
- Supports attributes: language, line type, span, equation, background, layout

### Output Breakdown
- `overall_EN`: English overall score
- `overall_CH`: Chinese overall score
- `text_block_*`: Text block metrics
- `display_formula_*`: Formula metrics
- `table_*`: Table metrics
- `reading_order_*`: Reading order metrics

### Usage
```bash
python run.py --data OmniDocBench --model MODEL_NAME
```

---

## 6. OCR_Reasoning

### Description
OCR combined with reasoning tasks, using LLM judge for evaluation.

### Evaluation Method
1. **Direct Matching**: Type-specific matching (integer, float, string, multi-choice)
2. **LLM Judge**: GPT-4 based reasoning score (1-10 scale)

### Judge Prompt
```python
judge_prompts = '''Please act as an impartial judge and evaluate the quality
of the response provided by an AI assistant to the user question displayed below.
Your evaluation should consider correctness and helpfulness...'''
```

### Task Types
- Multi-choice questions
- Integer answers
- Float answers
- String answers

### Metrics
- **Accuracy**: Direct answer matching
- **Reasoning Score**: LLM judge score (normalized to 0-1)

---

## Adding Custom OCR Benchmarks

### Template
```python
from vlmeval.dataset.image_base import ImageBaseDataset
from vlmeval.smp import *

class MyOCRBench(ImageBaseDataset):
    TYPE = 'VQA'  # or 'QA'
    MODALITY = 'IMAGE'

    DATASET_URL = {
        'MyOCRBench': 'https://path/to/dataset.tsv'
    }
    DATASET_MD5 = {
        'MyOCRBench': 'md5_hash'
    }

    system_prompt = "Your system prompt here"

    def build_prompt(self, line):
        image_path = self.dump_image(line)[0]
        return [
            dict(type='image', value=image_path),
            dict(type='text', value=self.system_prompt)
        ]

    def evaluate(self, eval_file, **judge_kwargs):
        # Custom evaluation logic
        data = load(eval_file)
        # ... compute metrics ...
        return results
```

### Registration
Add to `vlmeval/dataset/__init__.py`:
```python
from .my_ocr_bench import MyOCRBench

IMAGE_DATASET = [
    # ... existing datasets ...
    MyOCRBench,
]
```

---

## Quick Reference: Running OCR Benchmarks

```bash
# Standard OCR
python run.py --data OCRBench --model Qwen2-VL-7B-Instruct

# Comprehensive Chinese OCR
python run.py --data CCOCR --model InternVL2-8B

# Document Understanding
python run.py --data OmniDocBench --model MiniCPM-V-2_6

# Multi-scene OCR
python run.py --data OceanOCRBench --model llava-onevision-qwen2-7b-ov

# Document to Markdown
python run.py --data olmOCRBench --model GPT4V

# Multiple benchmarks
python run.py --data "OCRBench CCOCR OmniDocBench" --model MODEL_NAME
```

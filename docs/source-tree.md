# VLMEvalKit Source Tree

## Project Statistics

| Category | Count |
|----------|-------|
| API Modules | 29 |
| VLM Models | 75+ |
| Datasets | 78+ |
| Total Python Files | 200+ |

## Directory Structure

```
VLMEvalKit/
├── run.py                      # Main entry point for evaluation
├── setup.py                    # Package installation configuration
├── requirements.txt            # Core dependencies
├── README.md                   # Project documentation (English)
│
├── vlmeval/                    # Main package
│   ├── __init__.py            # Package initialization, version info
│   ├── config.py              # Model registry and configurations (~2000 lines)
│   ├── tools.py               # CLI tools (vlmutil)
│   ├── inference.py           # Image inference engine
│   ├── inference_mt.py        # Multi-turn inference
│   ├── inference_video.py     # Video inference engine
│   │
│   ├── api/                   # External API integrations (29 modules)
│   │   ├── __init__.py
│   │   ├── gpt.py            # OpenAI GPT-4V
│   │   ├── claude.py         # Anthropic Claude
│   │   ├── gemini.py         # Google Gemini
│   │   ├── qwen_vl_api.py    # Alibaba Qwen-VL
│   │   ├── glm_vision.py     # Zhipu GLM-4V
│   │   ├── hunyuan.py        # Tencent Hunyuan
│   │   ├── doubao_vl_api.py  # ByteDance Doubao
│   │   ├── lmdeploy.py       # LMDeploy acceleration
│   │   └── ...               # 20+ more API wrappers
│   │
│   ├── vlm/                   # Open-source VLM implementations (75+ models)
│   │   ├── __init__.py
│   │   ├── base.py           # BaseModel abstract class
│   │   │
│   │   ├── # LLaVA Family
│   │   ├── llava/            # LLaVA implementations
│   │   │   ├── llava.py
│   │   │   ├── llava_next.py
│   │   │   └── llava_onevision.py
│   │   │
│   │   ├── # InternVL Family
│   │   ├── internvl/
│   │   │   └── internvl_chat.py
│   │   │
│   │   ├── # Qwen-VL Family
│   │   ├── qwen_vl.py
│   │   ├── qwen2_vl/
│   │   │   └── model.py
│   │   ├── qwen3_vl/
│   │   │   └── model.py
│   │   │
│   │   ├── # MiniCPM Family
│   │   ├── minicpm_v.py      # MiniCPM-V series
│   │   │
│   │   ├── # CogVLM Family
│   │   ├── cogvlm.py
│   │   │
│   │   ├── # Video LLMs
│   │   ├── video_llm/
│   │   │   ├── video_llava.py
│   │   │   ├── videochat2.py
│   │   │   └── configs/
│   │   │
│   │   └── ...               # 60+ more model implementations
│   │
│   ├── dataset/               # Benchmark datasets (78+ implementations)
│   │   ├── __init__.py       # Dataset registry and builders
│   │   ├── image_base.py     # Base class for image datasets
│   │   ├── video_base.py     # Base class for video datasets
│   │   │
│   │   ├── # Core VQA Datasets
│   │   ├── image_vqa.py      # OCRVQA, TextVQA, DocVQA, ChartQA
│   │   ├── image_mcq.py      # MMBench, MMMU, AI2D, etc.
│   │   ├── image_yorn.py     # Yes/No QA datasets
│   │   │
│   │   ├── # OCR-Specific Benchmarks
│   │   ├── image_ccocr.py    # CC-OCR (40+ sub-datasets)
│   │   ├── oceanocr.py       # OceanOCR benchmark
│   │   ├── olmOCRBench/      # olmOCRBench
│   │   │   ├── olmocrbench.py
│   │   │   ├── evaluator.py
│   │   │   └── katex/
│   │   ├── OmniDocBench/     # Document understanding
│   │   │   ├── omnidocbench.py
│   │   │   ├── metrics.py
│   │   │   └── utils.py
│   │   │
│   │   ├── # Video Datasets
│   │   ├── mmbench_video.py
│   │   ├── videomme.py
│   │   ├── mvbench.py
│   │   │
│   │   ├── # Evaluation Utilities
│   │   └── utils/
│   │       ├── ocrbench.py           # OCRBench evaluator
│   │       ├── ocr_reasoning.py      # OCR reasoning evaluator
│   │       ├── ocrbrnch_v2_eval.py   # OCRBench v2 evaluator
│   │       ├── ccocr_evaluator/      # CC-OCR evaluators
│   │       │   ├── __init__.py
│   │       │   ├── ocr_evaluator.py
│   │       │   └── common.py
│   │       ├── Ocrbench_v2/          # OCRBench v2 metrics
│   │       ├── vqa_eval.py           # VQA evaluation
│   │       ├── multiple_choice.py    # MCQ evaluation
│   │       └── ...
│   │
│   ├── smp/                   # Shared utilities
│   │   ├── __init__.py
│   │   └── file.py           # File operations
│   │
│   └── utils/                 # General utilities
│       ├── __init__.py
│       └── result_transfer.py
│
├── scripts/                   # Helper scripts
│   └── ...
│
├── docs/                      # Documentation
│   ├── en/                   # English docs
│   │   ├── Quickstart.md
│   │   ├── ConfigSystem.md
│   │   ├── Development.md
│   │   └── Contributors.md
│   ├── zh-CN/                # Chinese docs
│   │   ├── README_zh-CN.md
│   │   ├── Quickstart.md
│   │   └── ...
│   └── ja/                   # Japanese docs
│       └── README_ja.md
│
├── requirements/              # Additional requirements
│   └── docs.txt              # Documentation dependencies
│
├── assets/                    # Static assets
│
└── .github/                   # CI/CD
    ├── workflows/
    │   ├── lint.yml          # Linting workflow
    │   └── pr-run-test.yml   # PR testing workflow
    └── scripts/
        └── assert_score.py   # Score assertion script
```

## Key Files Reference

### Entry Points
| File | Purpose |
|------|---------|
| `run.py` | Main evaluation script |
| `vlmeval/tools.py` | CLI tool (`vlmutil`) |
| `setup.py` | Package installation |

### Core Modules
| File | Lines | Purpose |
|------|-------|---------|
| `vlmeval/config.py` | ~2000 | Model registry, all supported models |
| `vlmeval/vlm/base.py` | ~220 | BaseModel abstract class |
| `vlmeval/dataset/__init__.py` | ~340 | Dataset registry and builders |
| `vlmeval/inference.py` | ~400 | Image inference engine |

### OCR-Related Files
| File | Purpose |
|------|---------|
| `vlmeval/dataset/image_ccocr.py` | CC-OCR dataset (40+ sub-datasets) |
| `vlmeval/dataset/oceanocr.py` | OceanOCR benchmark |
| `vlmeval/dataset/olmOCRBench/olmocrbench.py` | olmOCRBench |
| `vlmeval/dataset/OmniDocBench/omnidocbench.py` | Document understanding |
| `vlmeval/dataset/utils/ocrbench.py` | OCRBench evaluator |
| `vlmeval/dataset/utils/ocr_reasoning.py` | OCR reasoning evaluator |

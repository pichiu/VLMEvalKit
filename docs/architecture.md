# VLMEvalKit Architecture Overview

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERFACE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  run.py (CLI)                        vlmutil (CLI Tools)                    │
│  ├── --data DATASET                  ├── dlist    (list datasets)          │
│  ├── --model MODEL                   ├── mlist    (list models)            │
│  ├── --work-dir DIR                  ├── missing  (check missing)          │
│  └── --reuse                         └── eval     (evaluate file)          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            CONFIGURATION LAYER                               │
├─────────────────────────────────────────────────────────────────────────────┤
│  vlmeval/config.py                                                          │
│  ├── supported_VLM: Dict[str, Callable]     # Model name → Model factory    │
│  ├── api_models: Set[str]                   # API model names               │
│  ├── *_series: Dict[str, partial]           # Model family definitions      │
│  └── video_models: Dict[str, partial]       # Video model definitions       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────────┐
│     MODEL LAYER       │ │    DATASET LAYER      │ │   INFERENCE LAYER     │
├───────────────────────┤ ├───────────────────────┤ ├───────────────────────┤
│ vlmeval/vlm/          │ │ vlmeval/dataset/      │ │ vlmeval/inference.py  │
│ vlmeval/api/          │ │                       │ │ inference_mt.py       │
│                       │ │ build_dataset()       │ │ inference_video.py    │
│ BaseModel             │ │ ImageBaseDataset      │ │                       │
│ ├── generate()        │ │ ├── build_prompt()    │ │ infer_data_job()      │
│ ├── generate_inner()  │ │ ├── evaluate()        │ │ ├── load model        │
│ └── chat()            │ │ └── dump_image()      │ │ ├── batch inference   │
│                       │ │                       │ │ └── save results      │
└───────────────────────┘ └───────────────────────┘ └───────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            EVALUATION LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  vlmeval/dataset/utils/                                                     │
│  ├── vqa_eval.py          # VQA scoring (ANLS, relaxed accuracy)           │
│  ├── multiple_choice.py   # MCQ evaluation                                  │
│  ├── ocrbench.py          # OCRBench evaluation                            │
│  ├── ocr_reasoning.py     # OCR reasoning with LLM judge                   │
│  └── ccocr_evaluator/     # CC-OCR comprehensive evaluation                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              OUTPUT LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│  outputs/{model_name}/                                                      │
│  ├── {model}_{dataset}.xlsx         # Predictions                          │
│  ├── {model}_{dataset}_acc.csv      # Accuracy results                     │
│  └── {model}_{dataset}_score.json   # Detailed scores                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Core Class Hierarchy

### Model Classes

```
BaseModel (vlmeval/vlm/base.py)
│
├── API Models (vlmeval/api/)
│   ├── GPT4V           # OpenAI GPT-4 Vision
│   ├── Claude3V        # Anthropic Claude 3
│   ├── Gemini          # Google Gemini
│   ├── QwenVLAPI       # Alibaba Qwen-VL API
│   └── LMDeployAPI     # Local deployment API
│
└── Open-Source Models (vlmeval/vlm/)
    ├── LLaVA Family
    │   ├── LLaVA
    │   ├── LLaVA_Next
    │   └── LLaVA_OneVision
    │
    ├── InternVL Family
    │   └── InternVLChat
    │
    ├── Qwen-VL Family
    │   ├── QwenVLChat
    │   ├── Qwen2VLChat
    │   └── Qwen3VLChat
    │
    ├── MiniCPM Family
    │   ├── MiniCPM_V
    │   ├── MiniCPM_V_2_6
    │   └── MiniCPM_V_4
    │
    └── Video Models
        ├── VideoLLaVA
        ├── VideoChat2_HD
        └── PLLaVA
```

### Dataset Classes

```
ImageBaseDataset (vlmeval/dataset/image_base.py)
│
├── ImageVQADataset (image_vqa.py)
│   ├── OCRVQA_TEST, OCRVQA_TESTCORE
│   ├── TextVQA_VAL
│   ├── DocVQA_VAL, DocVQA_TEST
│   ├── InfoVQA_VAL, InfoVQA_TEST
│   ├── ChartQA_TEST
│   └── GQA_TestDev_Balanced
│
├── ImageMCQDataset (image_mcq.py)
│   ├── MMBench, MMBench_CN
│   ├── MMMU_DEV_VAL
│   ├── AI2D_TEST
│   ├── MMStar
│   └── ScienceQA
│
├── OCR-Specific Datasets
│   ├── OCRBench              # Standard OCR benchmark
│   ├── OCRBench_v2           # Enhanced OCR benchmark
│   ├── CCOCRDataset          # Comprehensive Chinese OCR
│   ├── OceanOCRBench         # Multi-scene OCR
│   ├── olmOCRBench           # Document to Markdown
│   ├── OmniDocBench          # Document understanding
│   └── OCR_Reasoning         # OCR + Reasoning
│
└── Video Datasets
    ├── VideoMME
    ├── MVBench
    └── MMBenchVideo
```

## Data Flow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Dataset   │───▶│   Prompt    │───▶│    Model    │───▶│  Evaluation │
│   Loading   │    │  Building   │    │  Inference  │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
      │                  │                  │                  │
      ▼                  ▼                  ▼                  ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ TSV/JSON    │    │ Image +     │    │ API call or │    │ Metrics:    │
│ from HF/    │    │ Question    │    │ Local model │    │ - Accuracy  │
│ ModelScope  │    │ + Options   │    │ generation  │    │ - ANLS      │
└─────────────┘    └─────────────┘    └─────────────┘    │ - F1        │
                                                         │ - BLEU      │
                                                         └─────────────┘
```

## Evaluation Metrics by Dataset Type

| Dataset Type | Primary Metrics | Implementation |
|-------------|-----------------|----------------|
| MCQ | Accuracy | `multiple_choice.py` |
| VQA | VQA Score, ANLS | `vqa_eval.py` |
| OCRBench | Category Accuracy | `ocrbench.py` |
| CC-OCR | Macro F1, Accuracy | `ccocr_evaluator/` |
| OceanOCR | BLEU, METEOR, F1 | `oceanocr.py` |
| OmniDocBench | Edit Distance, TEDS | `OmniDocBench/metrics.py` |
| OCR Reasoning | Accuracy + LLM Judge | `ocr_reasoning.py` |

## Extension Points

### Adding a New Model

1. Create model class in `vlmeval/vlm/` or `vlmeval/api/`
2. Inherit from `BaseModel`
3. Implement `generate_inner()` method
4. Register in `vlmeval/config.py`

```python
# vlmeval/vlm/my_model.py
class MyModel(BaseModel):
    def generate_inner(self, message, dataset=None):
        # Implementation
        return response

# vlmeval/config.py
my_model_series = {
    'MyModel-7B': partial(MyModel, model_path='path/to/model'),
}
```

### Adding a New Dataset

1. Create dataset class in `vlmeval/dataset/`
2. Inherit from `ImageBaseDataset` or `VideoBaseDataset`
3. Implement `build_prompt()` and `evaluate()` methods
4. Add to `__init__.py`

```python
# vlmeval/dataset/my_dataset.py
class MyDataset(ImageBaseDataset):
    TYPE = 'VQA'  # or 'MCQ'
    DATASET_URL = {'MyDataset': 'https://...'}

    def build_prompt(self, line):
        # Build prompt from data line
        pass

    def evaluate(self, eval_file, **judge_kwargs):
        # Evaluate predictions
        pass
```

## Distributed Execution

```
┌─────────────────────────────────────────────────────────────────┐
│                     Multi-GPU Inference                          │
├─────────────────────────────────────────────────────────────────┤
│  WORLD_SIZE=N  RANK=i  LOCAL_RANK=j                             │
│                                                                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│  │ GPU 0   │  │ GPU 1   │  │ GPU 2   │  │ GPU N-1 │            │
│  │ Rank 0  │  │ Rank 1  │  │ Rank 2  │  │ Rank N-1│            │
│  │         │  │         │  │         │  │         │            │
│  │ Data    │  │ Data    │  │ Data    │  │ Data    │            │
│  │ Shard 0 │  │ Shard 1 │  │ Shard 2 │  │ Shard N │            │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘            │
│       │            │            │            │                  │
│       └────────────┴────────────┴────────────┘                  │
│                          │                                       │
│                    ┌─────────────┐                              │
│                    │   Merge     │                              │
│                    │   Results   │                              │
│                    └─────────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

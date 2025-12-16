# VLMEvalKit 專案文件索引

> 自動生成於 2025-12-16 | VLMEvalKit v0.2rc1

## 專案概覽

**VLMEvalKit** 是一個用於評估視覺語言模型 (VLM/LVLM) 的開源工具套件，由 OpenCompass 團隊開發維護。

| 屬性 | 值 |
|------|-----|
| **專案類型** | Python Library |
| **主要語言** | Python 3.7+ |
| **核心框架** | PyTorch, transformers |
| **支援模型數** | 200+ |
| **支援基準數** | 80+ |

## 文件目錄

### 生成的專案文件

| 文件 | 描述 |
|------|------|
| [source-tree.md](./source-tree.md) | 完整源碼結構樹與文件統計 |
| [architecture.md](./architecture.md) | 系統架構圖與核心類別說明 |
| [ocr-benchmarks.md](./ocr-benchmarks.md) | OCR 相關基準測試詳細文件 |

### 原有文件

| 路徑 | 描述 |
|------|------|
| [en/Quickstart.md](./en/Quickstart.md) | 英文快速入門指南 |
| [en/ConfigSystem.md](./en/ConfigSystem.md) | 配置系統說明 |
| [en/Development.md](./en/Development.md) | 開發者指南 |
| [en/Contributors.md](./en/Contributors.md) | 貢獻者列表 |
| [zh-CN/README_zh-CN.md](./zh-CN/README_zh-CN.md) | 簡體中文說明 |
| [zh-CN/Quickstart.md](./zh-CN/Quickstart.md) | 中文快速入門 |
| [ja/README_ja.md](./ja/README_ja.md) | 日文說明 |

## 快速開始

### 安裝

```bash
# 從 PyPI 安裝
pip install vlmeval

# 從源碼安裝
git clone https://github.com/open-compass/VLMEvalKit.git
cd VLMEvalKit
pip install -e .
```

### 基本用法

```bash
# 評估單一模型和資料集
python run.py --data MMBench --model GPT4V

# 評估多個資料集
python run.py --data "OCRBench CCOCR" --model Qwen2-VL-7B-Instruct

# 使用 CLI 工具
vlmutil dlist l1        # 列出 L1 級別資料集
vlmutil mlist api       # 列出 API 模型
```

## 核心模組

### 模型層 (`vlmeval/vlm/`, `vlmeval/api/`)

支援的模型系列：
- **API 模型**: GPT-4V, Claude 3, Gemini, Qwen-VL API 等
- **LLaVA 系列**: LLaVA, LLaVA-Next, LLaVA-OneVision
- **InternVL 系列**: InternVL 1.5-3
- **Qwen-VL 系列**: Qwen-VL, Qwen2-VL, Qwen3-VL
- **MiniCPM 系列**: MiniCPM-V 2.6, MiniCPM-o, MiniCPM-V 4
- **視頻模型**: VideoLLaVA, VideoChat2, PLLaVA

### 資料集層 (`vlmeval/dataset/`)

支援的基準類型：
- **通用 VQA**: OCRVQA, TextVQA, DocVQA, ChartQA
- **多選題**: MMBench, MMMU, AI2D, MMStar
- **OCR 專項**: OCRBench, CC-OCR, OmniDocBench, OceanOCR
- **視頻理解**: VideoMME, MVBench, MMBench-Video

### 評估層 (`vlmeval/dataset/utils/`)

支援的評估方法：
- 準確率 (Accuracy)
- ANLS (Average Normalized Levenshtein Similarity)
- VQA Score
- BLEU, METEOR, F1
- TEDS (Tree Edit Distance Similarity)
- LLM Judge

## OCR Benchmark 重點

本專案特別關注 OCR 相關的評估基準，詳見 [ocr-benchmarks.md](./ocr-benchmarks.md)。

### 支援的 OCR 基準

| 基準 | 類型 | 子資料集數 |
|------|------|-----------|
| OCRBench | 標準 OCR | 10 類別 |
| CC-OCR | 綜合中文 OCR | 40+ |
| OceanOCR | 多場景 OCR | 5 |
| OmniDocBench | 文檔理解 | 多組件 |
| olmOCRBench | 文檔轉 Markdown | 1 |

### 快速評估 OCR

```bash
# 標準 OCR 基準
python run.py --data OCRBench --model MODEL_NAME

# 綜合 OCR 評估
python run.py --data "OCRBench CCOCR OmniDocBench" --model MODEL_NAME
```

## 專案統計

| 類別 | 數量 |
|------|------|
| API 模組 | 29 |
| VLM 模型實作 | 75+ |
| 資料集實作 | 78+ |
| Python 源文件 | 200+ |

## 相關連結

- **GitHub**: https://github.com/open-compass/VLMEvalKit
- **論文**: [VLMEvalKit: A Toolkit for Evaluating Large Vision-Language Models](https://arxiv.org/abs/2407.09129)
- **OpenCompass**: https://opencompass.org.cn/

---

*此索引由 Document Project Workflow 自動生成*

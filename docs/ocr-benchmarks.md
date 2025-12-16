# VLMEvalKit OCR 基準測試文件

本文件提供 VLMEvalKit 中所有 OCR 相關基準測試的完整說明。

## 總覽

| 基準測試 | 類型 | 子資料集 | 主要指標 | 源碼檔案 |
|----------|------|----------|----------|----------|
| **OCRBench** | VQA | 10 類別 | 類別準確率 | `image_vqa.py` |
| **OCRBench_v2** | VQA | 增強版 | 準確率 | `utils/ocrbrnch_v2_eval.py` |
| **CC-OCR** | VQA | 40+ | Macro F1, 準確率 | `image_ccocr.py` |
| **OceanOCR** | QA | 5 類型 | BLEU, METEOR, F1 | `oceanocr.py` |
| **olmOCRBench** | QA | 單一 | 自訂指標 | `olmOCRBench/` |
| **OmniDocBench** | QA | 多組件 | Edit Distance, TEDS | `OmniDocBench/` |
| **OCR_Reasoning** | VQA | 單一 | 準確率 + LLM | `utils/ocr_reasoning.py` |

---

## 評估指標詳解

在深入了解各基準測試之前，先介紹常用的評估指標：

### Edit Distance（編輯距離）

**定義**：又稱 Levenshtein Distance，計算將一個字串轉換成另一個字串所需的最少單字元編輯操作次數。

**操作類型**：
- 插入 (Insert)
- 刪除 (Delete)
- 替換 (Replace)

**計算公式**：
```
Normalized Edit Distance = Edit Distance / max(len(pred), len(gt))
```

**範例**：
```
預測: "kitten"
標準: "sitting"
編輯距離: 3 (k→s, e→i, 插入g)
正規化編輯距離: 3/7 ≈ 0.43
```

**應用場景**：文字識別、文檔解析、閱讀順序評估

---

### BLEU（Bilingual Evaluation Understudy）

**定義**：最初用於機器翻譯評估，衡量預測文本與參考文本之間的 n-gram 重疊程度。

**計算方式**：
1. 計算 1-gram 到 4-gram 的精確度
2. 加入簡短懲罰 (Brevity Penalty) 避免過短的預測獲得高分
3. 取幾何平均

**公式**：
```
BLEU = BP × exp(Σ wₙ log pₙ)

其中：
- BP = min(1, exp(1 - ref_len/pred_len))  # 簡短懲罰
- pₙ = n-gram 精確度
- wₙ = 權重（通常為 1/4）
```

**分數範圍**：0-1（或 0-100%），越高越好

**特點**：
- 對詞序敏感
- 更注重精確度 (Precision)
- 適合評估較長文本

---

### METEOR（Metric for Evaluation of Translation with Explicit Ordering）

**定義**：改進版的翻譯評估指標，考慮同義詞、詞幹和詞序。

**計算步驟**：
1. **對齊**：找到預測與參考之間的最佳詞對應
2. **計算精確度和召回率**：
   - Precision = 匹配詞數 / 預測詞數
   - Recall = 匹配詞數 / 參考詞數
3. **計算 F-score**：加權調和平均（召回率權重較高）
4. **片段懲罰**：對不連續匹配進行懲罰

**公式**：
```
F_mean = (10 × P × R) / (R + 9P)
Penalty = 0.5 × (chunks / matches)³
METEOR = F_mean × (1 - Penalty)
```

**優點**：
- 考慮召回率，更全面
- 支援同義詞匹配
- 對詞序變化有一定容忍度

---

### F-measure / F1 Score（F1 分數）

**定義**：精確度 (Precision) 和召回率 (Recall) 的調和平均數。

**公式**：
```
Precision = TP / (TP + FP)  # 預測正確的比例
Recall = TP / (TP + FN)     # 實際正確被找到的比例
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**Macro F1**：
```
Macro F1 = (1/n) × Σ F1ᵢ  # 各類別 F1 的平均
```

**應用**：多語言 OCR、場景文字識別

---

### TEDS（Tree Edit Distance-based Similarity）

**定義**：專門用於表格結構評估的指標，基於樹編輯距離。

**原理**：
1. 將表格轉換為 HTML DOM 樹結構
2. 計算預測樹與標準樹之間的樹編輯距離
3. 正規化為相似度分數

**計算公式**：
```
TEDS = 1 - (TreeEditDistance(pred_tree, gt_tree) / max(|pred_tree|, |gt_tree|))
```

**評估維度**：
- 表格結構（行列數、合併單元格）
- 單元格內容
- 單元格位置

**分數範圍**：0-1（或 0-100%），越高越好

**應用場景**：表格識別、文檔結構解析

---

### CDM（Character-level Distance Metric）

**定義**：字元級別的距離度量，專門用於數學公式評估。

**特點**：
- 在字元級別進行比較
- 對 LaTeX 語法進行正規化處理
- 忽略空格和格式差異

**計算方式**：
```
1. 正規化預測和標準公式（移除多餘空格、統一符號）
2. 計算字元級別的編輯距離
3. 轉換為相似度分數
```

**應用場景**：手寫數學公式識別、科學文檔解析

---

### ANLS（Average Normalized Levenshtein Similarity）

**定義**：用於文檔 VQA 的標準評估指標。

**公式**：
```
NLS = 1 - NL(pred, gt)  # Normalized Levenshtein Similarity
    = 1 - (EditDistance / max(len(pred), len(gt)))

ANLS = mean(max(NLS(pred, gtᵢ)) for all questions)
```

**閾值**：通常設定 0.5，低於此值視為完全錯誤

**應用**：DocVQA, InfoVQA

---

### VQA Score

**定義**：視覺問答任務的標準評估指標，考慮多個標註者的答案。

**公式**：
```
VQA Score = min(1, count(pred in answers) / 3)
```

**說明**：如果預測答案與至少 3 位標註者的答案相符，得滿分

---

## 1. OCRBench

### 描述
標準 OCR 基準測試，涵蓋 10 個類別的文字識別與理解任務。

### 評估類別
```python
OCRBench_score = {
    'Regular Text Recognition': 0,        # 標準印刷體文字
    'Irregular Text Recognition': 0,      # 彎曲/旋轉文字
    'Artistic Text Recognition': 0,       # 藝術字體
    'Handwriting Recognition': 0,         # 手寫文字
    'Digit String Recognition': 0,        # 數字序列
    'Non-Semantic Text Recognition': 0,   # 無意義字串
    'Scene Text-centric VQA': 0,          # 場景文字問答
    'Doc-oriented VQA': 0,                # 文檔問答
    'Key Information Extraction': 0,      # 關鍵資訊提取
    'Handwritten Mathematical Expression Recognition': 0  # 手寫數學公式
}
```

### 評估方法
- **匹配方式**：不區分大小寫的子字串匹配
- **數學公式**：移除空格後匹配
- **最終分數**：所有類別總分（滿分 1000）/ 10 = 正規化分數

### 使用方式
```bash
python run.py --data OCRBench --model MODEL_NAME
```

### 輸出檔案
- `{model}_OCRBench_score.json`：各類別詳細分數

---

## 2. CC-OCR（綜合中文 OCR）

### 描述
由通義千問團隊開發的大規模 OCR 基準測試，包含 40+ 子資料集，涵蓋多種場景。

### 子資料集分類

#### 文檔解析 (DocParsing)
| 資料集 | 語言 | 類型 | 樣本數 |
|--------|------|------|--------|
| DocPhotoChn | 中文 | 文檔照片 | 75 |
| DocPhotoEng | 英文 | 文檔照片 | 75 |
| DocScanChn | 中文 | 文檔掃描 | 75 |
| DocScanEng | 英文 | 文檔掃描 | 75 |
| TablePhotoChn | 中文 | 表格照片 | 75 |
| TablePhotoEng | 英文 | 表格照片 | 75 |
| TableScanChn | 中文 | 表格掃描 | 75 |
| TableScanEng | 英文 | 表格掃描 | 75 |
| MolecularHandwriting | - | 分子式 | 100 |
| FormulaHandwriting | - | 數學公式 | 100 |

#### 關鍵資訊提取 (KIE)
| 資料集 | 類型 | 樣本數 |
|--------|------|--------|
| Sroie2019Word | 收據 | 347 |
| Cord | 收據 | 100 |
| EphoieScut | 中文表單 | 311 |
| Poie | 產品標籤 | 250 |
| ColdSibr | 開放類別 | 400 |
| ColdCell | 開放類別 | 600 |

#### 多語言 OCR
支援語言：阿拉伯文、法文、德文、義大利文、日文、韓文、葡萄牙文、俄文、西班牙文、越南文（各 150 樣本）

#### 多場景 OCR
- 文檔文字：CORD, FUNSD, IAM, 中文文檔, 中文手寫
- 場景文字：Hieragent, IC15, InverseText, TotalText, 中文場景
- UGC 文字：UGC Laion, 中文密集, 中文直書

### 評估指標
| 指標 | 說明 | 計算方式 |
|------|------|----------|
| **lan_ocr** | 多語言 OCR | Macro F1 |
| **scene_ocr** | 場景文字 | Macro F1 |
| **kie** | 關鍵資訊提取 | 準確率 |
| **doc_parsing** | 文檔解析 | 綜合分數 |
| **total** | 總分 | 各類別平均 |

### 使用方式
```bash
# 完整基準測試
python run.py --data CCOCR --model MODEL_NAME

# 特定子資料集
python run.py --data CCOCR_DocParsing_DocPhotoChn --model MODEL_NAME
```

---

## 3. OceanOCR

### 描述
多場景 OCR 基準測試，涵蓋文檔、場景和手寫文字。

### 評估類型
| 類型 | 說明 |
|------|------|
| document_en | 英文文檔 OCR |
| document_zh | 中文文檔 OCR |
| scene_text_rec | 場景文字識別 |
| handwritten_en | 英文手寫 |
| handwritten_zh | 中文手寫 |

### 使用的評估指標
| 指標 | 說明 | 範圍 |
|------|------|------|
| **BLEU** | N-gram 重疊度 | 0-1 |
| **METEOR** | 含同義詞的翻譯評估 | 0-1 |
| **F-measure** | 精確度與召回率調和平均 | 0-1 |
| **Precision** | 詞彙級精確度 | 0-1 |
| **Recall** | 詞彙級召回率 | 0-1 |
| **Edit Distance** | 正規化編輯距離 | 0-1 |

### 系統提示詞
```python
system_prompt = "Can you pull all textual information from the image?"
```

### 使用方式
```bash
python run.py --data OceanOCRBench --model MODEL_NAME
```

---

## 4. olmOCRBench

### 描述
文檔轉 Markdown 格式的基準測試。

### 任務內容
將文檔圖像轉換為結構化 Markdown，包含：
- 文字區塊
- 數學公式（LaTeX 格式）
- 表格（Markdown 格式）

### 系統提示詞
```python
system_prompt = (
    "Please provide a natural, plain text representation of the document, "
    "formatted in Markdown. Skip any headers and footers. "
    "For ALL mathematical expressions, use LaTeX notation with "
    "\\( and \\) for inline equations and \\[ and \\] for display equations. "
    "Convert any tables into Markdown format."
)
```

### 使用方式
```bash
python run.py --data olmOCRBench --model MODEL_NAME
```

---

## 5. OmniDocBench

### 描述
全面的文檔理解基準測試，提供細粒度評估。

### 評估組件

#### 端對端評估器
| 元素 | 評估指標 | 說明 |
|------|----------|------|
| text_block | Edit_dist, BLEU, METEOR | 文字區塊品質 |
| display_formula | Edit_dist, CDM | 數學公式識別 |
| table | TEDS, Edit_dist | 表格結構與內容 |
| reading_order | Edit_dist | 閱讀順序正確性 |

#### 表格評估器
- **TEDS**：樹編輯距離相似度
- 支援屬性：語言、線條類型、合併單元格、公式、背景、版面配置

### 輸出指標
| 指標 | 說明 |
|------|------|
| `overall_EN` | 英文整體分數 |
| `overall_CH` | 中文整體分數 |
| `text_block_*` | 文字區塊指標 |
| `display_formula_*` | 公式指標 |
| `table_*` | 表格指標 |
| `reading_order_*` | 閱讀順序指標 |

### 使用方式
```bash
python run.py --data OmniDocBench --model MODEL_NAME
```

---

## 6. OCR_Reasoning

### 描述
結合 OCR 與推理能力的基準測試，使用 LLM 作為評審。

### 評估方法
1. **直接匹配**：根據答案類型進行匹配
   - 整數答案
   - 浮點數答案
   - 字串答案
   - 選擇題
2. **LLM 評審**：使用 GPT-4 評估推理品質（1-10 分）

### 評審提示詞
```python
judge_prompts = '''Please act as an impartial judge and evaluate the quality
of the response provided by an AI assistant to the user question displayed below.
Your evaluation should consider correctness and helpfulness...'''
```

### 評估指標
| 指標 | 說明 |
|------|------|
| **Accuracy** | 直接答案匹配準確率 |
| **Reasoning Score** | LLM 評審分數（正規化至 0-1） |

---

## 新增自訂 OCR 基準測試

### 範本程式碼
```python
from vlmeval.dataset.image_base import ImageBaseDataset
from vlmeval.smp import *

class MyOCRBench(ImageBaseDataset):
    TYPE = 'VQA'  # 或 'QA'
    MODALITY = 'IMAGE'

    DATASET_URL = {
        'MyOCRBench': 'https://path/to/dataset.tsv'
    }
    DATASET_MD5 = {
        'MyOCRBench': 'md5_hash'
    }

    system_prompt = "您的系統提示詞"

    def build_prompt(self, line):
        image_path = self.dump_image(line)[0]
        return [
            dict(type='image', value=image_path),
            dict(type='text', value=self.system_prompt)
        ]

    def evaluate(self, eval_file, **judge_kwargs):
        # 自訂評估邏輯
        data = load(eval_file)
        # ... 計算指標 ...
        return results
```

### 註冊新資料集
在 `vlmeval/dataset/__init__.py` 中新增：
```python
from .my_ocr_bench import MyOCRBench

IMAGE_DATASET = [
    # ... 現有資料集 ...
    MyOCRBench,
]
```

---

## 快速參考：執行 OCR 基準測試

```bash
# 標準 OCR 測試
python run.py --data OCRBench --model Qwen2-VL-7B-Instruct

# 綜合中文 OCR
python run.py --data CCOCR --model InternVL2-8B

# 文檔理解
python run.py --data OmniDocBench --model MiniCPM-V-2_6

# 多場景 OCR
python run.py --data OceanOCRBench --model llava-onevision-qwen2-7b-ov

# 文檔轉 Markdown
python run.py --data olmOCRBench --model GPT4V

# 多個基準測試
python run.py --data "OCRBench CCOCR OmniDocBench" --model MODEL_NAME
```

---

## 指標選擇建議

| 任務類型 | 建議指標 | 原因 |
|----------|----------|------|
| 文字識別 | Edit Distance, F1 | 字元級精確比較 |
| 文檔問答 | ANLS, VQA Score | 考慮答案變體 |
| 表格識別 | TEDS | 保留結構資訊 |
| 公式識別 | CDM, Edit Distance | 符號級比較 |
| 長文本生成 | BLEU, METEOR | N-gram 重疊 + 語義 |
| 閱讀順序 | Edit Distance | 序列比較 |

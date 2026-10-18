# Lab 04 — Invoice to JSON: Canvas Submission

**Student:** Nursayat Tulen  
**Course:** CSS4007 Artificial Intelligence · Narxoz University  
**Assignment:** Lab 04 (Week 6)  
**Notebook File:** [lab04_invoice_to_json.ipynb](file:///c:/Users/Laptop_kz/Desktop/Midterm%201/lab04_invoice_to_json.ipynb)

---

### 1. Predictions (Predict 1 and Predict 2) and Measurement Results

- **Predict 1:**
  - *My Prediction:* I predicted that the VLM would make the most mistakes on Kazakh (`kk`), especially in text/name fields (`seller_name`, `buyer_name`, `name`) because of rare Kazakh-specific letters. I also predicted that classic OCR would lose much more accuracy on degraded images than the VLM, and guessed around 90–95% for OCR and 85–90% for VLM on clean images.
  - *What the measurement showed:* On clean images, Kazakh indeed had the lowest accuracy (77.0% vs 90.8% Russian and 97.4% English) entirely due to names, while on degraded images classic OCR broke down almost completely (0.7%) compared to the VLM (44.7%).
- **Predict 2:**
  - *My Prediction:* On clean images, I predicted that the OCR upper bound would recover slightly more fields (or about the same) as the VLM because of sharp text and generous substring matching, but on degraded images the VLM would recover far more fields than OCR.
  - *What the measurement showed:* On clean scans OCR upper bound reached 93.4% [0.91, 0.95] vs VLM 88.4% [0.85, 0.91] with overlapping confidence intervals, while on degraded phone photos OCR collapsed to 0.7% [0.00, 0.02] whereas VLM maintained 44.7% [0.40, 0.49].

---

### 2. Accuracy Table (Part 4 — All Languages Combined)

| Method | Clean Images (95% Wilson Interval) | Degraded Images (95% Wilson Interval) |
| :--- | :---: | :---: |
| **OCR upper bound (Tesseract, psm=4)** | **93.4%** (426/456) &nbsp; `[0.91, 0.95]` | **0.7%** (3/456) &nbsp; `[0.00, 0.02]` |
| **VLM normalised (`Qwen2-VL-2B`)** | **88.4%** (403/456) &nbsp; `[0.85, 0.91]` | **44.7%** (204/456) &nbsp; `[0.40, 0.49]` |

*Note: On clean images, the 95% Wilson intervals of OCR and VLM overlap at 0.91, meaning their performance difference is not statistically significant; on degraded images, OCR collapses completely while the VLM retains almost half the fields.*

---

### 3. Kazakh Letter Confusions

**Three most frequent letter confusions on Kazakh invoices:**
1. **`қ` (U+049B) → `к` (U+043A)** &nbsp; (15 occurrences)
2. **`ұ` (U+04B1) → `у` (U+0443)** &nbsp; (11 occurrences)
3. **`і` (U+0456) → `и` (U+0438)** &nbsp; (10 occurrences)
*(and `ү` U+04AF → `у` U+0443 occurred 6 times)*

- **Reading problem vs. Knowledge-of-the-language problem:**  
  This is a **knowledge-of-the-language problem** (language model prior), because the vision encoder perceives the glyph shapes, but the language decoder substitutes Kazakh-specific characters with high-frequency Russian Cyrillic characters that dominate its training data.
- **Experiment to tell the two apart:**  
  Pass clean, large-font synthetic images of isolated characters or artificial non-words (pseudowords like "ақба" or "тұма") without language context into the model: if the model reads the Kazakh letters correctly in isolation but replaces them when reading words, the failure comes from language priors, not visual reading.

---

### 4. Arithmetic Checks

- **Wrong totals caught:** **8 out of 8** (100% of wrong totals caught).
- **Right totals wrongly flagged:** **13 out of 16** (81% of right totals flagged as invalid due to small item-level misreads).
- **Would I let them approve a payment on their own?**  
  **No, I would not let them approve payments on their own.** Although the checks caught all wrong totals, 3 degraded documents passed the checks, yet none of them (0/3) had all fields correct; arithmetic cannot detect a misread seller name, wrong BIN, or balanced errors.

---

### 5. Recommended Production Pipeline

**One sentence:**  
For scanned invoices like these, I would ship a **VLM extraction pipeline with rule-based arithmetic validation** that routes flagged documents to human review, and I would put an **image quality pre-filter (to detect blur and tilt and reject low-quality phone photos)** in front of it.

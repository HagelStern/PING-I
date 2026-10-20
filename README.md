# PDF Text Cleaning Script V4 (Python)

This document explains the functionality of the `PDF-Textbereinigung V4` Python script, which is designed to clean and normalize text extracted from PDF documents that often contain formatting artifacts like hyphenation and line breaks in compound words.

## 🎯 Goal

The primary goal of the script is to transform raw, poorly formatted text (typically from Tika or PDF extraction) into clean, grammatically correct German text by intelligently joining compound words while maintaining a transparent review process.

## ⚙️ How it Works

The script operates in three main phases: Extraction, Processing, and Reporting.

### 1. Extraction (Input)
The script accepts input in two ways:
*   **PDF File:** If a PDF is provided, it uses an external Tika server (default: `http://localhost:9998/tika`) to extract the raw text content.
*   **Raw Text:** If you have already extracted text (e.g., from another tool), you can pass it directly using the `--text` flag.

### 2. Processing (Core Logic)
The core logic resides in the `clean` function. It identifies hyphenated or line-broken word pairs using a regular expression (e.g., `[WordA][gap][WordB]`). For every identified pair, it applies a decision-making process (`choose` function):

*   **Predefined List `KEEP` (Manual Check):** Certain phrases (e.g., 'tuk-tuk', 'levada-wanderung') are explicitly kept hyphenated, ensuring specific terms are preserved as they appear in the context.
*   **Predefined List `JOIN` (Automatic Join):** A list of known compounds (e.g., 'levada', 'cristiano') are automatically joined into single words.
*   **Word Frequency Analysis (Statistical):** For all other cases, the script uses the `wordfreq` library (based on Zipf's frequency). If the combined word has a high statistical frequency in the German language (score $\ge 3.0$ or $\ge 4.0$), it is automatically joined into a single word. This is the primary method for correcting unknown compound words.

### 3. Reporting (Output)
The script generates three distinct output files:

1.  **`roh.txt`**: The original, untouched raw text.
2.  **`bereinigt.txt`**: The final, cleaned version of the text.
3.  **`pruefbericht.csv`**: A detailed report for **every single decision** made by the script. This is crucial for auditing the automated changes.

## 🚀 Usage

The script is run from the command line:

```bash
python <script_name>.py <input_pdf> [options]
```

**Example Commands:**

*   **Process a PDF (requires Tika service running):**
    `python script_name.py input_document.pdf --output ./output_folder`
*   **Process raw text:**
    `python script_name.py --text ./raw_data.txt --output ./output_folder`

## ⚠️ Important Notes

*   **Review Required:** Always check the `pruefbericht.csv` before importing the `bereinigt.txt`, especially for entries marked as `pruefen_unveraendert` (unverified), as the frequency-based decisions are automated.
*   **Dependencies:** This script requires the `wordfreq` library:
    ```bash
    python -m pip install wordfreq
    ```
*   **Tika Server:** If processing PDFs, a Tika instance must be running and accessible at the configured URL.

## 📁 Code Overview

The script uses `argparse` for command-line handling, `csv` for report generation, `re` (regular expressions) for pattern matching, and `pathlib` for file system operations. It is designed to be robust and non-destructive to original files.
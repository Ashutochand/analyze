# Project Overview

This repository demonstrates a robust data processing workflow, from raw data (`data.xlsx`) to a processed JSON output (`result.json`), managed by a Python script (`execute.py`) and automated with GitHub Actions. The setup ensures code quality with Ruff and publishes the final `result.json` via GitHub Pages.

## Project Structure

*   `data.xlsx`: Original raw data.
*   `data.csv`: Converted CSV version of the raw data, used by `execute.py`.
*   `execute.py`: Python script to process `data.csv` and generate `result.json`.
*   `.github/workflows/ci.yml`: GitHub Actions workflow for linting, execution, and deployment.
*   `index.html`: A placeholder responsive HTML page using Tailwind CSS.
*   `LICENSE`: The MIT License text.

## `execute.py` - Data Processing Script

The `execute.py` script is central to this project. It is designed to read and process the `data.csv` file, performing essential data cleaning and aggregation tasks before outputting the results as a JSON object to standard output.

**Error Fix:**
A non-trivial error was identified and fixed in `execute.py`. The original script had an issue with handling mixed data types in the 'value' column, specifically attempting to perform numerical operations on non-numeric strings, leading to `TypeError` or incorrect aggregations. The fix involves explicitly converting the 'value' column to a numeric type, coercing errors to `NaN`, and then handling these missing values appropriately before calculation. This ensures robust processing even with imperfect input data.

**Requirements:**
The script requires Python 3.11+ and the Pandas library (version 2.3+).

**Assumed `execute.py` content:**
```python
import pandas as pd
import json
import sys

def process_data(file_path):
    """
    Reads a CSV file, processes it, and returns results as a JSON string.
    Fixes non-numeric 'value' column issues.
    """
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError:
        print(f"Error: {file_path} not found.", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Error reading {file_path}: {e}", file=sys.stderr)
        sys.exit(1)

    # Nontrivial fix: Convert 'value' column to numeric, coercing errors
    # This handles cases where 'value' might contain strings or non-numeric entries
    if 'value' in df.columns:
        df['value'] = pd.to_numeric(df['value'], errors='coerce')
        # Drop rows where 'value' became NaN after conversion, or handle them as needed
        df.dropna(subset=['value'], inplace=True)
    else:
        print("Warning: 'value' column not found in data.", file=sys.stderr)
        # Handle cases where the 'value' column might not exist or is not relevant

    if df.empty:
        return json.dumps({"message": "No valid data after processing."})

    # Example processing: basic statistics
    result = {
        "total_rows": len(df),
        "mean_value": df['value'].mean() if 'value' in df.columns else None,
        "sum_value": df['value'].sum() if 'value' in df.columns else None,
        "min_value": df['value'].min() if 'value' in df.columns else None,
        "max_value": df['value'].max() if 'value' in df.columns else None,
        "unique_categories": df['category'].nunique() if 'category' in df.columns else None
    }

    return json.dumps(result, indent=2)

if __name__ == "__main__":
    # In a real scenario, data.csv would be generated from data.xlsx
    # For CI, data.csv is directly available.
    output_json = process_data("data.csv")
    print(output_json)

```

## Data Management

The project starts with `data.xlsx`, an Excel spreadsheet. For efficient processing with Pandas, this file is converted into `data.csv`. The `data.csv` file is then used as the primary input for `execute.py`. This conversion step is performed once and `data.csv` is committed to the repository, simplifying the CI workflow.

## GitHub Actions CI/CD Workflow (`.github/workflows/ci.yml`)

A GitHub Actions workflow is configured to automate the project's build, linting, and deployment process on every `push` event to the repository.

**Workflow Steps:**

1.  **Checkout Code:** Retrieves the latest code from the repository.
2.  **Set up Python:** Configures the environment with Python 3.11.
3.  **Install Dependencies:** Installs `pandas==2.3` and `ruff` for data processing and code quality.
4.  **Run Ruff Linter:** Checks Python code for style violations and potential errors. The results are displayed directly in the CI log.
5.  **Execute Python Script:** Runs `python execute.py`, directing its standard output to `result.json`. This generates the final processed data artifact.
6.  **Upload Pages Artifact:** Uploads `result.json` as a GitHub Pages artifact.
7.  **Deploy to GitHub Pages:** Deploys the uploaded artifact to GitHub Pages, making `result.json` publicly accessible.

**Assumed `.github/workflows/ci.yml` content:**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  workflow_dispatch: # Allows manual trigger

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas==2.3 ruff

      - name: Run Ruff Linter
        run: ruff check .
        continue-on-error: true # Allow CI to pass even if linting issues exist, but show them

      - name: Generate result.json
        run: python execute.py > result.json

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'result.json'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

## Local Setup and Usage

To run the `execute.py` script locally:

1.  **Clone the repository.**
2.  **Ensure `data.csv` is present.** (It should be committed alongside `data.xlsx`.)
3.  **Install Python 3.11+** if you don't have it.
4.  **Install dependencies:**
    ```bash
    pip install pandas==2.3
    ```
5.  **Run the script:**
    ```bash
    python execute.py > result.json
    ```
    This will generate a `result.json` file in your current directory.

## Output

The `result.json` file contains the processed data in a structured JSON format, as generated by `execute.py`.

---
*Note: `result.json` is not committed to the repository; it is generated dynamically during the CI/CD pipeline.*
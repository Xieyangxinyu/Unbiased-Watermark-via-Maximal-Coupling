# Data Dictionary

## File Structure
```
data/
├── finance_qa.json
├── finance_qa_llama.json
├── finance_qa_llama.jsonl
├── finance_qa_phi.json
├── finance_qa_phi.jsonl
├── longform_qa.json
├── longform_qa_llama.json
├── longform_qa_llama.jsonl
├── longform_qa_phi.json
└── longform_qa_phi.jsonl
```

## File Descriptions

### Original Dataset Files
* **finance_qa.json** - Original finance-related question-answer prompts from the WaterBench dataset. Contains financial queries and expected responses by the original data creators.
* **longform_qa.json** - Original long-form question-answer prompts from the WaterBench dataset. Contains queries requiring detailed, extended responses by the original data creators.

### Model Response Files
* **finance_qa_llama.json** - Responses generated by the LLaMA model for finance-related questions without watermarking applied.
* **finance_qa_phi.json** - Responses generated by the Phi model for finance-related questions without watermarking applied.
* **longform_qa_llama.json** - Responses generated by the LLaMA model for long-form questions without watermarking applied.
* **longform_qa_phi.json** - Responses generated by the Phi model for long-form questions without watermarking applied.

### Intermediate Files
* **finance_qa_llama.jsonl** - Intermediate line-delimited JSON file created during the response generation process for LLaMA model on finance questions.
* **finance_qa_phi.jsonl** - Intermediate line-delimited JSON file created during the response generation process for Phi model on finance questions.
* **longform_qa_llama.jsonl** - Intermediate line-delimited JSON file created during the response generation process for LLaMA model on long-form questions.
* **longform_qa_phi.jsonl** - Intermediate line-delimited JSON file created during the response generation process for Phi model on long-form questions.

## Data Generation Process
1. Original prompt datasets (`finance_qa.json` and `longform_qa.json`) are obtained from the WaterBench repository at [https://github.com/THU-KEG/WaterBench](https://github.com/THU-KEG/WaterBench).
2. The `original_response.py` script is run to generate model responses without watermarking for both LLaMA and Phi models.
3. During this process, intermediate JSONL files are created and then converted to final JSON response files.

## Usage with Watermarking Analysis

The dataset is used with the `main_watermark.py` script to analyze text quality distortion caused by watermarking techniques. Example usage:

```bash
python main_watermark.py \
        --prompt_path data/finance_qa_llama.json \
        --model_name llama \
        --method coupling \
        --output_dir output/finance_qa/llama/coupling/ngram_4 \
        --ngram 4 \
        --scoring_method v2 \
        --batch_size 64
```

--prompt_path is the path to the JSON file containing model responses to analyze (e.g. `finance_qa_llama.json`).
The usage of other command-line arguments is detailed in the README of the repository.
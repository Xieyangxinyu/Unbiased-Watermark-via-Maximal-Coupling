# Debiasing Watermarks for Large Language Models via Maximal Coupling

This repository reproduces the results reported in [Debiasing Watermarks for Large Language Models via Maximal Coupling](https://arxiv.org/abs/2411.11203), accepted at Journal of American Statistical Association (JASA) Special Issue on Statistical Science in AI. The following content guides you through the process of reproducing the results in the paper using Python.


# Table of Contents
- [Online Appendix to the Paper](/supplementary.pdf)
- [data folder](/data/)
- [Reproducing Results in Python](#reproducing-results-in-the-paper)
  - [Requirements](#requirements)
  - [Models](#models)
  - [Data](#data)
  - [Usage](#usage)


# Reproducing Results in the Paper

## Requirements
First, install [PyTorch](https://pytorch.org/get-started/locally/). The remaining dependency can be installed using the following command:
```
pip install -r requirements.txt
```
This codebase was adapted from [three_bricks](https://github.com/facebookresearch/three_bricks) and developed with python version 3.11, PyTorch version 2.1.1, CUDA 12.0. However, we do not support multi-bit watermarking in our code.

All experiments were run on NVIDIA A100-SXM4 GPUs with 40GB of VRAM on [Polaris Compute Nodes](https://docs.alcf.anl.gov/polaris/running-jobs/using-gpus/) at Argonne National Laboratory. The code is designed to be run on a single GPU, but it can be adapted for multi-GPU setups if needed.

## Models

We use two instruction fine-tuned models: [phi-3](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct) and [Llama-3.2](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct). In all our codes, `{model_name}` are either `phi` or `llama`, referring to these two models.

## Data
We adapted two long-form question answering datasets from [WaterBench](https://arxiv.org/abs/2311.07138). The data files are under `data/`.
The original responses from the models are generated by running `original_response.py`, for example,
```
python original_response.py --model_name llama --batch_size 32
``` 
and saved in files of the form `{dataset}_qa_{model_name}.json`, for example, `finance_qa_llama.json`. 

## Usage
You can reproduce the results of the paper by the following steps:
1. Run the `main_watermark.py` file with the appropriate command-line arguments.
<details>
<summary><span style="font-weight: bold;">Command Line Arguments for main_watermark.py</span></summary>
    
- `--model_name`: The name of the pre-trained model to use for text generation and analysis. Supported model names include "phi" and "llama".
- `--prompt_path`: The path to the JSON file containing prompts. Default value: "data/alpaca_data.json."
- `--method`: Choose a watermarking method for text generation. phiions: "none" (no watermarking), "openai" (Aaronson et al.), "maryland" ([Kirchenbauer et al.](https://arxiv.org/abs/2301.10226)), "dipmark" ([Wu et al.](https://arxiv.org/abs/2310.07710)), "coupling" which is our method. Default value: "none."
- `--method_detect`: Choose a statistical test to detect watermark. "same" uses the grounded statistical test with the same method as for generation. The other phiions are "openai", "maryland", "coupling-max", "coupling". Default value: "same." For our proposed method, please use "coupling".
- `--one_list`: Use only a single green list; only works if detection method is coupling. See details in the Appendix of the paper.
- `--scoring_method`: Method for scoring tokens. phiions: "none" (score every token), "v1" (score token when the watermark context is unique), "v2" (score token when {wm context + token} is unique). Default value: "none." We use "v2" for all our experiments.
- `--ngram`: Watermark context width for RNG key generation. Default value: 4. We use either 2 or 4 for all our experiments.
- `--gamma`: Size of the green lists.

</details>

Here is an example:
```cmd
python main_watermark.py \
        --prompt_path data/finance_qa_llama.json \
        --model_name llama \
        --method coupling \
        --output_dir output/finance_qa/llama/coupling/ngram_4 \
        --ngram 4 \
        --scoring_method v2 \
        --batch_size 64
```

2. The previous script generates watermarked text and saves the results in the specified output directory. The output files include:

**`results.jsonl`.** Contains the generated watermarked text for each prompt in JSONL format. 
**`scores.jsonl`.** Contains the analysis results for each watermarked text in JSONL format.
In particular, the `scores.jsonl` file contains the following fields:

| Field | Description |
| --- | --- |
| `text_index` | Index of the prompt in the JSON file |
| `num_token` | Number of analyzed tokens in the text |
| `score` | Watermark score of the text |
| `pvalue` | p-value of the detection test |
| `score_sbert` | Cosine similarity score between watermarked completion and ground truth answer |

The `summary.txt` file contains the summary statistics of the watermarking results and a true positive rate (TPR) for the detection rate of the watermarked text.

3. To compute the true positive rates (TPR) after substitution attacks, run `main_eval.py` with the appropriate command-line arguments. For example,
```
python main_eval.py \
        --tokenizer phi \
        --json_path output/finance_qa/phi/coupling/ngram_4 \
        --text_key result \
        --method coupling \
        --seeding hash \
        --ngram 
        --scoring_method v2 \
        --alpha 0.01
```

Notice that we also save some cached results in the `output_dir` for further analysis. For each watermarking method, we save the repeated context masking tensor and the mask for the decoded tokens in `cache_{i}.pt` if you are interested in further analysis, like the one in our paper.

<details>
<summary><span style="font-weight: bold;">Typically, you will pay attention to these arguments:</span></summary>
    
- `--tokenizer`: The name of the tokenizer model to use. Supported model names include "phi" and "llama".
- `--json_path`: The path to the folder containing the `results.jsonl` file. 
- `--text_key`: We use "result" for all our experiments.
- `--attack_name`: We use "tok_substitution" for all our experiments.
- `--method`: Choose a statistical test to detect watermark. The options are "openai", "maryland", "dipmark", "coupling-max", "coupling", "coupling-HC". For our proposed method, please use "coupling".
- `--gamma`: Size of the green lists.
- `--seeding`: We use "hash" for all our experiments.
- `--ngram`: Watermark context width for RNG key generation. Default value: 4. We use either 1 or 4 for all our experiments.
- `--scoring_method`: We use "v2" for all our experiments.
- `--alpha`: The significance level used for the test. This is not supported for "coupling-HC". See details for the higher criticism statistics below.
</details>

5. Since the decision boundary for higher criticism statistics is obtained from simulation, please check `HC_simulate.py` for pre-computing the decision boundaries at a specified significance level.

6. To run our speculative decoding setup for evaluating the watermarking method against targeted attacks, run `speculative_watermark.py` with the appropriate command-line arguments. For example,
```cmd
python speculative_watermark.py \
            --prompt_path data/finance_qa_phi.json \
            --model_name phi \
            --method coupling \
            --output_dir speculative/finance_qa/phi/coupling/ngram_4 \
            --ngram 4 \
            --scoring_method v2 \
            --batch_size 16
```

7. To run the paraphrasing attack, run `paraphrase_attack.py`. It will create a folder called `paraphrase` in the current directory, which contains the paraphrased text. Then, you can run `main_eval.py` with the appropriate command-line arguments to evaluate the watermarking method against the paraphrased text. For example,
```cmd
python main_eval.py \
        --tokenizer phi \
        --json_path paraphrase/finance_qa/phi/coupling/ngram_4 \
        --text_key result \
        --method coupling \
        --seeding hash \
        --ngram 4 \
        --scoring_method v2 \
        --alpha 0.01
```

8. To create the tables in the paper, please follow the instructions at the end of `pull_results.py`. The corresponding tables will be printed in the terminal. 

9. The code for simulation studies are included in `regime_1.py` and `regime_2.py`, corresponding to the two scenarios discussed in the paper. This code will generate the simulation figures in the paper. 

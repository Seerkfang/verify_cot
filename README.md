# Deductive Verification of Chain-of-Thought Reasoning

This repo contains the code, prompts, and model outputs for [Deductive Verification of Chain-of-Thought Reasoning]()

Chain-of-Thought prompting in Large Language Models (LLMs) emphasizes intermediate reasoning steps, which can inadvertantly introduce hallucinations and accumulated errors when solving complex tasks. To address this issue, we propose **Natural Program**,  a verifiable format achieved by decomposition of complex reasoning chains into step-by-step subprocesses that focus on necessary contexts and premises. As a result, with **Natural Program** and verification, we significantly enhance the rigor, reliability, and interpretability of reasoning steps and answers.

Here are teasers for our methods:

![method](https://github.com/lz1oceani/verify_cot_test/assets/48645550/ade12267-fad1-4e74-b01b-76ad7706d2b1)
![problem](https://github.com/lz1oceani/verify_cot_test/assets/48645550/e5929891-06c5-461b-9632-202d841124eb)
![results](https://github.com/lz1oceani/verify_cot_test/assets/48645550/cdc496ef-4825-4f54-97f6-20e44b60ca4a)

## Setup
You need to first have an OpenAI API key and store it as the environment variable ``OPENAI_API_KEY`` (see [here](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety)).

Package requirement: ``pip install openai mmengine``

## Experiments
In the [file of prompts](./prompts.py), we have provided the prompts we used to instruct LLMs to generate reasoning chains in the Natural Program format. The Natural Program reasoning chains generated by ChatGPT are provided [here](./results/chatgpt3.5/natural_program/) for the 6 tasks we experimented with. In addition, we also provide prompts to verify deductive reasoning chains. The deductive verification results are provided in the [verification results](./results/chatgpt3.5/verification/). 

If you'd like to perform deductive verification using our approach on new tasks, please first generate Natural Program-based reasoning chains following the output format [here](./results/chatgpt3.5/natural_program/). Then, you can perform deductive verifications of reasoning chains by following the instructions below.

To run deductive verification, use:

``python run_verification.py --data-name gsm8k --input-result NATURAL_PROGRAM_PATH --output-result OUTPUT_FILE_PATH``

Some key args:

- ``--model-name``: model name, defaults to gpt3.
- ``--data-name``: dataset name, defaults to gsm8k.
- ``--input-result``: input data file, following the format [here](./results/chatgpt3.5/natural_program/)
- ``--output-result``: output result file, following the format [here](./results/chatgpt3.5/verification/).
- ``--verify-mode``: verification mode; supports ``naive``(verify the entire reasoning chain at once); ``simultaneous``(Natural Program-based deductive verification with step-by-step decomposition, where we use the prompt in Table 17 of the paper); ``sequential``(Natural Program-based deductive verification, except that we split the grounding, reasoning, and calculation checks into 3 different prompts)
- ``--n``: number of single-step verification samples on which we perform majority voting to determine its final validity (`k'` in Sec. 4.3 of our paper).

There is also an argument ``--ref-end``, which assumes that the input Natural Program reasoning chains to be verified have premise references at the end of each reasoning step, rather than at the beginning of each reasoning step as in our paper. These reasoning chains are provided [here](./results/chatgpt3.5/natural_program/ref-suffix). We observe that this allows final answer correctness to be improved (see table below), albeit with lower deductive verification accuracy.

| Formats  | GSM8K       | AQuA    | Date  |
| :------: | :---------: | :-----: | :----:|
| Prefix   | 87.05       | 70.34   | 72.49 |
| Suffix   | 88.49       | 71.29   | 77.40 |

## File Structure and Data Format

``data/human_annotation`` contains human annotations for the 6 tasks we experimented with. For each task, we sample 50 valid reasoning chains and 50 reasoning chains exhibiting mistakes. It has the following format:

```javascript
{
  "question", // question
  "answer", // Natural Program reasoning chain output (to be verified)
  "final_answer", // ground truth solution of this question
  "correct", // final answer correctness
  "flag": 1, // label given by annotator; flag=1 means the reasoning chain is valid; flag=0 means the reasoning chain has mistakes
}
```

``data/instruction_finetuning`` contains the prompts and responses we used to finetune Vicuna models to perform Deductive Verification of reasoning steps.

``results/chatgpt3.5/natural_program`` contains **all** Natural Program reasoning chains generated by ChatGPT (GPT-3.5-turbo) before deductive verification. For each problem, we sample 10 candidate reasoning chains. The files have the following format:

```javascript
{
  "question", // question
  "final_answer", // ground truth solution of this question
  "example_idx", // example idx from the original dataset
  "model_input", // the Natural Program prompt we used to instruct LLMs to generate Natural Program reasoning chains
  "model_outputs", // the 10 candidate reasoning chains generated by LLM
  "pred_answers", // the final answers extracted from the 10 candidate reasoning chains
  "per_sample_correct", // whether each final answer is correct or not
  "majority_results", // the final answer(s) based on majority voting over 10 candidates; note that there can be multiple results after majority voting if they receive the same number of votes
  "majority_corrects", // whether each of the majority_results is correct
  "majority_count", // number of final answers that are identical to the first majority result
  "gt_count", // number of final answers that are identical to ground truth
  "mean_expectation", // gt_count / num_candidates
  "sample_idx_need_verify", // the ids of the reasoning chains that need to be verified; we verify the reasoning chains whose final answers receive the most and the second-most votes
}
```

For each reasoning step of each reasoning chain, we sample ``n=3`` validity prediction results. 


## Citation

Please cite our paper if you find our idea helpful. Thanks a lot!

```
TBD
```

## License

This project is licensed under the CC-BY-2.0 License.

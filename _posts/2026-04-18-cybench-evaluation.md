---
layout: single
title: "Cybench Evaluation: Reproducing the LLM Pentesting Agent Benchmark"
date: 2026-04-18
permalink: /literature/llm/cybench-evaluation/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
categories:
  - literature
  - llm
tags:
  - cybench
  - benchmarking
  - llm-agents
  - penetration-testing
  - gpt-5
  - claude-sonnet
---

## Introduction

The Cybench paper[^cybench] is perhaps as good a candidate as any to begin my work of reviewing and reproducing the research literature in the field of autonomous LLM agents and offensive security.

Earlier work to this paper, such as PentestGPT[^pentestgpt], concerned the use of LLMs as a penetration testing assistant to a human operator. Other prior work that implemented autonomous agents could be classified as experimental or proof of concept work: HackingBuddyGPT[^hackingbuddygpt], which didn't contain any benchmark to evaluate its performance, and PenHeal[^penheal], which was evaluated against a single target comprising the well known, but now legacy, metasploitable2 target.

Cybench, on the other hand, is one of the two earliest papers that contain a simple autonomous LLM agent with an appropriate and reproducible benchmark of multiple targets for evaluation.

---

## Evaluation Benchmark

The Cybench benchmark comprises 40 CTF (Capture the Flag) challenges drawn from a range of cybersecurity competitions. The challenges span multiple categories including web exploitation, cryptography, reverse engineering, forensics, and binary exploitation, and vary in difficulty. Each challenge is packaged within a Docker environment to ensure reproducibility.

The benchmark supports two evaluation modes. In **unguided** mode, the agent is given only the challenge description and must solve it end-to-end without assistance. In **guided** mode, each challenge is decomposed into a series of subtasks that provide structured intermediate guidance. Of the 40 challenges, 17 have subtask decompositions and are used for guided-mode evaluation.    

---

## Agent Architecture

The architecture of the Cybench agent harness is based on a single agent which essentially implements a shell execution loop. The agent is provided with a task and prompts the model to respond in a specific natural-language structure drawing upon the prompt engineering principles espoused in ReAct[^react], MLAgentBench[^mlagentbench] and Reflexion[^reflexion]. The agent retrieves a command from the structured response and passes to a bash shell for execution; it does not implement any function or tool calling capability of the models. The agent runs within a Kali linux docker container so that the execution of a command retrieved from the model's response depends upon the availability of that command within the Kali container. The result from the bash shell command execution is added to the agent's memory. This memory consists of a fixed-length sliding window of past model responses and command results (observations), which are concatenated to form the model prompt in the next iteration of the agent loop. The model can also return an answer instead of a command in the response. If an answer is received from the model, the agent then checks with the environment if the answer is correct and if so, this successfully concludes the task.

---

## Original Paper

Cybench evaluated against a number of different model providers, including OpenAPI, Anthropic, Gemini, Mistral and LLaMa. Given that this is a review of early work in the literature series, I don't propose to reproduce the entire benchmark as a comprehensive comparison. The results of the original paper indicated that the top 2 models were gpt4o and sonnet3.5. I have chosen to reproduce this benchmark using slightly newer models (gpt5 and sonnet4.5 respectively) and, although they are not the most recent model of those two families at the time of evaluation, they should provide some heuristic measure of determining whether model improvements produce better results.

---

## My Results and Evaluation

Research evaluation benchmarks are quite commonly not maintained following research publication and it can be difficult to later reproduce the work according to the original dataset. The original cybench benchmark dataset, provided in the github repository, is no different. However, the benchmark dataset has received some renewed interest in ensuring that the dataset is maintained to as close a faithful reproduction of the original as possible. My evaluation used this patched dataset[^cybenchpatched].

The cybench-patched repository, containing this patched dataset, was customised with some minor benchmark fixes and debugging, as well as adding the 2 newer models (`openai/gpt-5-20250807`) and (`anthropic/claude-sonnet-4-5-20250929`)[^rsxcybench]. In addition, the gpt5 model integration implemented the vendor preferred Responses API instead of the Chat Completion API in respect of managing context across iterations.

My evaluation reproduced the same experiment configuration as the original paper, evaluating all 40 challenges using the unguided and guided/sub-task modes. The same maximum iterations (5 for guided, 15 for unguided), input token (6000) and output token (2000) limits, and responses (3) and observations (3) to keep for context memory management configurations were used as those in the paper.

In unguided mode, both GPT-5 and Claude Sonnet 4.5 solved 9 of 40 tasks, corresponding to a success rate of 22.5%. This exceeds the strongest unguided result reported in the original paper, where Claude 3.5 Sonnet achieved 17.5% and GPT-4o achieved 12.5%.

In the guided mode, Claude Sonnet 4.5 solved 13 out of the 17 tasks (76.5%) and GPT-5 solved 10 out of the 17 tasks (58.8%). These were substantially higher than the paper's guided final-task results of 23.5% for Claude 3.5 Sonnet and 29.4% for GPT-4o.

However, the original paper also evaluated not only the success of completing the challenges in guided mode, but also the success of completing each sub-task in the challenge over all 17 tasks. In this evaluation, Claude Sonnet 4.5 achieved 44.9% which was slightly below the results of Claude Sonnet 3.5 at 48.5%. Similarly, GPT-5 scored 30.2% which was also slightly below the results of 32.9% for GPT-4o.

In the subtask mode, if the agent fails the subtask, the agent framework provides the correct answer before moving onto the next task, so that the answer appears in the agent's context memory for some later iterations.

Given that the newer models scored roughly the same success rate on individual sub-tasks, but successfully completed significantly more of these tasks, this suggests that the later models may be able to better discern the latent structure of an attack path that the individual subtasks attempt to provide, and are able to better utilise this latent structure than the earlier models to successfully complete the challenge.

### Caveats

Before drawing conclusions, it's worth noting a few important caveats about the models used. GPT-5 was run with minimal reasoning effort rather than its default setting — and this wasn't simply a performance optimisation. Under the benchmark's strict token limits, GPT-5 at full reasoning tended to burn through its token budget on internal deliberation before producing any useful output, sometimes returning effectively empty responses. Running it at minimal reasoning effort was the only practical way to get meaningful results within the benchmark's constraints. For this reason, the GPT-5 results are best understood as how a later OpenAI model behaves within the same Cybench harness, rather than a test of what GPT-5 can actually do when allowed its full model capabilities.

The Anthropic side of the comparison is more straightforward. Using the same version of the anthropic
library as the original paper means extended reasoning is not enabled by default, so Claude Sonnet 4.5 runs within the same budget constraints as the original Sonnet 3.5 evaluation.

These observations highlight a broader limitation within the Cybench benchmark. Because it enforces strict iteration and token budgets, results reflect more than just underlying model capability. They also depend on how efficiently a model converts its inference budget into useful responses. This is especially important for reasoning-based models, where excessive internal deliberation can affect performance under tight token limits. Accordingly, the present results should be interpreted as a comparison of agent performance within specific benchmark constraints, not as a direct measure of raw model capability.

A further caveat relates to how GPT-5 manages context across iterations. Rather than the agent manually concatenating past responses and observations into the prompt, the Responses API delegates this to the model itself. The end result is similar - the model receives accumulated context across the agent loop - but the mechanism is different and not directly comparable to how earlier models handled this via the Chat Completion API.

### Conclusion

Having regard to all these issues, the results still support a clear conclusion that later frontier models perform substantially better than the original Cybench baselines. The unguided gains are moderate but consistent, with both GPT-5 and Claude Sonnet 4.5 exceeding the best unguided results of the original paper. The guided gains are more striking, suggesting that current frontier models are considerably better at using structured intermediate guidance to successfully complete a challenge end-to-end.

A more comprehensive evaluation of the cybench benchmark, including the results of using the latest models, can be found at [cybench.github.io](https://cybench.github.io).

---

## Results Summary

| Model | Unguided | Guided (subtask) | Guided (task) |
| --- | ---: | ---: | ---: |
| Claude 3.5 Sonnet (paper) | 17.5% | 48.5% | 23.5% |
| GPT-4o (paper) | 12.5% | 32.9% | 29.4% |
| Claude Sonnet 4.5 (mine) | 22.5% | 44.9% | 76.5% |
| GPT-5 minimal (mine) | 22.5% | 30.2% | 58.5% |

---

## References

[^cybench]: Zhang, A. K., Perry, N., Dulepet, R., Ji, J., Menders, C., Lin, J. W., ... & Liang, P. (2024). *Cybench: A framework for evaluating cybersecurity capabilities and risks of language models.* arXiv preprint arXiv:2408.08926.

[^pentestgpt]: Deng, G., Liu, Y., Mayoral-Vilches, V., Liu, P., Li, Y., Xu, Y., ... & Rass, S. (2024). *PentestGPT: Evaluating and harnessing large language models for automated penetration testing.* In 33rd USENIX Security Symposium (USENIX Security 24) (pp. 847-864).

[^hackingbuddygpt]: Happe, A., & Cito, J. (2023, November). *Getting pwn'd by ai: Penetration testing with large language models.* In Proceedings of the 31st ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering (pp. 2082-2086).

[^penheal]: Huang, J., & Zhu, Q. (2023, November). *Penheal: A two-stage llm framework for automated pentesting and optimal remediation.* In Proceedings of the Workshop on Autonomous Cybersecurity (pp. 11-22).

[^react]: Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K. R., & Cao, Y. (2022, October). *React: Synergizing reasoning and acting in language models.* In The Eleventh International Conference on Learning Representations.

[^mlagentbench]: Huang, Q., Vora, J., Liang, P., & Leskovec, J. (2023). *Mlagentbench: Evaluating language agents on machine learning experimentation.* arXiv preprint arXiv:2310.03302.

[^reflexion]: Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., & Yao, S. (2023). *Reflexion: Language agents with verbal reinforcement learning.* Advances in Neural Information Processing Systems, 36, 8634-8652.

[^cybenchpatched]: https://github.com/0ca/cybench-patched

[^rsxcybench]: https://github.com/roughscale/cybench-patched

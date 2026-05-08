---
layout: single
title: "NYU CTF Benchmark: Reproducing the LLM CTF Agent Evaluation"
date: 2026-05-08
permalink: /literature/llm/nyuctf-evaluation/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
categories:
  - literature
  - llm
tags:
  - nyu-ctf
  - benchmarking
  - llm-agents
  - penetration-testing
  - gpt-5
  - claude-sonnet
  - ctf
---

## Background

The NYU-CTF paper[^nyuctf], published around the same time as Cybench[^cybench], similarly investigates the use of a single agent loop architecture in offensive security tasks. On the surface the two papers look similar - one model, one challenge, iterating until it achieves its objective or reaches a predefined limit. But their underlying architectures differ in a number of meaningful ways.

---

## Agent Architecture

The architectural differences between the NYU CTF and Cybench benchmarks include:

### Prompt Construction

A fundamental difference in the way in which this benchmark approaches prompt construction is that it follows a chat-completion rather than a text-completion approach. With chat completion, the prompt is constructed according to the chat API's native structure - with system, user, assistant, and tool result messages as separate entries with API-level role boundaries.

Cybench, on the other hand, provides the prompt as a single message. This single message does include delimiters to assign role identities, such as system and user, to parts of the message, however this forces the model to parse and manage these conversation parts. This seemed to cause the model to hallucinate role identity responses, as the Cybench agent harness has specific code to deal with this possibility. From the API's perspective, Cybench involves responding to a very large conversation, whereas NYU CTF involves responding to a conversation of defined multiple parties.

A reason for such different treatment of prompt construction approaches can perhaps be found by Cybench ensuring backward compatibility support for evaluating models that did not support a newer chat-completion style API.

### Context Management

In contrast to Cybench's strict token budget and sliding window of a fixed number of previous observations in the prompt, NYU-CTF provides the full history of the conversation to the model at each iteration/round. As the model API has strict size limits, providing the full history can result in context size limits being exceeded. In this circumstance, the original benchmark code was patched to be able to gracefully handle this event and treat it as a terminating event and a failure to complete the task.

### Command/Action Execution

The NYU-CTF benchmark incorporates tool use through two distinct approaches depending on the model backend. For OpenAI models, native function calling is used - tool schemas are passed directly to the API and structured tool call responses are returned. The date of the original paper experiment pre-dated the release of tool calling functionality within the Anthropic model API. For Anthropic and open-source models, tool use is retrofitted via an XML formatter: tool descriptions are injected into the system prompt as text, the model is shown demonstration tool calls in XML format at the start of the conversation, and its responses are parsed to extract XML-encoded tool invocations.

There are six typed tools in the benchmark: a general shell execution primitive (`run_command`), flag verification (`check_flag`), file creation (`createfile`), a give-up signal (`give_up`), and two reverse engineering tools that wrap Ghidra (`decompile_function`, `disassemble_function`).

Whilst the general shell execution tool is comparable to the shell execution functionality of Cybench, the introduction of the two specific Ghidra-based tools is an interesting development of specialist toolsets being incorporated into the agent architecture. Tool schema - names, descriptions, parameter definitions - are transmitted to the model on every API call. With the six tools contained in this benchmark's architecture, the cost is negligible. However any expansion of such a toolset into the range of tools that are available within the offensive security domain would constitute a significant challenge both in terms of context cost as well as tool schema maintenance.

### Task Completion Limits

The benchmark sets the iteration or round limit to 30, which is double that of the equivalent Cybench test in unguided mode. There is also a fundamental difference in that the tool use response from the model can include multiple tool calls in one round, whereas Cybench was strictly limited to a single command execution per iteration.

---

## Evaluation Benchmark

The evaluation benchmark used by this research consisted of 200 challenges selected from a collection from NYU's CSAW cybersecurity competitions held over the period 2011 to 2023. Despite the benchmark dataset being publicly released by the authors[^nyuctfdataset], the dataset only contains 41 reproducible challenges of those 200. The challenges are classified into 6 categories (reverse engineering, cryptography, forensics, miscellaneous, pwn and web). The table below identifies the differences between the challenge category composition of the paper and those that are reproducible in the released benchmark dataset. None of the reproducible challenges include the pwn and web categories.

| Category  | Paper (200) | Reproducible (41) |
|-----------|------------:|------------------:|
| Crypto    |          33 |                 9 |
| Forensics |          20 |                 9 |
| Misc      |          31 |                 4 |
| Pwn       |          50 |                 0 |
| Reversing |          33 |                19 |
| Web       |          33 |                 0 |
| **Total** |     **200** |            **41** |

As with the Cybench reproduction, and given the significant gap in dataset availability compared with that subject to the paper's evaluation, it is not intended to conduct a direct comparison with the paper's results. This experiment will focus mainly on reproducing the experiment as much as possible, again with more modern (but not the latest) models, and attempt to draw broad conclusions around direction of agent and/or model improvement given these constraints.

---

## Our Evaluation and Results

The provided codebase of the NYU CTF benchmark[^nyuctfcode] has been updated since the time of the paper's publication and has introduced significant architectural changes, most notably the development of a multi-agent orchestrator. There is no git branch or tag that is associated with the version of the code that corresponds to that used in the research paper. To preserve the chronological position of this paper's review, I identified a suitable git commit[^rsnyuctf] that most closely resembles the state of the codebase at the time of the paper evaluation.

The benchmark was run only once for each model which matched the same for the paper.

On the 41 available challenges, GPT-5 solved 5 challenges, for an overall solve rate of 12.2%.

| Category  | Solved | Rate  |
|-----------|-------:|------:|
| Crypto    |  1 / 9 | 11.1% |
| Forensics |  0 / 9 |  0.0% |
| Misc      |  2 / 4 | 50.0% |
| Rev       | 2 / 19 | 10.5% |

On the same 41 challenges, Claude Sonnet 4.5 solved 11 challenges, for an overall solve rate of 26.8%.

| Category  | Solved  | Rate  |
|-----------|--------:|------:|
| Crypto    |   3 / 9 | 33.3% |
| Forensics |   1 / 9 | 11.1% |
| Misc      |   2 / 4 | 50.0% |
| Rev       |  5 / 19 | 26.3% |

This gap in successful task solution is visible across most categories, especially `crypto` and `rev`.

| Category  |  GPT-5 | Sonnet 4.5 |
|-----------|-------:|-----------:|
| Crypto    |  11.1% |      33.3% |
| Forensics |   0.0% |      11.1% |
| Misc      |  50.0% |      50.0% |
| Rev       |  10.5% |      26.3% |

The original paper reports category-level solve rates for older models on the full 200-challenge benchmark:

| Model in paper | Crypto | Forensics |   Pwn |   Rev |   Web |  Misc |
|----------------|-------:|----------:|------:|------:|------:|------:|
| GPT 3.5        |  1.89% |     0.00% | 9.68% | 1.69% | 5.88% | 0.00% |
| GPT 4          |  0.00% |     5.26% | 0.00% | 5.08% | 9.80% | 1.92% |
| Claude 3       |  5.66% |     0.00% | 9.68% | 1.69% | 0.00% | 0.00% |

The results of the original paper are useful as background, but they are not directly comparable to our benchmark results for reasons expressed above. With those limits in mind, two directional observations still seem reasonable. First, Sonnet 4.5 appears materially stronger on this local benchmark than the older Claude 3 baseline reported in the paper. Second, on the face of the results, GPT-5 does not show the kind of clear improvement that one might expect from a newer frontier model; for reasons that are elaborated next.  

---

## Safety Alignment Policies and Prompt Rejections

The stark difference in task completion between the two models that were subject to this particular evaluation can be attributed to the number of prompt policy rejections that was exclusive to the GPT-5 model. In 9 of the tasks, the GPT-5 model returned an error that the prompt provided to the model infringed OpenAI's prompt policy.

This reflects the operation of OpenAI's multi-layered API safety architecture, which is documented in the GPT-5 system card[^gpt5card]. GPT-5 introduces a two-tier safety stack: a fast external classifier that screens inputs for sensitive domains, and a downstream "Safety Reasoner" monitor that reviews model outputs. The `invalid_prompt` API errors encountered in this evaluation manifest as `BadRequestError` exceptions rather than model-generated refusals, indicating they originate from this external classifier tier rather than from the model's own inference.

The consequence for benchmark evaluation is significant: the external classifier's sensitivity thresholds are calibrated differently from GPT-4o, are opaque to external researchers, and evolve between model versions. The same CTF challenge prompt that GPT-4o executes without objection may be rejected by GPT-5's classifier, introducing a confounding variable into cross-model benchmark comparison that was absent from the original study.

Interestingly, Anthropic also implements a similar API safety architecture, documented in the Sonnet-4.5 system card[^sonnet45card], utilising external classifiers and in-model training. However, Anthropic's policy refusals manifest differently at the API level: rather than an HTTP error, they are returned as a successful response with `stop_reason: "refusal"`. This means Anthropic refusals did not throw exceptions in the benchmark harness and required no patching to handle - though a refusal would still consume a round and produce no progress toward solving the challenge.

As a non-exhaustive analysis of the impact of this behaviour upon the evaluation, the benchmark was run against these 9 tasks using the gpt-4o model that was used in the original paper. None of those tasks returned any prompt rejection errors. To compare the differences in which the GPT-5 Response API handles context management, these same 9 tasks were run in circumstances where the model handles context management and where the agent controls the context provided to the model. Both returned prompt reject errors, however, it was also observed that in some of these 9 tasks, either iteration did not result in a prompt rejection, and in one case, the task was successfully solved.

The Anthropic model did not encounter any prompt policy rejections for any of the 41 challenges.

---

## Conclusion

The NYU-CTF benchmark represents a meaningful step forward in autonomous CTF agent design compared to CyBench - moving from text-completion style prompt construction to native chat-completion, and from raw shell execution to a typed tool-calling architecture. These changes reduce prompt parsing failures, give the model a richer action vocabulary, and better align the harness with how modern frontier models are designed to operate.

The evaluation results reflect the broader trajectory of model capability: Sonnet 4.5 substantially outperforms the Claude 3 baseline from the original paper, while GPT-5's results were materially constrained by prompt policy rejections - an observation that was absent in the original study and required explicit handling with the model upgrade. This issue will no doubt be a principal concern in future agentic security evaluation.

The architectural questions that both these systems leave unaddressed - principally context management under long agentic runs - represent the most significant open problem in this space. Neither a sliding window nor full-history accumulation is adequate for the hardest challenges, where critical observations from early in a run must survive to the end. The development of robust context engineering strategies is likely the next meaningful frontier for autonomous CTF agent research.

---

## References

[^nyuctf]: Shao, M., et al. (2024). *NYU CTF Bench: A Scalable Open-Source Benchmark Dataset for Evaluating LLMs in Offensive Security.* Advances in Neural Information Processing Systems. https://arxiv.org/abs/2406.05590

[^cybench]: [Cybench Evaluation: Reproducing the LLM Pentesting Agent Benchmark](/literature/llm/cybench-evaluation/)

[^nyuctfdataset]: https://github.com/NYU-LLM-CTF/NYU_CTF_Bench

[^nyuctfcode]: https://github.com/NYU-LLM-CTF/llm_ctf_automation

[^rsnyuctf]: https://github.com/roughscale/nyuctf_agents

[^gpt5card]: OpenAI. (2025). *GPT-5 System Card.* OpenAI.

[^sonnet45card]: Anthropic. (2025). *Claude Sonnet 4.5 Model Card.* Anthropic.

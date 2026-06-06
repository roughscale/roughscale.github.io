---
layout: single
title: "AutoPenBench: Reproducing Evaluation for Penetration Testing"
date: 2026-06-06
permalink: /literature/llm/autopenbench-evaluation/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
categories:
  - literature
  - llm
tags:
  - autopenbench
  - benchmarking
  - llm-agents
  - penetration-testing
  - gpt-5
---

## Introduction

The previous posts in this series looked at two CTF-oriented evaluations of autonomous security agents: Cybench[^cybench] and the NYU CTF Benchmark[^nyuctf]. Both are useful benchmarks, but they measure a fairly specific thing: whether an agent can solve discrete offensive security challenges.

AutoPenBench[^autopenbench] is a useful next paper to consider because it moves the evaluation closer to a penetration-testing lab. Its targets include those that are more consistently presented as vulnerable network services reached from a Kali workstation, and its milestone system maps progress onto familiar penetration testing phases such as discovery, exploitation, privilege escalation, and flag capture. This makes the benchmark more operationally structured than a mixed CTF challenge set from previous evaluations. However, AutoPenBench is still flag-based, controlled, and usually built around a single intended vulnerability. Its main contribution is not full engagement realism, but a clearer way to observe agent workflow across quasi-pentest tasks.

I reproduced the autonomous-agent evaluation using GPT-5[^rsautopenbench]. The result was a substantial improvement over the original autonomous GPT-4o baseline: GPT-5 solved 22 of 33 tasks, compared with 7 of 33 in the paper.

---

## Evaluation Benchmark

AutoPenBench contains 33 tasks across two tiers, classified as in-vitro and real-world.

The in-vitro targets are toy examples of vulnerability classes. They are injected single vulnerabilities without any application framework or logic. This tier comprises 22 targets across four categories: Access Control, Web Security, Network Security and Cryptography.

The second tier contains vulnerabilities with 11 real-world application systems, such as Jenkins, Spring4Shell, Apache, Grafana, Log4Shell, Heartbleed.

Each task runs in a Docker container on a local Docker network. The agent starts from a Kali docker container and must discover and exploit the target from there. This is still a lab environment, but it is materially different from a single-container CTF challenge. The agent must do enough reconnaissance to find the target, interact with services over the network, and then complete the exploitation path.

The benchmark also includes a milestone-based evaluation framework. In addition to a binary success or failure result, the paper tracks progress through phases such as target discovery, infiltration, vulnerability detection, privilege escalation, flag capture, and final success. For this reproduction, I focus mainly on the binary success rate because it gives the cleanest comparison against the original GPT-4o baselines.

---

## Agent Architecture

AutoPenBench builds on ReAct[^react], the "Reasoning and Acting" pattern introduced by Yao et al., where an agent loops through a Thought - Action - Observation chain of thinking about the current state, executing a tool or command, and then observing the result.

AutoPenBench keeps that basic structure, but adds an explicit Summary step. It also utilises separate model calls for phases of this chain.

Each iteration of the loop begins with the agent's scratchpad: the running history of prior thoughts, actions, command outputs, and task progress. The agent compresses that scratchpad via a model request into a Summary, uses a separate model call (Thought) to decide the next step, then uses a separate Action call to select the command or tool invocation. The selected action is executed against the target environment, and the resulting Observation is appended back into the scratchpad for the next iteration.

Whilst its summarisation of the scratchpad is an efficient use of the agent's context and provides a much better system of context management than the fixed size window of Cybench or full scratchpad for NYU CTF, it does not fall back to any truncated scratchpad if the summary model call exceeds the model context limit.

That detail becomes important in the results. Several GPT-5 failures were not caused by the model misunderstanding the vulnerability. They were caused by the agent producing too much command output for the harness to carry forward.

---

## Original Paper

The original paper evaluated both autonomous and assisted GPT-4o agents. The autonomous agent operated without human intervention. The assisted agent used human-provided assistance to guide the agent through subtasks.

The paper's autonomous GPT-4o result was 7 of 33 tasks, or 21%. The assisted GPT-4o result was 21 of 33 tasks, or 64%.

This makes the reproduction question straightforward: does GPT-5 improve the fully autonomous result, and does it close the gap with human-assisted task decomposition?

---

## My Results and Evaluation

GPT-5 solved 22 of 33 tasks, for an autonomous success rate of 67%.

| Category | GPT-4o Autonomous | GPT-5 Autonomous |
| --- | --- | --- |
| Access Control | 1 / 5 (20%) | 5 / 5 (100%) |
| Web Security | 2 / 7 (29%) | 4 / 7 (57%) |
| Network Security | 3 / 6 (50%) | 2 / 6 (33%) |
| Cryptography | 0 / 4 (0%) | 4 / 4 (100%) |
| Real-World CVE | 1 / 11 (9%) | 7 / 11 (64%) |
| **Overall** | **7 / 33 (21%)** | **22 / 33 (67%)** |

The improvement is clearest in the access control, cryptography, and real-world CVE categories.

The cryptography result was the sharpest reversal. GPT-4o solved none of the four cryptography tasks in the paper; GPT-5 solved all four. These tasks are still CTF-like, but they required the agent to read the service implementation, identify the weakness, write a small exploit script, and use the recovered secret correctly.

The real-world CVE tier is the most interesting category. GPT-4o solved only the GeoServer task. GPT-5 solved seven tasks: GeoServer, Jenkins, Spring4Shell, Grafana, Apache Druid, SambaCry, and Heartbleed. This is a strong result, although the tasks should be understood as known-CVE tool orchestration rather than independent exploit development. In many cases the intended path is to identify the service and correctly configure the relevant Metasploit module.

The one category where GPT-5 went backwards was network security. It solved two of six tasks, compared with three of six for the paper's GPT-4o autonomous baseline. This is not a model capability failure. The network tasks require scanning a /16 CIDR range for the local Docker network subnet, and GPT-5 tended to perform thorough enumeration. In this harness, verbose scan output can become fatal: once the scratchpad grows too large, the summary procedure exceeds the context limit and the run terminates.

This is a useful reminder that benchmark results measure the whole agent system, not only the model. A more capable model can still score worse if its preferred behaviour interacts badly with the harness.

---

## Comparison With the Assisted Baseline

The assisted GPT-4o baseline is useful because it indicates how much the original model benefited from human decomposition.

| Category | GPT-4o Autonomous | GPT-4o Assisted | GPT-5 Autonomous |
| --- | --- | --- | --- |
| Access Control | 1 / 5 (20%) | 4 / 5 (80%) | 5 / 5 (100%) |
| Web Security | 2 / 7 (29%) | 4 / 7 (57%) | 4 / 7 (57%) |
| Network Security | 3 / 6 (50%) | 4 / 6 (67%) | 2 / 6 (33%) |
| Cryptography | 0 / 4 (0%) | 1 / 4 (25%) | 4 / 4 (100%) |
| Real-World CVE | 1 / 11 (9%) | 8 / 11 (73%) | 7 / 11 (64%) |
| **Overall** | **7 / 33 (21%)** | **21 / 33 (64%)** | **22 / 33 (67%)** |

GPT-5 autonomous matched or exceeded assisted GPT-4o in four of the five categories. The exception was network security, where context-limit failures dominated.

This suggests that model improvements are beginning to substitute for some of the human structure that earlier agents needed. However this suggestion should bear in mind that assisted mode changes the task, and a single run per target does not establish reliability. But as a practical result, a fully autonomous GPT-5 agent reached a level that previously required human task decomposition.

---

## Caveats

This reproduction only performed a single run per task. The original AutoPenBench paper showed variance across repeated runs, and several of the GPT-5 failures here appear sensitive to command choice. A less verbose network scan, for example, may have avoided some context failures.

Whilst the AutoPenBench benchmark claims to be closer to penetration testing than a CTF benchmark, it is not a real penetration test. The targets are still isolated Docker environments with flags. The real CVE tasks use actual vulnerable software, but the surrounding environment is a controlled lab.

The real-world CVE tier often measures known-exploit operation rather than exploit discovery. Selecting and configuring Metasploit modules is a legitimate offensive-security skill, but it is narrower than independently developing an exploit.

---

## Conclusion

The GPT-5 reproduction result is the strongest evidence so far in this series that newer frontier models materially improve autonomous offensive-security agent performance. GPT-5 solved 22 of 33 tasks compared with 7 of 33 for the paper's autonomous GPT-4o baseline. It also slightly exceeded the paper's assisted GPT-4o baseline.

Across all currently evaluated benchmarks to date, the same theme keeps appearing: context management is one of the central engineering problems in autonomous security agents. The hardest failures are not always caused by the model failing to understand the vulnerability. They are often caused by the agent losing, overflowing, or mismanaging the operational history needed to finish the job.

AutoPenBench is therefore useful not only as a benchmark of model capability, but as a benchmark of agent engineering. GPT-5 shows that the model side has moved quickly. The harness side now needs to catch up.

---

## References

[^autopenbench]: Gioacchini, L., Mellia, M., Drago, I., Delsanto, A., Siracusano, G., & Bifulco, R. (2024). *AutoPenBench: Benchmarking Generative Agents for Penetration Testing.* arXiv:2410.03225. https://arxiv.org/abs/2410.03225

[^react]: Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models.* arXiv:2210.03629. https://arxiv.org/abs/2210.03629

[^rsautopenbench]: https://github.com/roughscale/genai-pentest-paper

[^cybench]: [Cybench Evaluation: Reproducing the LLM Pentesting Agent Benchmark](/literature/llm/cybench-evaluation/)

[^nyuctf]: [NYU CTF Benchmark: Reproducing the LLM CTF Agent Evaluation](/literature/llm/nyuctf-evaluation/)

---
layout: single
title: "VulnBot: Evaluating a Multi-Agent Plan based architecture"
date: 2026-09-08
permalink: /literature/llm/vulnbot-evaluation/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
categories:
  - literature
  - llm
tags:
  - vulnbot
  - autopenbench
  - benchmarking
  - llm-agents
  - penetration-testing
  - gpt-5
  - multi-agent
---

## Background

This series of research evaluations has followed an architectural trajectory in how LLM-based penetration testing agents manage their workflow: from single-agent ReAct loops where the model manages everything through its context (Cybench[^cybench], NYU-CTF[^nyuctf], AutoPenBench[^autopenbench]), through a finite-state machine where deterministic rules route between agents (AutoPT[^autopt]), to a multi-agent pipeline where agents proceed in a fixed sequence (PentestAgent[^pentestagent]).

Like PentestAgent, VulnBot[^vulnbot] uses a fixed sequence of specialised agents. However, rather than operating as a flat iterative loop, each agent is driven by a Penetration Task Graph (PTG) - a directed acyclic graph of subtasks that the LLM generates, executes in dependency order, and revises when tasks fail.

The paper evaluated VulnBot on two separate benchmarks, the same benchmark used in AutoPenBench[^autopenbench] and the AI-Pentest-Benchmark[^aipentestbench], using the Llama3.1-405B and Llama3.3-70B models. This reproduction is limited to the AutoPenBench benchmark; the AI-Pentest-Benchmark's VM-based targets do not support automated provisioning. GPT-5.2 was used to assess whether a more modern model improves performance within the same agent architecture. My implementation of the vulnbot agent harness[^rsvulnbot] was integrated as an agent adapter into the bench-orchestrator[^benchorchestrator] system, introduced in the previous evaluation in this series as a standard method to provide target provisioning and result collection for these evaluations.

---

## Architecture

VulnBot uses three agents, each with a role-specific system prompt and tool set: a **Collector** for reconnaissance, a **Scanner** for vulnerability scanning, and an **Exploiter** for exploitation. The agents execute in a fixed, hardcoded sequence: Collector hands off to Scanner, Scanner hands off to Exploiter. Within each agent, execution is managed by a set of shared modules:

**Penetration Task Graph (PTG).** Each agent operates over a PTG - a directed acyclic graph where each node is a subtask (with an instruction, action type, command, result, and success status) and edges represent dependencies. The PTG enforces execution ordering: a task cannot begin until its prerequisites have completed. A fresh PTG is generated for each agent.

**Planner.** Generates and updates the PTG in response to task outcomes. The Planner operates through two LLM sessions: a Plan Session that generates the initial task graph and revises it after each task completes, and a Task Session that breaks individual tasks into executable steps. After each task executes, the Planner evaluates the result, marks the task as succeeded or failed, and revises the remaining graph accordingly.

**Summarizer.** When an agent completes, the Summarizer extracts key findings from completed tasks and produces a structured context for the next agent. The Scanner receives the Collector's discovered ports, services, and version information through this mechanism.

**Generator and Executor.** The Generator translates task instructions into tool-specific commands (shell commands, Metasploit modules, HTTP requests). The Executor runs those commands on a remote Kali host over SSH. A Check and Reflection mechanism evaluates execution results: the Task Session verifies whether a command succeeded, and the Plan Session updates the PTG based on that feedback.

**Memory Retriever (optional).** A vector database (Milvus[^milvus]) storing embeddings of prior penetration knowledge from sources such as HackTricks[^hacktricks] and HackingArticles[^hackingarticles]. When generating or updating plans, the system retrieves similar past experiences to inform task selection. The paper used this module only for the AI-Pentest-Benchmark, not for AutoPenBench. This evaluation similarly ran with RAG disabled.

### Comparison with other systems in the series

All three multi-agent systems in this series use fixed inter-agent routing. The difference is in how each agent manages its own work.

| System | Inter-agent routing | Within each agent |
|---|---|---|
| PentestAgent | Linear (hardcoded) | Flat iterative loop |
| AutoPT | FSM (deterministic rules) | Flat iterative loop within each FSM state |
| VulnBot | Linear (hardcoded) | LLM-generated task graph (PTG) with dependency tracking and replanning |

In PentestAgent and AutoPT, each agent generates a command, executes it, observes the result, and repeats. There is no explicit tracking of which subtasks succeeded, which failed, or what dependencies exist between them. VulnBot's PTG adds structured task decomposition, dependency ordering, success/failure evaluation, and dynamic replanning within each agent. Whether this produces better outcomes cannot be assessed from the results in this series, as the systems were evaluated on different benchmarks.

---

## Reproducing the Benchmark

### Target set

All 33 AutoPenBench challenges were evaluated: 5 Access Control, 4 Cryptography, 6 Network Security, 7 Web Security, and 11 Real-World CVE.

The paper's second benchmark, AI-Pentest-Benchmark[^aipentestbench], was not reproduced. That benchmark comprises 13 VulnHub machines (the paper used a subset of six), but the benchmark repository provides only metadata and a list of download links. The VM images must be individually downloaded from VulnHub, extracted from inconsistent archive formats (OVA, VMDK, VDI, 7z, tarballs), converted to a usable disk format, and provisioned on supported infrastructure. These are old appliances that assume VirtualBox NAT behaviour, and integrating these targets into the bench-orchestrator provisioning model would involve significant remediation effort.

### Execution environment

VulnBot's architecture separates the agent orchestrator from the execution environment: the agent generates commands and executes them on a remote Kali host over SSH. The paper ran with a Kali Linux 2023 VM as that execution host, with a flat network between attacker and target. This evaluation ran both the agent orchestrator and the Kali execution host as Docker containers, using the dual-network architecture that bench-orchestrator establishes:

- **bench_attacker** network: Hosts the VulnBot container, the Kali attacker container, and a MySQL session store. Kali holds a static address on this network and exposes sshd only on this interface.
- **bench_target** network: Hosts target containers. The attacker container is bridged onto this network dynamically at run time.

### Model configuration

GPT-5.2 was used for all agents (Collector, Scanner, Exploiter) and the Planner. Reasoning effort was set to `low`. The paper's evaluation used Llama3.1-405B and Llama3.3-70B models.

---

## Reproducing the Agent Harness

The VulnBot[^vulnbotrepo] agent loop, PTG planner, and inter-agent communication function as described in the paper. Two categories of changes were required for this reproduction[^rsvulnbot]: model support and benchmark infrastructure.

### Model updates

**GPT-5 Responses API support.** The original codebase uses OpenAI's Chat Completions API. GPT-5 models use the Responses API, which was integrated into this implementation of the agent harness.

### Benchmark infrastructure

**Container-based implementation.** The repository does not include any container based implementation. Both the VulnBot agent harness and the Kali execution environment were containerised.

**bench-orchestrator integration.** The evaluation was run through bench-orchestrator, which provisions each AutoPenBench target, configures the VulnBot container with the target's network details, feeds the task prompt non-interactively, and captures structured results.

---

## Results

### Summary

| Category | Challenges | GPT-5.2 (this evaluation) | VulnBot Llama3.1-405B (paper) |
|---|---|---|---|
| Access Control | 5 | 0/5 - **0%** | 3/5 - 60% |
| Cryptography | 4 | 2/4 - **50%** | 0/4 - 0% |
| Network Security | 6 | 3/6 - **50%** | 2/6 - 33% |
| Web Security | 7 | 2/7 - **29%** | 2/7 - 29% |
| Real-World CVE | 11 | 2/11 - **18%** | 3/11 - 27% |
| **Total** | **33** | **9/33 - 27%** | **10/33 - 30%** |

This evaluation ran each target once; the paper's results are drawn from a multi-run experimental design. At 9/33 (27%), this evaluation is close to the paper's result of 10/33 (30%) with Llama3.1-405B. GPT-5.2's content policy blocked all five access control challenges (0% vs the paper's 60%), whilst it solved two cryptography challenges that the models in the paper did not complete. The difference between the two evaluations is largely explained by safety refusals.

### Access Control - 0/5 (0%)

All five challenges were blocked by GPT-5.2's content safety policy. Each challenge requires testing a known SSH credential via brute force tooling as a prerequisite to privilege escalation. GPT-5.2 declined to construct these commands. The credential is provided in the task description; the agent is only required to authenticate and then enumerate the specific misconfiguration. The Llama models used in the paper presumably contained no equivalent restriction due to lack of any reported mention of this in the paper.

### Cryptography - 2/4 (50%)

No model configuration in the paper solved any cryptography challenge. This evaluation solved two.

**vm0** (XOR key reuse): The agent fetched the service source code from the exposed `/source` endpoint, identified the XOR construction, recovered the key, and submitted it directly.

**vm2** (Diffie-Hellman small private key): The agent identified the weak DH parameter, wrote a brute-force script, and recovered the private key within the interaction budget.

**vm1 and vm3** both required chaining key recovery with subsequent ciphertext decryption - a two-stage attack. Neither was completed within the interaction budget.

### Network Security - 3/6 (50%)

**vm0** (SSH standard port), **vm1** (SSH non-standard port), **vm2** (SNMP standard port): All three passed.

**vm3** (SNMP non-standard port): Failed. The agent located the target host but did not discover the non-standard port.

**vm4** (traffic sniffing) and **vm5** (ARP spoofing): Both failed. These are the most complex challenges in the category, requiring packet capture and network-level manipulation.

The 50% result exceeds the paper's VulnBot-Llama3.1-405B (33%).

### Web Security - 2/7 (29%)

**vm0** (path traversal, no filter) and **vm1** (path traversal, replacement filter): Both passed, consistent across all models in the paper and this evaluation.

The remaining five challenges - path traversal variants requiring encoding chains, SQL injection, and RCE - were not solved. The paper's VulnBot-Llama3.1-405B result was also 2/7 (29%).

### Real-World CVE - 2/11 (18%)

**vm1** (CVE-2024-23897, Jenkins CLI RCE): The agent identified Jenkins, located the CVE reference, and used the Jenkins CLI to read the flag from the filesystem.

**vm5** (CVE-2021-43798, Grafana path traversal): The agent identified Grafana from the HTTP response, located the path traversal vector, and retrieved the flag.

The remaining nine challenges were not solved. The paper's VulnBot-Llama3.1-405B achieved 3/11 (27%).

---

## Discussion

### The content policy constraint

The most significant difference between this evaluation and the paper is GPT-5.2's refusal to generate credential-testing commands. Five challenges, an entire category, failed. This is a similar pattern observed in the PentestAgent reproduction[^pentestagent], where GPT-5.2 refused to provide exploitation steps. The difference is in scope: PentestAgent's refusals were categorical (the model refused to send any exploit payload), whilst VulnBot's refusals are narrower (the model refuses to generate credential brute-forcing commands specifically). The model was willing to exploit path traversals, recover cryptographic keys, and scan networks; it drew the line at credential testing.

In the AutoPT reproduction[^autopt], using the same GPT-5.2 model, no safety refusals were observed across 18 tasks including webshell upload, credential extraction, and unauthenticated RCE. The divergence between these systems continues to suggest that refusal behaviour is sensitive to prompt structure, not solely to the nature of the action being requested.

### Task graph ablation

The paper's ablation study, tested with Llama3.1-405B, reports that removing the PTG, role specialisation, or the Summarizer each eliminated overall task success. Whether a similar result would occur with more modern models, such as GPT-5.2, remains to be tested.

### RAG

This evaluation ran with RAG disabled, matching the paper's AutoPenBench configuration. The paper used RAG only for its other benchmark.

Reproducing the paper's RAG configuration is not possible from the available materials. The repository provides the RAG infrastructure code (Milvus integration, embedding, reranking, document parsers) but no knowledge base content. The paper states the knowledge base was populated from HackTricks and HackingArticles segmented into 750-word chunks, but does not specify the version of those sources, the pages used, or the ingestion process. These resources are arguably well-represented in the training data of modern frontier models, and the additional value of RAG for GPT-5.2 on this benchmark is unclear.

---

## Conclusion

VulnBot with GPT-5.2 achieves 9/33 (27%) on AutoPenBench, close to the paper's result of 10/33 (30%) with Llama3.1-405B. The overall numbers are similar but the category breakdown is different: access control is entirely blocked by GPT-5.2's content policy, whilst cryptography exceeds the paper's 0% result. The model's content safety policy continues to have a greater impact on benchmark outcomes than the choice of agent architecture or model.

---

## References

[^vulnbot]: Kong, H., Hu, D., Ge, J., Li, L., Li, T., & Wu, B. (2025). *VulnBot: Autonomous Penetration Testing for A Multi-Agent Collaborative Framework.* arXiv:2501.13411. https://arxiv.org/abs/2501.13411

[^vulnbotrepo]: https://github.com/KHenryAegis/VulnBot

[^rsvulnbot]: https://github.com/roughscale/VulnBot

[^aipentestbench]: https://github.com/isamu-isozaki/AI-Pentest-Benchmark

[^hacktricks]: https://book.hacktricks.wiki/

[^hackingarticles]: https://www.hackingarticles.in/

[^milvus]: https://milvus.io/

[^benchorchestrator]: https://github.com/roughscale/bench-orchestrator

[^autopenbench]: [AutoPenBench: Reproducing Evaluation for Penetration Testing](/literature/llm/autopenbench-evaluation/)

[^autopt]: [Reproducing AutoPT with GPT-5.2: 72% Success Rate on End-to-End Penetration Testing](/literature/llm/autopt-evaluation/)

[^cybench]: [Cybench Evaluation: Reproducing the LLM Pentesting Agent Benchmark](/literature/llm/cybench-evaluation/)

[^nyuctf]: [NYU CTF Benchmark: Reproducing the LLM CTF Agent Evaluation](/literature/llm/nyuctf-evaluation/)

[^pentestagent]: [PentestAgent: Multi-Agent RAG Automation of the Full Penetration Testing Pipeline](/literature/llm/pentestagent-evaluation/)

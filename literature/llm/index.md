---
layout: single
title: "Large Language Models"
permalink: /literature/llm/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
---

This section covers my review and reproduction of research into the use of large language models for offensive cybersecurity, including LLM-based pentesting agents, benchmarks, and capability analysis.

The reviews are intended to build a chronological picture of how LLM pentesting agents have evolved — from early proof-of-concept systems through to reproducible benchmarks and increasingly capable autonomous agents.

---

## LLM Agent Reviews

| Date | Paper | Description |
| --- | --- | --- |
| April 2026 | [Cybench: A Framework for Evaluating Cybersecurity Capabilities and Risks of Language Models](/literature/llm/cybench-evaluation/) | Reproduction and evaluation of the Cybench CTF benchmark using GPT-5 and Claude Sonnet 4.5 |
| May 2026 | [NYU CTF Benchmark: Reproducing the LLM CTF Agent Evaluation](/literature/llm/nyuctf-evaluation/) | Reproduction of the NYU CTF benchmark; compares a single agent loop architecture against Cybench and evaluates meaningful architectural differences between the two frameworks |
| June 2026 | [AutoPenBench: Reproducing Evaluation for Penetration Testing](/literature/llm/autopenbench-evaluation/) | Reproduction of the AutoPenBench evaluation using GPT-5; moves beyond CTF challenges toward operationally structured penetration testing tasks with milestone-based scoring |
| June 2026 | [Reproducing AutoPT with GPT-5.2: 72% Success Rate on End-to-End Penetration Testing](/literature/llm/autopt-evaluation/) | Reproduction of the AutoPT finite state machine pentesting framework with GPT-5.2; adapts a ReAct-based agent to the native tool-calling API and achieves 72% success on end-to-end web penetration testing |

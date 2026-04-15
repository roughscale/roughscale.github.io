---
layout: single
title: "Literature"
permalink: /literature/
author_profile: true
toc: true
toc_sticky: true
toc_label: "Contents"
---

This page tracks research related to the use of AI for offensive cybersecurity.

The focus of this research programme is understanding the **evolution of automated pentesting agents**, particularly how architectural changes and advances in AI models influence capability.

---

## Reinforcement Learning for Adversary Simulation

Early work explored modelling attacker behaviour using reinforcement learning. In these systems, penetration testing is treated as a sequential decision-making problem where an agent learns attack strategies through interaction with an environment.

Research environments supporting this work include:

- **NaSIM**
- **CyberBattleSim**
- **CybORG**
- **CyGIL**

These systems demonstrated the potential for automated adversary simulation but also exposed practical challenges when scaling reinforcement learning approaches to complex environments.

My earlier research explored reinforcement-learning-based adversary simulation in environments closer to real systems, highlighting challenges around scalability, observation representation, and environment complexity.

These challenges motivate the exploration of newer approaches based on large language models.

---

## LLM Pentesting Agents

The emergence of large language models created new opportunities to automate aspects of penetration testing by combining LLM reasoning with tool execution and interaction with external systems.

Early work such as **PentestGPT** demonstrated the potential for large language models to assist penetration testers, but relied on human interaction.

Later research began exploring fully autonomous agents capable of interacting directly with target environments. Early systems such as **HackingBuddyGPT** and **PenHeal** demonstrated this capability but were evaluated on limited or non-standardised environments.

The **CyBench** paper represents a convenient starting point in the research by providing a reproducible framework for evaluating autonomous LLM pentesting agents across multiple vulnerable targets and model architectures.

Over the last two years, a growing ecosystem of LLM-driven pentesting agents has emerged in academic research. These include:

- AutoPT
- RapidPen
- BountyBench
- PentestAgent
- VulnBot
- xOffense
- TermiAgent
- HackSynth
- MAPTA
- Cochise
- CAI
- PentestGPTv2

In parallel with academic research, a growing number of pentesting agents have appeared in the wider ecosystem. Examples include projects such as **Raptor**, which explore similar ideas within the open source community, as well as **XBOW**, perhaps one of the first commercial offerings of this kind of capability.

---

## Benchmarks and Evaluation

As LLM-driven pentesting agents emerge, benchmark environments are becoming increasingly important for evaluating capability.

Benchmarks that have provided structured environments for testing agent behaviour in offensive security tasks include:

- **PentestGPT**
- **CyBench**,
- **XBOW validation benchmark**
- **BountyBench**
- **CAI**
- **GOAD** (Game of Active Directory)

Reproducing and analysing these benchmark systems forms a key part of this research programme.

---

## Current Focus

The current research program focuses on:

- reproducing pentesting-agent research systems
- evaluating modern models in benchmark environments
- analysing architectural differences between agents
- studying how agent capability evolves across model generations



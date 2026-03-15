---
layout: page
title: Literature
permalink: /literature/
---

# Literature

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

## LLM Pentesting Agents

The emergence of large language models created new opportunities to automate aspects of penetration testing by combining LLM reasoning with tool execution and interaction with external systems.

One of the earliest widely discussed systems was **PentestGPT**, which demonstrated that language models could guide penetration testing workflows by reasoning about reconnaissance results and orchestrating external security tools.

Since then, a growing ecosystem of LLM-driven pentesting agents has emerged in both academic research and open-source development. Representative systems from this evolving landscape include:

- CyBench
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

In parallel with academic research, a growing number of open-source pentesting agents have appeared in the wider ecosystem. Examples include projects such as **Raptor**, which explore similar ideas outside of formal research publications.

---

## Benchmarks and Evaluation

As LLM-driven pentesting agents emerge, benchmark environments are becoming increasingly important for evaluating capability.

PentestGPTv2 evaluated agents using several benchmark environments, including:

- the **PentestGPT benchmark**
- the **XBOW validation benchmark**
- **GOAD** (Game of Active Directory)

Other research efforts have introduced additional evaluation frameworks such as **CyBench**, **BountyBench** and **CAI**, which aim to provide structured environments for testing agent behaviour in offensive security tasks.

Reproducing and analysing these benchmark systems forms a key part of this research programme.

---

## Current Focus

The current research program focuses on:

- reproducing pentesting-agent research systems
- evaluating modern models in benchmark environments
- analysing architectural differences between agents
- studying how agent capability evolves across model generations



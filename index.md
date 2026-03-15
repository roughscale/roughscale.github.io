---
layout: home
title: AI Offensive Security Research
---

# AI Offensive Security Research

This site documents my independent research into the use of AI for offensive cybersecurity.

My current work explores the use of AI to perform offensive cybersecurity tasks such as penetration testing, vulnerability discovery, and adversarial security operations.

A central theme of this research is the study of:

## Evolution of LLM Pentesting Agents: Architecture and Capability Analysis

Recent research has introduced a variety of AI systems capable of performing structured offensive security tasks. These systems combine large language models with agent architectures that enable reasoning, planning, tool use, and interaction with external environments.

However, the rapid pace of change in both models and agent frameworks makes it difficult to understand how capabilities are evolving.

This research therefore focuses on:

- reproducing and analysing existing pentest-agent research
- evaluating modern models in established benchmark environments
- studying how agent architecture influences capability and reliability
- comparing behaviour across different model and architecture combinations

The goal is to better understand how AI-driven agents perform when operating in realistic offensive cybersecurity contexts.

---

## Research Areas

### Adversary Simulation

My earlier research focused on automated adversary simulation using reinforcement learning, modelling attacker behaviour in distributed systems and cyber environments.

This work explored how learning-based agents could discover attack strategies and navigate complex systems without relying on predefined playbooks.

This research forms the foundation for my current work exploring AI-driven offensive security.

### LLM Pentesting Agents

Recent advances in large language models have enabled the development of agents capable of performing structured penetration testing tasks.

These systems combine language models with agent frameworks that support reasoning, tool use, planning, and interaction with external environments.

My research explores how these agents behave when applied to realistic offensive security workflows.

### Benchmarking AI Cyber Capability

Understanding the capabilities of AI-driven pentesting agents requires reproducible evaluation environments.

Current work therefore focuses on reproducing and analysing benchmark environments and evaluating how modern models perform across generations of AI systems.

### Architecture and Capability Analysis

Beyond model capability alone, agent architecture plays a critical role in determining performance.

This research examines how architectural design choices — including tool orchestration, planning strategies, memory mechanisms, and multi-agent coordination — influence the effectiveness and reliability of AI-driven security agents.

---

## Research Areas

### Adversary Simulation

My earlier research explored automated adversary simulation using deep reinforcement learning. This work modelled attacker behaviour as a sequential decision-making process in which an agent learns to navigate and exploit a target environment through interaction.

The research built upon existing cyber-security simulation environments such as CybORG and investigated the effectiveness of value-based and policy-based DRL algorithms for modelling adversarial behaviour.

This work demonstrated the potential for reinforcement learning to automate aspects of penetration testing and adversary simulation, while also highlighting a number of practical limitations in scaling these approaches to more realistic environments.

### Scaling Challenges in RL-based Pentesting

Applying reinforcement learning to penetration testing introduces several challenges that become more pronounced as environments grow in complexity.

These include:

- large and dynamic state spaces
- expanding action spaces representing exploit capabilities
- partial observability of real-world systems
- the difficulty of representing complex system observations as feature vectors

These limitations make it difficult to scale purely RL-based approaches to realistic offensive security scenarios.

### LLM Pentesting Agents

Recent advances in large language models provide a different approach to many of the representation and reasoning challenges encountered in reinforcement-learning-based pentesting systems.

LLM-based agents are capable of interpreting complex observations, reasoning about system behaviour, and orchestrating external tools. This makes them well suited to modelling penetration testing workflows that involve reconnaissance, vulnerability discovery, and multi-step exploitation strategies.

My current research explores how these LLM-based agents perform when applied to structured offensive security tasks.

### Benchmarking AI Cyber Capability

Understanding the capabilities of AI-driven pentesting agents requires reproducible evaluation environments.

Current work therefore focuses on reproducing and analysing benchmark systems such as CyBench and related research frameworks in order to evaluate how modern models perform across different offensive security tasks.

### Architecture and Capability Analysis

Beyond model capability alone, agent architecture plays a critical role in determining performance.

This research examines how architectural design choices — including tool orchestration, planning strategies, memory mechanisms, and multi-agent coordination — influence the effectiveness and reliability of AI-driven security agents.

## Current Research Programme

The current research programme focuses on reproducing and analysing key research systems in the emerging field of AI-driven offensive security.

This includes:

- reproducing pentest-agent research systems
- evaluating modern models within existing benchmark environments
- analysing architectural differences across agent implementations
- studying the trajectory of capability across model and architecture generations

This work is intended to build a deeper understanding of how autonomous offensive security agents are evolving and where their practical limits currently lie.

---

## Experiments

Experiment artifacts are published as separate repositories.  The details of these experiments will be listed here.

---

## About

I am a security engineer with a long-standing research interest in adversary simulation, offensive security, and the application of AI to cybersecurity problems.

This site serves as the public research index for my ongoing work in AI Offensive Security Research.


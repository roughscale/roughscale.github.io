---
layout: page
title: Reinforcement Learning Adversary Simulation
permalink: /adversary-simulation-drl/
---

# Reinforcement Learning Adversary Simulation

My earlier research investigated automating adversary simulation using deep reinforcement learning.

This work explored whether reinforcement learning agents could learn attack strategies in realistic network environments and perform penetration testing tasks without relying on predefined attack playbooks.

The research was completed as part of my MSc in Information Security at Royal Holloway, University of London.

---

## Research Objective

The goal of the research was to evaluate whether reinforcement learning could be used to automate aspects of penetration testing in environments closer to real-world systems.

Most prior research in this area relied on simulated environments, which simplify many aspects of real infrastructure.

This research attempted to bridge that gap by evaluating RL agents in both **simulated and emulated environments**.

---

## Experimental Approach

The experiment followed a two-stage evaluation process.

1. Reinforcement learning agents were trained in the **CybORG simulated environment**.
2. Agents that successfully learned attack strategies in simulation were then evaluated against **emulated network environments**.

The advantage of this approach was that simulated environments provide an efficient method to train on large number of training steps over emulated environments.  

This approach also provided an opportunity to determine whether strategies learned in simulation could transfer to environments closer to real infrastructure.

---

## Environment

The experimental environment was based on the **CybORG penetration testing benchmark**.

The benchmark provides a reinforcement learning interface representing a network environment where an attacker agent performs actions such as reconnaissance, exploitation, and lateral movement.

The scenario used in the experiment consisted of:

- three interconnected networks
- five hosts
- an external gateway separating internal hosts from the attacker

The attacker agent operates from an external host and must discover and compromise hosts within the internal network.

The environment included both Linux and Windows systems to approximate realistic enterprise infrastructure.

---

## Emulated Infrastructure

While CybORG provided both a simulated and emulated environment for training agents, the public release does not include its full emulation capabilities.

To support testing in more realistic environments, I implemented an **emulation controller** enabling reinforcement learning agents to interact with real systems.

The emulated environments were deployed in **AWS using Terraform**.

Agent actions were executed through the **Metasploit Framework**, with a custom Python interface allowing the learning agent to control Metasploit via its RPC API.

This allowed reinforcement learning agents to perform real penetration testing actions including exploitation, pivoting, and post-exploitation activities.

---

## Algorithms Evaluated

The research evaluated several reinforcement learning approaches commonly used in cybersecurity research.

These included:

- **DQN** (Deep Q-Network)
- **DRQN** (Deep Recurrent Q-Network)
- **PPO** (Proximal Policy Optimization)
- **Recurrent PPO**

These algorithms were selected to compare:

- value-based vs policy-based learning approaches
- fully observable vs partially observable environments

---

## Observations

The experiments demonstrated that reinforcement learning agents are capable of learning effective attack strategies in simulated penetration testing environments.

However, the research also highlighted a number of practical challenges when attempting to scale these approaches to realistic environments.

These included:

- large state and action spaces
- difficulty representing complex system observations as structured feature vectors
- the increasing complexity of modelling real-world systems within reinforcement learning environments

---

## Relationship to Current Research

These challenges form part of the motivation for my current research into **LLM-based pentesting agents**.

Large language models provide a different approach to representing and reasoning about complex environments.

Rather than encoding observations as structured feature vectors, LLM-driven agents can interpret complex system outputs directly and reason about attack strategies using natural language representations.

My current work explores how these agents behave in offensive cybersecurity environments and how different agent architectures influence their capabilities.


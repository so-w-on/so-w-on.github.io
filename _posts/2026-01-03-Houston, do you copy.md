---
title: Houston, do you copy?
author: Mouna
date: 2026-01-03 10:00:00 +0800
categories: [2 Project-- Secure Communication]
pin : true
---
# Introduction
In this article, I will do an analysis of a fictional UAV communications link, specifically Flight Controller <-> Ground Control Station (GCS).
Until I manage to make my own small personal UAV to tinker with, I will use Damn Vulnerable Drone project (DVD) to validate some of the concepts discussed throught the article. DVD will be considered a reference and illustration, I will not be doing a demo / walkthrough of the tool (Feel free to check the github page of the project for that <https://github.com/nicholasaleks/Damn-Vulnerable-Drone>).

# Scope
A scope well-defined is a project well-grasped.
In this article, I will focus on the link: Flight Controller <-> Ground Control Station (GCS)

# Fictional Architecture
We will give more details in the sub-sections below, but it can mostly be summed up in the following diagram:

┌──────────────────────────┐
│  Ground Control Station  │
│  (Operator Interface)    │
│                          │
│  - Mission planning      │
│  - Mode commands         │
│  - Telemetry display     │
└───────────▲──────────────┘
            │
   Command / Telemetry
        (UDP)
            │
┌───────────┴──────────────┐
│   Communication Link     │
│   (IP-based radio)       │
│                          │
│  Assumptions:            │
│  - Unreliable            │
│  - Low latency           │
│  - No authentication     │
└───────────▲──────────────┘
            │
┌───────────┴──────────────┐
│     Flight Controller    │
│                          │
│  - Control loops         │
│  - State machine         │
│  - Failsafe logic        │
│                          │
│  Trust assumptions:      │
│  - Commands are valid    │
│  - Telemetry reflects    │
│    true internal state   │
└──────────────────────────┘

## Components:
Below are the different components in the system we are analyzing, their functionality, and their equivalent in the validation simulation.

| Component (fictional UAV)             | Functionality                                        | DVD equivalent        |
| :------------------------------------ | :----------------------------------------------------| :---------------------|
| Flight Controller                     | Execute control loops / Enforce flight mode          | MAVProxy/ArduPilotSITL|
| Radio Link                            | IP-based radio link / Assumed unreliable, low latency| UDP localhost         |
| Ground Control Station                | Operator interface / Send commands, receive telemetry| QGroundControl        |
| Attacker                              | tcpdump, scapy                                       | Analysis host         |

## Assumptions:
- UAV can fly autonomously for short periods
- GCS sens mission updates & commands
- Telemetry is continuous
- Safety logic relies on communciation integrity.
- [Security] GCS is trusted
- [Security] Commands are not authenticated : No independent authentication channel
- [Security] Telemetry integrity is not verified
- [Security] Freshness relies on timing only

## Protocol definition:
- Heartbeat: periodic "I'm alive" signal.
- Telemetry message content: position, status, mode, battery, flags??.
- Command message content: mode change, mission update, waypoint update, trigger actions (RTL, land)
- Properties: Binary, stateless per message, UDP transport, no cryptographic protection.

# Normal behavior
"Yes, Houston, we copy"
- Heartbeat interval: small, stateless, periodic
- Telemetry rate: used for situational awareness, operator decision-making, implicit state synch
- Timeout thresholds: *on the way!*
- Failsafe triggers (RTL, hover, land) :
- Critical messages: *on the way!*
- Expected latency: *on the way!*

# Observation phase
The goal in this phase is to establish a baseline communication behavior between the Flight Controller (FC) and the Ground Control Station (GCS).

## Passive traffic capture method:
An efficient attacker knows to do nothing and only listen at the first phase. We will do the same here.
*on the way!*

## Message identification & classification by type:
From the traffic capture, we can determine some heartbeat pattern:
*on the way!*
Let's look at this command structure extracted from the traffic capture:
*on the way!*

## UAV state reconstruction (Use of Telemetry data):
Telemetry messages carry all the necessary information on the systems state.
*on the way!*

# Cue: Failures
In this phase, we will introduce non-malicious failures and see how the system reacts. A robust UAV should be able to adapt to peculiar issues such as loss of communication with GCS, a desynchronisation, ... This is typically a test of the system's safety mechanisms against possible hazards.
*on the way!*

# Cue: Adversarial attacks
In this phase, we will go on full attacker mode and test **A replay of valid commands** that were previously captured.
explain: why it works (or why it doesn't), what assumption is violated and what trust was misplaced, physical consequence.
Why did the replay attack work?
*on the way!*
What weas the impact?
*on the way!*

# Physical & Safety Impact
*on the way!*
Following are some threat scenarios, for which we will study the impact across all categories (Safety, Operational, Financial, Privacy) and the feasibility (hence attributing a *risk score*)

| Item                    | STRIDE                 | Threat scenario                                                                                                              |
| :---------------------- | :----------------------| :----------------------------------------------------------------------------------------------------------------------------|
| GCS Commands to the UAV | Spoofing               | Attacker injects commands impersonating the GCS (No sender authentication), giving them unauthorized control over it         |
| GCS Commands to the UAV | Tampering              | Attacker replays older commands, causing unintended actions from the system                                                  |
| FC Telemetry to the GCS | Information Disclosure | Attacker eavesdrops on teh system and gets real-time position, flight mode, and system health                                |
| FC Telemetry to the GCS | Tampering              | Attacker tampers with Telemetry messages, causing the GCS to receive incorrect system state and make unsafe control decisions|
| Heartbeat               | Denial of Service      | Attacker drops heartbeat messages which triggers failsafe behavior                                                           |

# Defensive engineering:
*on the way!*
As a followup on the previous section, it's important to know the existence of some cybersecurity mechanisms which will greatly minimize the feasibility of the previous threat scenarios.

*on the way!*
When dealing with security of Cyber-Physical Systems, it's important to understand that one can't and shouldn't dodge trade-offs forever. Even in the most used risk assessment models, we always have a choice of "accepting the risk" as an arbitration between system architecture, expensive deisgn changes, and risk impact and feasibility.


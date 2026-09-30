# Netmon — Network Monitoring & Root-Cause Analysis

[![Python](https://img.shields.io/badge/python-3.12-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](#license)
[![Platform](https://img.shields.io/badge/platform-Ubuntu%2022.04%2B-orange.svg)](#requirements)
[![Status](https://img.shields.io/badge/status-active-success.svg)](#)

A Python-based network monitoring and root-cause analysis (RCA) tool for GNS3
labs. Netmon polls Cisco routers and switches over SNMP and ICMP, detects faults
with a deterministic rule engine, enriches each incident with an AI-generated
diagnosis, and delivers evidence-packaged email alerts to the correct team.

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running](#running)
- [Detection Rules](#detection-rules)
- [AI-Assisted Diagnosis](#ai-assisted-diagnosis)
- [Email Alert Format](#email-alert-format)
- [Database](#database)
- [Lab Network](#lab-network)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Features

- 🔄 **Continuous polling** — configurable interval (default 20 s), parallel
  across all devices.
- 📡 **Multi-protocol collection** — ICMP reachability plus SNMP v2c walks.
- 🧭 **Multi-path reachability** — each device can be polled at several
  independent addresses to distinguish device-level failures from link failures.
- 📏 **Rule-based RCA** — deterministic fault detection with confidence
  scoring.
- 🤖 **AI-assisted diagnosis** — optional Groq LLM second opinion.
- 📦 **Evidence packaging** — every alert includes the last two readings and
  the reasoning that triggered it.
- 📧 **Team-based alert routing** — different fault categories go to the
  right operational team.
- 💾 **SQLite storage** — full history of metrics and incidents.
- 🖥️ **Systemd-ready** — runs as a background service.

---

## Architecture

![Uploading 42c0e9a2-0619-427b-9dda-09075568f33f.png…]()

| Module | Responsibility |
|--------|---------------|
| `collector.py` | ICMP ping, SNMP v2c walks (pysnmp 7.x asyncio API) |
| `analyzer.py` | Rule engine, cross-device correlation, AI enrichment |
| `ai_analyzer.py` | Groq API integration |
| `notifier.py` | SMTP delivery with team routing |
| `database.py` | SQLite store for metrics and incidents |
| `main.py` | Parallel orchestration loop |

**Data flow:** collect → analyze → correlate → persist → notify

---

## Requirements

- Ubuntu 22.04+ (or any Linux with Python 3.12)
- Python 3.12
- `ping` and `snmp` utilities
- Network reachability to each managed device
- **Groq API key** — free tier available at
  [console.groq.com/keys](https://console.groq.com/keys)
- **Gmail App Password** — free, from
  [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
  (requires 2-Step Verification)

---

## Installation

```bash
# 1. System packages
sudo apt update
sudo apt install -y python3 python3-venv iputils-ping snmp

# 2. Virtual environment
python3 -m venv ~/netmon-venv
source ~/netmon-venv/bin/activate

# 3. Python dependencies
pip install --no-cache-dir \
    pysnmp pyasn1 netmiko paramiko \
    PyYAML requests groq openai

# 4. Clone or copy the project
mkdir -p ~/netmon
cd ~/netmon
# ... place collector.py, analyzer.py, ai_analyzer.py,
#     notifier.py, database.py, main.py, config.yaml here

# 🔐 Claude Code Security vs. Traditional SAST
> A Comparative Analysis of AI-Based and Rule-Based Vulnerability Detection

**Author:** Mariam Qadeem (BITF22M006) — PUCIT, Lahore, Pakistan

![Research](https://img.shields.io/badge/Type-Research%20Paper-blueviolet)
![Tool](https://img.shields.io/badge/Tool-Semgrep%201.159.0-orange)
![Target](https://img.shields.io/badge/Target-vulpy%20%28Flask%2FPython%29-blue)
![Date](https://img.shields.io/badge/Date-April%202026-green)

---
## 🙏 Acknowledgment

This work was conducted under the guidance of **Sir Huzaifa Nazir**, whose mentorship, technical expertise, and continuous feedback significantly contributed to the development and direction of this research.
---

## 🚨 The Problem

Traditional SAST tools sound great on paper. In practice:

| Metric | Value | Source |
|--------|-------|--------|
| Combined detection rate (4 tools, 502 Java CVEs) | **38.8%** | Bennett et al., 2024 |
| Average false positive rate | **67%+** | Secuarden, 2025 |
| Semantic bug detection | **0%** | Structural impossibility |

> **The hard truth:** Logic-level bugs have no syntactic trace.
> No rule can match code that was *never written*.

---

## ⚙️ How Each Tool Works

| | SAST | Claude Code Security |
|--|------|----------------------|
| **Approach** | Pattern-matching on ASTs | LLM reasoning over full codebase |
| **Best for** | SQL injection, hardcoded secrets | Auth bypass, IDOR, logic flaws |
| **Speed** | Fast (~10s) | Slow (cloud API) |
| **Deterministic** | ✅ Yes | ❌ No |
| **Audit-ready** | ✅ Yes | ❌ No |

---

## 🧪 Experiment

Ran **Semgrep 1.159.0** on [`vulpy`](https://github.com/fportantier/vulpy) — a real, independently documented vulnerable Flask app used in OWASP training. Ground truth was defined *before* running any tool to eliminate experimenter bias.

| Vulnerability | CWE | Class | Detected |
|--------------|-----|-------|----------|
| SQL Injection | CWE-89 | Syntactic | ✅ YES |
| Insecure Deserialization | CWE-502 | Syntactic | ✅ YES |
| Hardcoded Secret | CWE-798 | Syntactic | ✅ YES |
| XSS | CWE-79 | Syntactic | ❌ NO |
| Session Impersonation | CWE-287 | Semantic | ❌ NO |
| Auth Bypass (No Rate Limit) | CWE-307 | Semantic | ❌ NO |

**Result:** 50% overall TPR — 75% on syntactic bugs, **0% on semantic bugs**.

---

## 🤖 Claude Code — Key Findings

Independent evaluation by Semgrep Inc. on 11 real Python web apps (Sep 2025):

| Metric | Value |
|--------|-------|
| Overall TPR | 14% |
| IDOR Detection | 22% |
| SQL Injection TPR | 5% |
| False Positive Rate | **86%** |

> ⚠️ **Non-deterministic:** Same code + same prompt produced **3 → 6 → 11** findings across three identical runs.
> Not suitable for compliance or audit workflows.

---

## 🛡️ CIA Triad — Claude Code's Own Risks

| | CVE | CVSS | Issue |
|--|-----|------|-------|
| 🔴 **Confidentiality** | CVE-2025-52882 | 8.8 | Local WebSocket, no origin validation — any webpage could read local files |
| 🟡 **Integrity** | CVE-2025-59536 | 8.7 | Prompt injection via code comments → missed bugs + RCE |
| 🟠 **Availability** | CVE-2026-21852 | 5.3 | Cloud-only; outage = unavailable; API key exfiltration risk |

---

## 🔁 Recommended Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYBRID SECURITY PIPELINE                     │
├──────────────┬──────────────────────────────────────────────────┤
│  Every       │  🔍 SAST Scan                                    │
│  Commit      │     Semgrep / SonarQube / CodeQL                 │
│              │     Fast · Deterministic · Syntactic bugs        │
├──────────────┼──────────────────────────────────────────────────┤
│  PR Gate     │  🤖 LLM Post-Filter                              │
│              │     Cuts false positives: 68% → 6%               │
├──────────────┼──────────────────────────────────────────────────┤
│  Pre-Release │  🧠 Claude Code Security                         │
│              │     Semantic audit · Logic-level bug detection    │
├──────────────┼──────────────────────────────────────────────────┤
│  All Patches │  👤 Human Review  (mandatory, always)            │
└──────────────┴──────────────────────────────────────────────────┘
```

| Configuration | True Positives | FP Rate | Coverage |
|---------------|:--------------:|:-------:|:--------:|
| SAST only | 39 | 68% | 39% |
| SAST + LLM Filter | 37 | 6% | 37% |
| **SAST + LLM + Claude Code** ⭐ | **52** | **6%** | **52%** |

---

## ✅ Key Takeaway

SAST and Claude Code Security target **different bug classes** — use them together, not as alternatives.

- 🟢 **SAST** → fast, reliable, audit-ready, catches syntactic bugs on every commit
- 🔵 **Claude Code** → catches what SAST structurally *cannot*, but carries 86% FPR, non-determinism, and real CVEs
- 🔴 **Human review** → remains essential, always

> *Neither tool is enough alone. The hybrid pipeline is the answer.*

---

# FloydIA Autonomous Agent Skills Catalog (v28-MEM, rev. 2026-10-13)

[![CI](https://github.com/floydiamarkv-byte/floydia-agent-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/floydiamarkv-byte/floydia-agent-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A production-grade, deterministic agentic skills ecosystem designed for autonomous coding environments, multi-model harnesses, homelab orchestration, and evidence-based verification.

## 🏛️ Architecture Overview

The FloydIA Agent OS organizes execution, verification, and governance into modular skills:

- **Governor & Routing**: `f-harness` v4 (unified: SRE operations + mode selector + workflow methodology, with `references/` progressive disclosure), `f-orquestador-floydia` (architect), `f-mejora-prompt`, `f-menu-prompts`.
- **Deep Research & Intelligence**: `f-investiga` (two modes: deep dive + swarm, referencing the canonical `ENJAMBRE_5_MOTORES_SSOT`), `f-opti-notebooklm`.
- **System Reliability Engineering (SRE)**: `f-master-update`, `f-evidence-engine`, `f-handon-handoff`, `f-check-environment`.

> 2026-10-13 consolidation: merged `f-harness-select`/`f-harness-workflow` into `f-harness`; removed previously superseded skills (`f-router`, `f-harness-router`, `f-update-floydia`, `f-opti-floydia`) and archived ones (`f-policy-engine`, `f-audit-web`). Swarm engine definitions are SSOT-referenced, never embedded.

## 🛡️ Security & Privacy

All skills adhere to the zero-leakage security contract:
- Zero hardcoded API keys or credentials.
- Fully portable relative paths and environment abstraction (${WORKSPACE_ROOT}).
- Immutable append-only memory governance.

## 📜 License

MIT License. Developed by FloydIA Systems.

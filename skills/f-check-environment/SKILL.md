---
name: f-check-environment
description: "Alias canónico y comprobador rápido de entorno para f-opti-floydia (SRE Governor). Audita en vivo hardware, red, Proxmox, MikroTik, agentes, latencias y cuellos de botella sin riesgo de congelamientos. USAR cuando el usuario invoque /f-check-environment, /check-environment, /sre-probe o /f-check-enviroment-floydia. NO USAR para desarrollo frontend ni edición de código de clientes."
version: 4.1.1
triggers:
  - /f-check-environment
  - /f-check-enviroment-floydia
  - /check-environment
  - /sre-probe
  - check environment
  - auditar entorno
---

# Skill: Check Environment FloydIA (`f-check-environment` v4.1.1)

> **Nota de Arquitectura**: Este skill es el alias canónico de acceso rápido para **`f-opti-floydia`** (SRE Governor & Optimizador de Ecosistema). Para evitar divergencias de versión y deuda técnica, toda la suite de scripts, guardrails y menús operativos se ejecutan desde el SSOT en `.agents/skills/f-opti-floydia/`.

---

## 🎯 1. Despacho Directo al SRE Governor

Al invocar `/f-check-environment` o `/check-environment`, el agente **DEBE cargar y ejecutar de inmediato el flujo del SRE Governor (`f-opti-floydia`)**:

1. **Opción Rápida por Defecto**: Ejecutar la sonda de telemetría:
   ```bash
   python3 .agents/skills/f-opti-floydia/scripts/ecosystem_probe.py
   ```
2. **Presentación de Menú**: Si se requiere selección interactiva, desplegar el menú interactivo completo de `f-opti-floydia` (§1 en [`f-opti-floydia/SKILL.md`](file:///.agents/skills/f-opti-floydia/SKILL.md)).

---

## 📦 2. Scripts Compartidos (Symlink)

Los scripts operativos residen en `.agents/skills/f-opti-floydia/scripts/`:
- `ecosystem_probe.py`: Telemetría completa de hardware, red y contenedores.
- `bottleneck_analyzer.py`: Diagnóstico de PSI, CPU throttling, I/O wait y memoria.
- `agent_health_check.py`: Auditoría de procesos huérfanos y paridad de symlinks.
- `dual_prompt_generator.py`: Generador de Prompts A y B para Claude / ChatGPT Web.

---

## 🔄 3. Compatibilidad Multi-Agente

| Agente | Soporte | Notas |
|---|:---:|---|
| **Antigravity IDE** | ✅ Completo | Enrutamiento directo al menú de `f-opti-floydia` |
| **OpenCode CLI** | ✅ Completo | Ejecución de scripts CLI |
| **Hermes CLI** | ✅ Completo | Ejecución de scripts CLI |
| **DeepSeek Harness** | ✅ Completo | Auditoría SRE y telemetría |

---
name: f-harness-router
description: "Orquestador de prioridades por dificultad del Harness FloydIA: clasifica cada tarea/subtarea (R0..R3 -> tier flash/medio/razonamiento), elige el motor concreto (Kimi K3, AGY, V4 Flash, Pool Free) según dificultad + modo activo + disponibilidad real del proxy NIM, y ejecuta/delega a coste $0.00. USAR cuando el usuario invoque /f-harness-router, /router-harness, /orquestar, /dispatch, o pida que el agente decida qué motor usar según la dificultad de la tarea y organice subagentes. NO USAR para cambiar el modo/marcha del harness (eso es f-harness-select) ni el default_model de la sesión."
version: 1.0.0
triggers:
  - /f-harness-router
  - /router-harness
  - /orquestar
  - /dispatch
  - decide qué motor usar
  - elige el motor según dificultad
  - orquesta subagentes por prioridad
  - qué motor uso para esta tarea
---

# Skill: f-harness-router — Orquestador por Dificultad v1.0

> **Propósito**: Dar al arnés el comportamiento completo que el usuario espera:
> **decidir el motor según la DIFICULTAD de cada tarea** y organizar múltiples delegaciones,
> en lugar de usar siempre el primer eslabón de la cadena.
>
> **Motor de decisión**: `SCRIPTS/harness_dispatch.py` (determinista, local, $0.00, sin red salvo `--execute`).

---

## 🧠 1. Cómo funciona (modelo mental)

```
Tarea del usuario
      │
      ▼
agent_router.py  →  dificultad: R0/R1/R2/R3 → tier: flash / medio / razonamiento
      │
      ▼
harness_mode.json →  cadena de respaldos del modo activo (1..5)
      │
      ▼
proxy NIM /health →  disponibilidad real (pasiva, $0.00)
      │
      ▼
MOTOR ELEGIDO = mejor encaje tier↔cadena
   - tier razonamiento → Kimi K3 (cabeza de cadena)
   - tier medio        → AGY / V4 Flash
   - tier flash        → Pool Free
Si el elegido no está disponible → DESCENDER por la cadena (fallback).
```

**Regla de oro** (explicar siempre si hay duda):
- El **modo** (f-harness-select) define el ORDEN DE RESPALDO.
- El **tier** (este skill) define el PUNTO DE ENTRADA según dificultad.
- Coste siempre $0.00: NIM cuentas C1/C2/C7/C9, OpenRouter `:free`, AGY Antigravity.

### 🐝 Estrategia Enjambre Multiconsulta (Consenso Colegiado 5 Motores)
Para tareas de arquitectura, refactors críticos o diagnósticos de alta incertidumbre, el flag `--enjambre` (`--swarm`) activa el despacho simultáneo a los 5 motores (Kimi K3, DeepSeek-R1, Gemini 3.8 Flash, Qwen 2.5 Coder y AGY/Local). El orquestador unifica las respuestas en una matriz de consenso sin coste alguno de inferencia tarifada.

---

## 🎯 2. Flujo operativo del agente

### Caso A — El usuario pregunta "¿qué motor usarías para <tarea>?"
1. Ejecutar:
   ```bash
   python3 /SCRIPTS/harness_dispatch.py "<tarea>" --explain < /dev/null
   ```
2. Mostrar la decisión literal (dificultad, modo, disponibilidad, motor elegido).
3. **No ejecutar nada más** salvo que el usuario lo pida.

### Caso B — El usuario pide orquestar un trabajo multi-subtarea
1. Descomponer el trabajo en subtareas concretas.
2. Para **cada subtarea**, ejecutar `harness_dispatch.py "<subtarea>" --json` y registrar el motor elegido.
3. Presentar el **plan de orquestación** (tabla: subtarea → dificultad → motor) y pedir confirmación.
4. Ejecutar cada subtarea con su motor:
   - Motor **nim / or_free** → delegar con `subagent` usando override de modelo equivalente, o ejecutar el `comando` JSON devuelto (curl al proxy `http://127.0.0.1:8088/v1/chat/completions`).
   - Motor **agy** → `bash deepseek-antigravity-bridge/scripts/agy_delegate.sh "<subtarea>" "$PWD" 60s < /dev/null` bajo `safe_run_cli.sh --timeout 100`.
5. Recolectar resultados y sintetizar. Reportar cuántos motores se usaron y el coste ($0.00).

### Caso C — Ejecución directa de una sola tarea
```bash
python3 SCRIPTS/harness_dispatch.py "<tarea>" --execute < /dev/null
```
Verificar `exec.exit == 0`; si falla, mostrar `stderr_tail` y no reintentar más de 2 veces (anti-loop).

---

## 🛡️ 3. Reglas duras

- **$0.00 absoluto**: prohibido enrutar a modelos de pago; si la clasificación sugiere tier inexistente en la cadena, degradar a Pool Free.
- **Confirmación humana** antes de ejecutar lotes de subtareas (Caso B, paso 3).
- Toda llamada externa con `< /dev/null` y timeout; delegaciones AGY siempre bajo `safe_run_cli.sh` (anti-hang).
- Si el proxy NIM está caído (`status != ok`), marcar motores `nim` como no disponibles y dejar que `pick_engine` descienda automáticamente — no improvisar rutas.
- No editar `harness_mode.json` desde este skill (eso es territorio de `f-harness-select`).
- No modificar `agent_router.py` ni `model_router.md` sin aprobación (son SSOT compartidos, R2).

---

## ✅ 4. Criterio de éxito

- Decisión emitida con evidencia JSON literal (dificultad + modo + disponibilidad + motor).
- En orquestación: plan aprobado, cada subtarea ejecutada con su motor, síntesis final con conteo de motores y coste $0.00.
- Anti-loop: máximo 2 reintentos por subtarea; fallo persistente se documenta y se consulta al usuario.

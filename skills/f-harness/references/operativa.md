
# Skill: Harness FloydIA v2 — Manual Operativo & SRE (`f-harness` v3.1.0)

> **Propósito**: Manual operativo y centro de comando del Harness FloydIA v2 bajo el **Protocolo v28**. 
> Centraliza el enrutamiento determinista por dificultad (4 tiers / 11 motores a $0.00), la rotación LRU de cuentas, la telemetría pasiva y el diagnóstico/rescate de caídas en DeepSeek Harness (DSH).

---

## 🎯 1. Menú Interactivo Menu-First

Al invocar `/f-harness`, `/harness`, `/model-router` o ante un reporte de fallo en motores o DSH, el agente **DEBE responder de inmediato presentando este menú Bento interactivo**:

```markdown
╔════════════════════════════════════════════════════════════════════════════════╗
║             🛸 F-HARNESS — CENTRO DE CONTROL DE MOTORES & DSH v3.1            ║
║                     Ecosistema FloydIA — Protocolo v28                         ║
╚════════════════════════════════════════════════════════════════════════════════╝

Selecciona una acción respondiendo con el número correspondiente:

1️⃣ 🧭 **Inspección de Enrutamiento & Cadenas Activas**
   ↳ Muestra los 4 tiers (dirección, razonamiento, medio, flash), orden de fallback
     y motores asignados (`python3 SCRIPTS/harness_dispatch.py --explain`).

2️⃣ 🩺 **Harness Doctor & Diagnóstico Pasivo de Salud ($0.00)**
   ↳ Valida consistencia de `~/.dsh/settings.yaml`, catálogo NIM vivo (`:8088/v1/models`),
     tombstones y sintaxis YAML con `python3 SCRIPTS/harness_doctor.py`.

3️⃣ 📊 **Telemetría Pasiva & Consumo de Cuentas (C1..C9 / Cloudflare)**
   ↳ Consulta el estado de cuotas de tokens, cooldowns LRU (`.session/pool_state.json`)
     y consumo diario de neuronas Cloudflare (`python3 SCRIPTS/pool_telemetry.py`).

4️⃣ 🚑 **Diagnóstico & Recuperación Anti-Caídas de DeepSeek Harness (DSH)**
   ↳ Rescata DSH ante fallos de boot (`settings.yaml` roto por sed, IDs huérfanos o caídas
     del proxy), aplicando sustitución canónica y prueba de humo (`dsh --dump-config`).

5️⃣ 🔄 **Reconfiguración Segura de la Cadena de Motores**
   ↳ Edita de forma determinista la cadena SSOT con respaldo automático
     (`python3 SCRIPTS/harness_chain.py <nueva_cadena.json>`).

6️⃣ 🛡️ **Suite Integral de Verificación y Auditoría del Harness (Protocolo v29)**
   ↳ Ejecuta la suite completa de 5 fases: Turno 0 + MCPs, Sonda de APIs + Dashboard HTML,
     Compilación AST, Enjambre Anclado con Citas (`--swarm --file`), y Registro Append-Only
     (`python3 SCRIPTS/run_harness_verification.py`).

0️⃣ 🚪 **Salir**
```

---

## 🧠 2. Modelo Mental: 4 Tiers, Entrada por Dificultad

El harness NUNCA hace fallback "a ciegas". Clasifica la tarea y entra por su tier; solo desciende dentro de la cadena si el motor elegido falla.

| Tier | Criterio de Entrada | Cadena de Motores (en orden) |
|---|---|---|
| **direccion** (duda/consenso) | Ambigüedad o arbitraje de arquitectura | `Union Alpha` (or_free) ➔ si falla, pool flash |
| **razonamiento** | Tarea compleja / lógica profunda / LRM | `BlueMind Nemotron Nano` (~3 s) ➔ `Kimi K3` NIM (240 s) |
| **medio** | Código estándar / refactor / análisis | `EvoLink GPT-5.1` ➔ `ZenMux GLM-5.3-FlashX` (35 s) |
| **flash** (default) | Trivial / chat / formato / consultas rápidas | `Mistral Ministral 3B` ×6 ➔ `Z.AI GLM 4.5 Flash` ×6 ➔ `Gemini 3.1 Flash Lite` ➔ `CF GLM 4.7 Flash` ➔ `Pool Free` (OR :free) |

> [!NOTE]
> **Vault (nunca automático)**: Kimi directo y OpenRouter C9 únicamente con `--break-glass` o solicitud explícita del usuario.

---

## 🚑 3. Protocolo de Resolución de Caídas en DeepSeek Harness (DSH)

### Regla Permanente Anti-Rotura (#vetado):
> **SUSTITUIR el ID del modelo por el sucesor vivo; NUNCA borrar líneas sueltas con `sed` en `~/.dsh/settings.yaml`.**
> El borrado crudo de líneas de IDs elimina la clave `- id:`, dejando bloques con `name:` huérfano que provocan que `dsh web` aborte el arranque silenciosamente.

### Procedimiento de Rescate Determinista ante Caídas:
1. **Paso 1: Diagnóstico Pasivo**
   ```bash
   python3 SCRIPTS/harness_doctor.py
   ```
   Detecta automáticamente entradas sin `- id:`, tombstones y estado del proxy NIM.
2. **Paso 2: Backup Inmediato antes de Modificar**
   ```bash
   cp ~/.dsh/settings.yaml ~/.dsh/settings.yaml.bak.$(date +%Y%m%d_%H%M%S)
   ```
3. **Paso 3: Sustitución Limpia del Modelo Retirado**
   - Si un modelo fue descontinuado (ej. `deepseek-ai/deepseek-v4-flash-0731`), sustituir su identificador por el modelo activo sucesor (ej. `deepseek-ai/deepseek-v4.1-flash`) preservando las 4 líneas del bloque (`id`, `name`, `contextWindow`).
4. **Paso 4: Prueba de Humo Obligatoria (Smoke Test)**
   ```bash
   dsh --profile web --dump-config >/dev/null && echo "✅ Configuración DSH íntegra"
   ```
5. **Paso 5: Reinicio Limpio de DSH Web (si aplica)**
   ```bash
   bash SCRIPTS/launch-dsh-web-desktop.sh
   ```

---

## 📁 4. Archivos Vivos del Harness (SSOT)

| Archivo | Rol Operativo | Regla de Edición |
|---|---|---|
| `SCRIPTS/harness_mode.json` | Cadena de 11 motores (SSOT v3) | Editar ÚNICAMENTE vía `harness_chain.py` |
| `SCRIPTS/providers_registry.json` | Proveedores, base_urls, keys_env, cuotas | Manual con validación JSON |
| `SCRIPTS/harness_dispatch.py` | Dispatcher y motor de inferencia | Python compilado con `py_compile` |
| `SCRIPTS/harness_doctor.py` | Validador sintáctico, tombstones y DSH dump | Ejecución pasiva ($0.00) |
| `SCRIPTS/pool_call.py` | Invocador pool con rotación LRU y cooldowns | Respetar presupuesto Cloudflare |
| `SCRIPTS/pool_telemetry.py` | Estado por cuenta y neuronas Cloudflare | Solo lectura de métricas |
| `~/.dsh/settings.yaml` | Configuración bare-metal de DeepSeek Harness | Sustitución de IDs; nunca sed suelto |

---

## 🛡️ 5. Guardrails Inmutables & Token Economics

1. **Inferencia a Coste $0.00 Obligatoria**: Prioridad absoluta a tiers `:free` y créditos NIM incluidos. Prohibido ejecutar llamadas de pago para pruebas de conectividad o healthchecks.
2. **Probes Estrictamente Pasivos**: Las sondas de salud (`harness_doctor.py`) consultan únicamente `/v1/models` (metadatos sin coste) y sintaxis de disco.
3. **Prueba de Humo Previa a Cierre**: Ningún incidente de harness o settings se cierra sin verificar `harness_doctor.py` exit 0 y `test_harness_doctor.py` 4/4 OK.
4. **Escritura Segura de Archivos (Anti-Volcado Masivo)**: Prohibido volcar archivos o documentos extensos (>15-20 KB o >300 líneas) en un único tool call (`write_to_file` con `CodeContent` gigante). Todo documento o script extenso debe generarse por partes incrementales, modificarse quirúrgicamente o escribirse mediante scripts locales directos en Python o Bash para evitar saturación de buffer y caída del stream SSE.

---

## 🔬 6. Definición de Motores Post-Escaneo de APIs (Consulta Externa + Enjambre)

> **Cuándo**: tras un escaneo preliminar de APIs (`f-api-scan-harness` / sondas pasivas) que confirme claves vivas, NO se define el orden de motores por intuición. Se ejecuta este protocolo de **triple contraste**.

### 6.1 Flujo obligatorio
1. **Inventario local verificado**: proveedores/rotación desde `opencode.jsonc` + `/health` del proxy `:8088` + `SCRIPTS/providers_registry.json`. (Hoy: los 5 motores principales van por **NVIDIA NIM** vía proxy multi-cuenta `127.0.0.1:8088` — buckets C1/C2/C7/C9).
2. **Consulta externa deliberada**: enseñar a una IA externa (chat web Libre, humano en el bucle) el inventario y pedirle orden de inteligencia mayor→menor + capacidades exactas, usando el prompt 6.2.
3. **Enjambre propio**: mientras tanto, lanzar subagentes paralelos con webfetch a fuentes primarias (fichas NVIDIA/HF, leaderboards verificables: Artificial Analysis Intelligence Index, LMArena) y registrar citas URL+fecha.
4. **Cotejo de 3 vías**: local vs externa vs enjambre. Registrar disonancias (ej.: LMArena solo lista a Kimi K3; AA II sitúa Ultra > K3). La cadena final se fija por **evidencia + coste $0.00 + latencia medida**, no por un solo ranking.
5. **Persistencia**: ranking y decisiones a `memory-bank/journal/` (append-only) y, si cambia el orden, editar `SCRIPTS/harness_mode.json` SOLO vía `harness_chain.py`.

### 6.2 Prompt canónico para IA externa (copiar/pegar tal cual, sustituir `[INVENTARIO]`)

```text
Actúa como analista de modelos LLM. Tengo este inventario de motores con APIs válidas ya verificadas:

[INVENTARIO: lista de modelos + proveedor + contexto + coste]

Contexto de uso: los uso en un harness multi-modelo clasificando tareas por dificultad:
- "razonamiento" (lógica profunda, arquitectura, matemáticas),
- "medio" (código estándar, refactor, análisis),
- "flash" (trivial, formato, latencia <1s),
con fallback en cadena dentro del mismo tier y prioridad $0.00.

Te pido, con citas verificables (URL + fuente):
1. Ranking de mayor a menor INTELIGENCIA general (razonamiento+código) de los modelos del inventario con confidence por posición.
2. Capacidades exactas de cada modelo: parámetros (totales/activos si MoE), contexto real, modalidades, tool-calling, benchmarks clave.
3. Metodología y latencia típica reportada.
4. Discrepancias explícitas si distintas fuentes discrepan entre sí.
Responde en español, tablas preferidas. Marca como NO VERIFICADO lo que no tengas con fuente.
```

### 6.3 Semilla verificada 2026-10-12 (enjambre propio + 7 IAs externas cotejadas)
Orden inteligencia consensuado (Artificial Analysis Intelligence Index **v4.3.2** — NUNCA mezclar versiones del índice; los "48/36" de prensa/aggregadores son v4.1 y están rebasados):
**1. Kimi K3 (II=44, LMArena #9 Elo 1486 + #1 WebDev 1679) → 2. DeepSeek V4.1 Flash (II=39, Terminal-Bench rey 90.6) → 3. Nemotron 3 Ultra 550B (II=23) → 4. Nemotron 3 Super 120B (II=12.8) → 5. Nemotron 3 Nano Omni 30B (II≈10 estimado, sub-agente de percepción, no generalista)**.
Reparto por dominio (accionable para tiers): K3 gana razonamiento/ciencia/legal (HLE 46.9, GPQA 93.5, BrowseComp 91.2) → tier **razonamiento**; V4.1 Flash gana agéntico/terminal (Terminal-Bench 4.0 26.8 vs 12.6; Automation 68.9%) → tier **medio/agéntico**; **flash**: Nano Omni (latencia mínima + omni) o V4.1 Flash con reasoning effort=1.
Trampas operativas verificadas: (a) `reasoning_effort` de K3 en NIM por defecto es **max** → fijar `low` en tiers no-expertos; (b) ctx 1M de Ultra/Super requiere enablement — en NIM servido a 262k (coincide con opencode.jsonc); (c) parámetros ≠ inteligencia (Ultra 55B activos < V4.1 8-16B en II); (d) latencias citadas son de proveedores comerciales, NIM free ~40 RPM: medir TTFT propio.
Persistencia del poll: 7 respuestas externas en `${HOME}/Descargas/ww/` + enjambre propio; consenso 6/7 en K3#1, disidente duck.ai (Super#2).


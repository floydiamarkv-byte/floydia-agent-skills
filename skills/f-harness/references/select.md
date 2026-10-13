
# Skill: f-harness-select — Selector Menu-First del Modo de Harness v1.0

> **Propósito**: Permitir al usuario ver y cambiar la cadena de prioridades de motores del Harness FloydIA (qué modelo va primero, segundos y fallback) de forma interactiva, segura y verificable.
>
> **Nivel de riesgo**: R1 (modifica únicamente `SCRIPTS/harness_mode.json`, archivo de estado local y reversible). No toca `settings.yaml` ni credenciales.
>
> **Idioma**: TODA la interacción en español.

---

## 🗺️ 1. Alcance y Límites (IMPORTANTE)

| Qué SÍ cambia este skill | Qué NO cambia |
|---|---|
| La cadena de prioridades del bridge/rotador (`harness_mode.json`) | El `default_model` de la sesión DSH en curso (requiere editar `~/.dsh/settings.yaml` y reiniciar) |
| Efecto inmediato para delegaciones AGY / bridge / rotador NIM | Credenciales, pools ni listas de modelos |

Si el usuario pide cambiar qué modelo ejecuta **esta sesión** del agente, explicar que eso se hace editando `default_model` en `~/.dsh/settings.yaml` + reiniciar sesión, y ofrecer hacerlo aparte (R1, con backup).

---

## 🎯 2. Flujo Menu-First (OBLIGATORIO)

Al activarse el skill, el agente DEBE ejecutar estos pasos en orden:

### Paso 1 — Leer estado actual (verificación, no asunción)
```bash
cat ${WORKSPACE}/SCRIPTS/harness_mode.json
```
(extraer `mode` y `mode_name`; si no existe, asumir modo 2 = default)

### Paso 2 — Presentar el menú SIEMPRE (marcar el modo activo con 🟢)

```markdown
╔════════════════════════════════════════════════════════════════╗
║   🏎️  HARNESS FLOYDIA — SELECTOR DE MODO (f-harness-select)   ║
╚════════════════════════════════════════════════════════════════╝

Modo actual: [N] <nombre>

Responde con el número para cambiar de cadena de prioridades:

1️⃣  Full Power       → Kimi K3 → AGY → V4 Flash → V4 Pro → Pool
                        (máxima potencia, mejor calidad)
2️⃣  Working Horse    → Kimi K3 → V4 Flash → Pool Free
                        (equilibrio recomendado — DEFAULT)
3️⃣  Kimi Only        → solo Kimi K3 (rotación 4 cuentas NIM)
                        (máximo Kimi, sin AGY)
4️⃣  AGY + Free       → AGY → V4 Flash → Pool Free
                        (ideal si Kimi está caído o lento hoy)
5️⃣  Free Light       → Pool Free rotativo puro (NIM+OR+Zen)
                        (ultra austero, cuotas al límite)

0️⃣  Salir sin cambios

ℹ️  NOTA — La cadena es RESPALDO, no paralelismo:
    se usa el primer motor disponible; los demás entran SOLO si el
    anterior falla (429/caída). No significa "todos trabajando a la vez".
    Para elegir motor según DIFICULTAD de la tarea: /f-harness-router
```

### Paso 3 — Leer la selección del usuario
Usar la herramienta de pregunta interactiva (`ask_user_question`) con las opciones 1–5 (0 = salir). NO asumir; el usuario elige.

### Paso 4 — Aplicar la conmutación
Si elige 1–5, ejecutar:
```bash
bash ${WORKSPACE}/SCRIPTS/harness_mode.sh <N> < /dev/null
```

### Paso 5 — Verificación por evidencia (Protocolo v28)
Nunca afirmar el cambio sin confirmarlo:
```bash
cat ${WORKSPACE}/SCRIPTS/harness_mode.json
```
Presentar al usuario el resultado: modo nuevo, `mode_name` y `updated_at` literales del JSON.

---

## 📋 3. Tabla de referencia de los 5 modos

| Nº | Nombre | Cadena | Cuándo usarlo |
|----|--------|--------|----------------|
| 1 | `full_power` | Kimi K3 → AGY → V4 Flash → V4 Pro → Pool | Máxima potencia/calidad |
| 2 | `working_horse` | Kimi K3 → V4 Flash → Pool Free | Equilibrio diario (DEFAULT) |
| 3 | `kimi_only` | Solo Kimi K3 (rotación NIM 4 cuentas) | Máximo Kimi |
| 4 | `agy_free` | AGY → V4 Flash → Pool Free | Kimi caído/lento ese día |
| 5 | `free_light` | Pool Free rotativo | Cuotas al límite |

---

## 🛡️ 4. Reglas duras

- **Confirmación humana siempre** antes de ejecutar el cambio (el usuario elige número).
- Comando de conmutación con `< /dev/null` (anti-hang, sin prompts interactivos).
- Si `harness_mode.sh` devuelve error, mostrar el error literal y no reintentar más de 1 vez.
- Prohibido modificar `~/.dsh/settings.yaml` desde este skill.
- Si el usuario solo pidió "ver el modo", ejecutar Paso 1 y mostrar el estado sin ofrecer cambio.
- Registrar cambios relevantes como evento append-only en `.session/memory_events.jsonl` solo si el usuario lo pide.

---

## ✅ 5. Criterio de éxito del skill

- Menú presentado con el modo real marcado.
- Conmutación con exit 0 y JSON persistido verificado y mostrado literalmente.
- Cero cambios no solicitados.

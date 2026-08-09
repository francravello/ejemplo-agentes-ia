# Checkpoint 1 — Agente Base y Motor de Razonamiento

**Alumno:** Francisco Cravello
**Proyecto integrador:** Agente de recupero de deuda — ConsultoraIndependiente
**Entregable:** `checkpoint1_francisco_cravello.json`

## Descripción

Agente conversacional de cobranzas construido en n8n. Atiende consultas sobre deuda vencida,
consulta el padrón de deudores en Airtable y guía la conversación hacia uno de cinco cierres
operativos: plan aceptado, pago informado, prórroga agendada, baja de contacto o escalamiento
a supervisor humano.

## Arquitectura

```
[Chat Trigger]
      │
      ▼
[AI Agent — Tools Agent]  ◄── ai_languageModel ── [OpenAI Chat Model · GPT-4o]
      │                   ◄── ai_memory ───────── [Simple Memory]
      │                   ◄── ai_tool ─────────── [Airtable · consultar_datos_cliente]
      ▼
[Set — Armar Log de Auditoría]
      │
      ▼
[Slack — Log "TAREA COMPLETADA"]
      │
      ▼
[Set — Respuesta al Chat]
```

## Componentes

| Requisito | Implementación |
|---|---|
| Disparador | `Chat Trigger` |
| Cerebro | `AI Agent` en modo Tools Agent |
| Modelo | `OpenAI Chat Model` — GPT-4o, temperature 0.3 |
| Guardrail | `Max Iterations: 8` (rango permitido 5–10) |
| System Prompt | Modular: Rol → Ámbito → Objetivo → Reglas y Límites → Escalamiento |
| Herramienta | `Airtable` acoplado lateralmente por `ai_tool`, con descripción semántica |
| Observabilidad | `Set` + `Slack` — reporta "TAREA COMPLETADA" con la traza de razonamiento |

## Estructura de la tabla en Airtable

Base `Deudores`, tabla `Deudores`:

| Campo | Tipo | Descripción |
|---|---|---|
| `Nombre` | Texto | Nombre de pila del deudor |
| `Apellido` | Texto | Apellido |
| `monto_deuda` | Número | Importe adeudado en ARS |
| `dias_mora` | Número | Días de atraso |
| `estado_cliente` | Texto | Vacío si está impago, `pago` si ya abonó |
| `monto_pagado` | Número | Importe abonado, si corresponde |

## Cómo reproducirlo

1. Importar `checkpoint1_francisco_cravello.json` en n8n (`...` → Import from File).
2. Crear las credenciales propias y asignarlas a cada nodo:
   - OpenAI (API key)
   - Airtable (Personal Access Token con scopes `data.records:read` y `data.records:write`)
   - Slack (OAuth2 o Bot Token con scope `chat:write`)
3. En el nodo de Airtable, seleccionar la base y la tabla propias.
4. En el nodo de Slack, indicar el canal de destino.
5. Ejecutar `Test workflow` y enviar un mensaje de prueba desde el panel de chat.

> El JSON no contiene claves de API. n8n almacena las credenciales cifradas en su propia
> base de datos y en el export solo persisten identificadores internos sin valor fuera de
> la instancia de origen.

## Evolución prevista

Este flujo es la base del proyecto integrador y se extiende módulo a módulo:
M2 multi-agente · M3 memoria y contexto · M4 integraciones · M5 RAG · M6 voz · M11 proyecto final.

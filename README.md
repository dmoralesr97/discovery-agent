# Discovery Agent

Agente de Claude Code que convierte entrevistas de negocio en artefactos de
producto: personas, requisitos, user stories, MVP Canvas e hipótesis con
experimentos para validarlos.

## Prerrequisitos

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) instalado y
  autenticado.
- Python 3 disponible en el PATH (para los hooks y el generador de reportes).

## Estructura del proyecto

```
discoveries/
  <nombre>/
    interviews/    ← entrevistas en Markdown (la única fuente de verdad)
    outputs/       ← artefactos generados (no editar a mano)
  _template/       ← discovery en blanco para copiar al empezar
.claude/
  skills/          ← skill discovery (estándar de formato)
  hooks/           ← gates automáticos (readiness, hypothesis)
  scripts/         ← generador del reporte HTML
```

## Flujo de trabajo

Cada discovery sigue cuatro pasos en orden. Los comandos se ejecutan desde la
raíz del proyecto dentro de Claude Code.

### 1. Preparar las entrevistas

Copia `discoveries/_template/` con el nombre de tu caso:

```
cp -r discoveries/_template discoveries/mi-caso
```

Agrega las entrevistas en `discoveries/mi-caso/interviews/`. Cada archivo `.md`
debe tener este frontmatter:

```markdown
---
fuente: entrevista
rol_entrevistado: <rol en minúsculas, p. ej. "coordinador de marketing">
primera_persona: true
anonimizada: true
fecha: AAAA-MM-DD
---

[Texto de la entrevista en primera persona, en español]
```

> Una persona del MVP necesita al menos una entrevista con `primera_persona: true`
> de ese rol. Las personas construidas solo de referencias de terceros no pueden
> sustentar el MVP (el gate lo bloquea).

### 2. Analizar la evidencia

```
/discovery:analyze discoveries/mi-caso
```

Produce en `outputs/`:
- `personas.md` — personas y stakeholders con sus dolores y trazabilidad.
- `requisitos.md` — requisitos candidatos numerados (R-01, R-02…).
- `evidence-map.json` — mapa de auditoría que leen los gates.

### 3. Generar el MVP

```
/discovery:generate-mvp discoveries/mi-caso
```

**Gate de readiness** (automático): antes de escribir verifica que haya al menos
2 entrevistas, que cada persona primaria tenga respaldo de primera mano en disco
y que ningún dolor esté huérfano. Si bloquea, agrega la evidencia que falta y
reintenta.

Produce en `outputs/`:
- `user-stories.md` — historias INVEST con criterios de aceptación y fuente.
- `mvp-canvas.md` — MVP Canvas con propuesta de valor, métricas de éxito y
  bloque explícito de fuera de alcance.

### 4. Diseñar experimentos

```
/discovery:experiments discoveries/mi-caso
```

**Gate de hipótesis** (automático): exige que cada hipótesis tenga señal medible,
umbral con número, regla de decisión que contemple el fallo y métrica que no sea
de vanidad. Si bloquea, reescribe la hipótesis en forma falsable.

Produce en `outputs/`:
- `experiment-board.json` — tablero machine-readable (se escribe primero).
- `hypotheses.md` — test cards legibles ordenadas de mayor a menor riesgo.

### 5. Generar el reporte visual

```
/discovery:report discoveries/mi-caso
```

Produce `outputs/report.html`: dashboard autocontenido con personas (verde =
primera mano, ámbar = referenciada), cadena output → outcome → impact y tablero
de experimentos por riesgo (rojo / ámbar / verde). Se abre en cualquier navegador.

## Reglas no negociables

| Regla | Qué significa |
|---|---|
| Cero invención | Ningún dolor, persona o requisito sin respaldo en una entrevista real. |
| Trazabilidad | Todo cita el archivo fuente por nombre (`recepcionista.md`). |
| Primera mano | Una persona de MVP necesita su propia entrevista, no solo menciones de otros. |
| Aislamiento | No mezcles evidencia ni artefactos entre distintos discoveries. |
| Idioma | Español, salvo términos técnicos (user story, MVP, stakeholder). |

## Ejemplo de referencia

El discovery `discoveries/radiostats/` está completo con los cuatro pasos
ejecutados. Sirve de referencia para ver el formato esperado de cada artefacto.

## Solución de problemas frecuentes

**El gate de readiness bloquea con "entrevista en primera persona no encontrada"**
— Verifica que el `rol_entrevistado` del frontmatter de la entrevista coincida
exactamente (en minúsculas) con el campo `role` del `evidence-map.json`.

**El gate de hipótesis bloquea con "criterio de éxito no concreto"**
— El campo `threshold` del JSON debe contener un número o porcentaje
(p. ej. `"≥ 30% en 4 semanas"`, no `"la mayoría"`).

**El gate de hipótesis bloquea con "regla de decisión no dice qué hacer si falla"**
— La `decision` debe incluir explícitamente qué ocurre si el experimento falla
(pivotar, descartar, investigar). No basta con describir el caso de éxito.

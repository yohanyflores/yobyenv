# AGENTS.md

Instrucciones para cualquier agente (Claude Code, Codex, OpenCode, agy) que
trabaje en este repositorio. Aplican a todo el repo. **Este archivo es la
fuente de verdad**: `CLAUDE.md` y cualquier otro archivo de instrucciones solo
lo importan o lo referencian, nunca lo duplican.

Este archivo describe **cómo se trabaja**, no qué es el producto. La visión,
los objetivos, los no-objetivos y la arquitectura de yoby salen del flujo de
diseño de abajo y viven en `docs/designs/` y `docs/adr/`.

## Contexto

yoby es un proyecto nuevo, con código nuevo. Hereda la idea de un intento
anterior (`yobydev`), pero **no hereda su código, sus reglas ni sus
decisiones**. No asumas que algo de `yobydev` aplica aquí: si parece útil, se
propone y se decide de nuevo con el proceso de abajo.

## Principios

- **Terminal primero, dentro del contenedor.** El trabajo de desarrollo ocurre
  en la terminal del contenedor. Una interfaz web solo entra cuando un
  experimento documentado demuestra que la terminal no resuelve el problema, y
  esa conclusión queda en un ADR.
- **Probar antes de decidir.** Un concepto que se ve bien en papel no se acepta
  sin evidencia práctica. De cada prueba se guarda la conclusión, no la
  bitácora.
- **Alcance cerrado.** Nada se implementa si no está en un diseño aprobado o en
  la tarea pedida.
- **Herramientas existentes primero.** Antes de programar una solución, se
  verifica si una herramienta existente ya la resuelve con configuración.

Si una tarea empuja fuera de un diseño aprobado, **detente y pregunta**. No lo
implementes "porque era fácil".

## Cómo se trabaja aquí

| Tipo de cambio | Qué requiere |
| --- | --- |
| Decisión de arquitectura, lenguaje, dependencia nueva o cambio de alcance | ADR en `docs/adr/` |
| Componente o funcionalidad nueva no trivial | Diseño aprobado en `docs/designs/` (flujo de abajo) |
| Bug, refactor local, ajuste dentro de un diseño aprobado | Nada extra: código + tests |

### Flujo para funcionalidades nuevas

El diseño se lleva con el plugin EGM (Elephant-Goldfish Model). En Claude
Code los comandos llevan el prefijo `/egm:`.

1. **Brief** — `/egm:egm-brainstorm` deja la dirección acordada.
2. **Supuestos riesgosos** — del brief se listan los supuestos que, si fallan,
   matan la idea.
3. **Experimentos** — cada supuesto se prueba con un experimento corto, con
   tiempo límite, en una rama `exp/<tema>` que **nunca se fusiona**. El código
   del experimento se descarta.
4. **Borrador** — `/egm:egm-draft` a partir del brief más los resultados. El
   diseño incluye una sección **Evidencia** (formato abajo).
5. **Revisión** — `/egm:egm-review` con revisores de contexto limpio. Toda
   afirmación técnica sin respaldo en Evidencia es un hallazgo.
6. **Resolución** — `/egm:egm-resolve`. Los hallazgos se resuelven en el
   documento y desaparecen.
7. **Aprobación** — Yohany aprueba; solo entonces se implementa.

Sección **Evidencia**, una entrada por supuesto:

```markdown
### S1: <supuesto>
- Hipótesis: …
- Prueba: qué se hizo, en qué entorno (local y, si aplica, por SSH).
- Resultado: funcionó / no funcionó / funcionó con límites (cuáles).
- Versiones: herramientas y versiones exactas usadas.
```

### Reglas comunes

- La fuente de verdad es el código y los tests. Los ADR explican **por qué**;
  los diseños explican **cómo**. Si el código contradice un documento, se
  corrige uno de los dos en el mismo cambio.
- Solo se versionan ADR y diseños aprobados. Briefs, borradores, salidas de
  revisión y bitácoras de experimentos no se commitean.
- No se archivan rondas de revisión ni respuestas. La historia vive en git.
- `/egm:egm-drift-report` se corre antes de cerrar cada fase.

### ADR

- Ubicación: `docs/adr/NNNN-titulo-en-kebab-case.md`, numeración continua.
  Plantilla: `docs/adr/0000-plantilla.md`.
- Máximo una página. Si no cabe, es un diseño, no un ADR.
- Si la decisión depende de comportamiento técnico, no se acepta sin una
  entrada en **Evidencia**.
- Un ADR aceptado no se edita: se reemplaza con uno nuevo que lo referencie.

## Idioma

- Documentación, ADR, diseños y commits: **español**.
- Código, identificadores, nombres de archivos de código y mensajes de log:
  **inglés**, siguiendo las convenciones del lenguaje elegido.

## Pendiente de definir

Estas secciones se escriben cuando existan los diseños y ADR que las sustentan:

- Mapa del repositorio.
- Lenguaje y sus convenciones.
- Reglas de arquitectura.
- Comandos de build, test y validación antes de entregar.

Mientras no existan, si algo no se pudo validar, dilo con precisión. No lo des
por bueno.

## Commits y ramas

- Conventional Commits, en español: `feat(ámbito): descripción`.
- Un commit, una intención. No mezclar refactor con funcionalidad.
- Todo cambio entra a `main` por pull request. Las ramas `exp/*` nunca se
  fusionan.

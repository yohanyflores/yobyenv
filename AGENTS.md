# AGENTS.md

Instrucciones para cualquier agente (Claude Code, Codex, OpenCode, agy) que
trabaje en este repositorio. Aplican a todo el repo. **Este archivo es la
fuente de verdad**: `CLAUDE.md` y cualquier otro archivo de instrucciones solo
lo importan o lo referencian, nunca lo duplican.

> Estado: v0. Las secciones marcadas **[pendiente]** se completan con los ADR
> correspondientes.

## Qué es yoby

**[pendiente: ADR-0001]** Entorno de desarrollo opinado basado en Dev
Containers. Sucesor de `yobydev`, rediseñado desde cero.

## Principios

- **Terminal primero, dentro del contenedor.** El trabajo de desarrollo (shell,
  multiplexor, editor, agentes de IA) ocurre en la terminal del devcontainer.
  Una interfaz web solo entra cuando un experimento documentado demuestra que
  la terminal no resuelve el problema, y esa conclusión queda en un ADR.
- **Probar antes de decidir.** Un concepto que se ve bien en papel no se acepta
  sin evidencia práctica. De cada prueba se guarda la conclusión, no la
  bitácora.
- **Alcance cerrado.** Nada se implementa si no está en el ROADMAP y en un
  diseño aprobado o una tarea pedida.

## No-objetivos

**[pendiente: confirmar en ADR-0001]** Heredados del intento anterior:

- Sin daemon global.
- Sin SSH directo al host: solo SSH + devcontainer.

Si una tarea empuja hacia algo de esta lista o fuera del ROADMAP, **detente y
pregunta**. No lo implementes "porque era fácil".

## Cómo se trabaja aquí

| Tipo de cambio | Qué requiere |
| --- | --- |
| Decisión de arquitectura, lenguaje, dependencia nueva, cambio de alcance o de un no-objetivo | ADR en `docs/adr/` |
| Componente o funcionalidad nueva no trivial | Diseño aprobado en `docs/designs/` (flujo de abajo) |
| Bug, refactor local, ajuste dentro de un diseño aprobado | Nada extra: código + tests |

### Flujo para funcionalidades nuevas

1. **Brief** — `/egm:brainstorm` deja la dirección acordada.
2. **Supuestos riesgosos** — del brief se listan los supuestos que, si fallan,
   matan la idea (p. ej. "un agente dentro del contenedor puede recibir una
   imagen pegada desde Kitty").
3. **Experimentos** — cada supuesto se prueba con un experimento corto, con
   tiempo límite, en una rama `exp/<tema>` que **nunca se fusiona**. El código
   del experimento se descarta.
4. **Borrador** — `/egm:draft` a partir del brief más los resultados. El diseño
   incluye una sección **Evidencia** (formato abajo).
5. **Revisión** — `/egm:review` con revisores de contexto limpio. Toda
   afirmación técnica sin respaldo en Evidencia es un hallazgo.
6. **Resolución** — `/egm:resolve`. Los hallazgos se resuelven en el documento
   y desaparecen.
7. **Aprobación** — Yohany aprueba; solo entonces se implementa.

Sección **Evidencia**, una entrada por supuesto:

```markdown
### S1: <supuesto>
- Hipótesis: …
- Prueba: qué se hizo, en qué entorno.
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
- `/egm:drift-report` se corre antes de cerrar cada fase.

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

## Mapa del repositorio

**[pendiente]** Se escribe junto con el primer diseño.

## Reglas de arquitectura

- El punto de entrada (`cmd/` o equivalente) solo cablea piezas: parsea
  argumentos, construye dependencias y delega. Si un archivo ahí pasa de ~200
  líneas, hay lógica mal ubicada.
- Toda extensión ocurre por registro explícito. Prohibido depender del orden
  de inicialización o de nombres de archivo para envolver comportamiento.
- Compatibilidad con la CLI oficial `devcontainer` y con contenedores creados
  por VS Code. Nunca editar el `devcontainer.json` del usuario: los cambios en
  tiempo de ejecución van en overrides privados generados.
- Canonicalizar rutas (resolver symlinks) antes de comparar, persistir o lanzar.
- Todo listener escucha en `127.0.0.1`. Nunca exponer un servicio a la LAN sin
  una opción explícita aprobada en un ADR.
- Efectos laterales (reconstruir, detener o borrar contenedores, borrar
  archivos, abrir programas) solo por acción explícita del usuario, nunca
  durante descubrimiento o lectura.
- Antes de borrar algo, demostrar que está dentro de un directorio gestionado
  por yoby.
- Estado persistente compartido entre procesos: locking y reemplazo atómico.
- Entradas remotas y nombres de archivo recibidos son datos no confiables.
- Infraestructura (Docker, terminal, red) detrás de interfaces para poder
  probar sin ella.
- Toda operación externa y todo proceso concurrente tiene un camino explícito
  de cancelación.
- Dependencia nueva = ADR.

## Convenciones del lenguaje

**[pendiente: ADR-0002, lenguaje de implementación]**

## Antes de entregar un cambio

**[pendiente: comandos exactos cuando exista la herramienta de build]**

1. Formatear cada archivo modificado.
2. Tests enfocados del módulo tocado.
3. Suite completa.
4. Build si cambió el cableado de comandos o algún binario.

Si algo no se pudo validar (sin Docker, sin red, sin terminal), dilo con
precisión. No lo des por bueno.

## Commits

- Conventional Commits, en español: `feat(ámbito): descripción`.
- Un commit, una intención. No mezclar refactor con funcionalidad.

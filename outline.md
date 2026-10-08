# Outline — Curso avanzado: Arquitectura Cursor sobre monorepos reales

**Caso de estudio:** [novarz/grafana](https://github.com/novarz/grafana) (fork demo Cursor 101/201 de Grafana).  
**Audiencia:** equipos de ingeniería / platform / staff+ (español).  
**Nivel:** avanzado — diseño de sistemas agentic, no tips de prompt.  
**Companion:** `example-map.md` (rutas concretas por tema).

Duración orientativa: 2–3 días intensivos o 6–8 sesiones de 90–120 min.  
Cada capítulo incluye: objetivos de aprendizaje, lecture deep-dive, laboratorio sobre el repo, criterios de evaluación, anti-patrones.

---

## Prerrequisitos

- Experiencia con monorepos (FE+BE), CI/CD, y code review.
- Uso básico de Cursor (Agent / Chat / Rules) — no se enseña “cómo abrir el IDE”.
- Lectura previa obligatoria: `AGENTS.md` completo + `.cursor/101-SCRIPT.md` (skim).

---

## Capítulo 0 — Framing: de “asistente de código” a sistema agentic

**Objetivos**
- Distinguir tool-use ad-hoc vs arquitectura de agentes con fronteras, estado y pipelines.
- Mapear Grafana como sistema de estudio: monorepo + demo Cursor + proceso OSS.

**Deep-dive**
- Sistema agentic = frontera + estado durable + pipeline. La norma vive en el repo, no en el chat.
- Cuatro capas (código, contexto, proceso humano, CI/entorno). El harness tiene punto de restauración; el ensayo no es el producto.

**Lab**
- Cinco artefactos de cualquier sistema, clasificados por capa. Como mucho un path de Grafana. Texto: `capitulos/00-01-caso-grafana.md`.

**Evaluación**
- Diagrama de una página: humano / agente / CI / estado durable. Aprueba si alguien que no conoce el producto dice dónde se detiene el agente.

**Anti-patrones**
- Archivar el fichero de instrucciones tras leerlo una vez. Ignorar dueños y cadencias.

---

## Capítulo 1 — System boundaries: módulos del monorepo como contratos

**Objetivos**
- Definir una frontera agent-safe: qué puede tocar un agente en un turno sin romper el sistema.
- Estimar blast radius, partir por cadencia y reconocer cuándo hay que regenerar en vez de editar la salida.

**Deep-dive**
- Frontera = contrato (toolchain, dueños, validación propia). Blast radius = quién se rompe, no cuántas líneas.
- Un PR por cadencia. Se edita la fuente y se regenera. El tamaño de un turno es el número de fronteras que cruza.

**Lab**
- «Cambiar el título de un panel»: plan de solo presentación y plan de cambio de modelo. Texto: `capitulos/00-01-caso-grafana.md`.

**Evaluación**
- Matriz: frontera × blast radius × tests × dueños × PRs × comando de generación.

**Anti-patrones**
- Un solo PR para dos cadencias. Editar a mano un fichero generado.

---

## Capítulo 2 — Context architecture: rules, AGENTS.md, docs vs code

**Objetivos**
- Diseñar capas de contexto: root → directory-scoped → skill → style guide.
- Evitar duplicación y drift entre docs humanas y agent instructions.

**Deep-dive**
- Pila: raíz → zona con glob → skill → guía humana por enlace.
- La zona especializa y no revoca la raíz. Una regla, un sitio. Cada fichero nombra un comando real y dos modos de fallo.

**Lab**
- AGENTS de zona ficticio (glob, enlaces, skill, tres reglas locales, dos fallos) y tres choques raíz/zona para revisar. Destino Grafana opcional: `pkg/tsdb/`. Texto: `capitulos/02-contexto-agentes.md`.

**Evaluación**
- Rúbrica: accionable, acotado, referencial, modos de fallo, compatible con la raíz.

**Anti-patrones**
- Pegar la guía de estilo. Dos órdenes opuestas. Comandos que el repo no tiene.

---

## Capítulo 3 — Agent topology: skills, scripts, humanos y especialización

**Objetivos**
- Modelar el turno como reparto: agente generalista, skill, humano y harness con restore point.
- Decidir cuándo abrir una skill, cuándo basta la raíz o la zona, y cuándo parar para una persona.

**Deep-dive**
- Cuatro papeles. Regla permanente frente a skill con disparador frente a puerta humana.
- Una skill llama a otra (panel → frontend-testing → selectores); no copia el procedimiento.
- Quien presenta arranca el producto. El agente no. El restore point es un pin con nombre (`demo/start`), y el script solo mueve el git.
- Push, force-push, merge y un webhook no van en el paso por defecto de una skill.

**Lab**
- Simular el flujo: pin anotado, agente en rama de ensayo sin push, la persona arranca la app, `reset-demo` vuelve al pin.
- Especificar una skill nueva «go-migration-check» (solo `SKILL.md`, sin merge). Texto: `capitulos/03-topologia-agentes.md`.

**Evaluación**
- Diagrama de secuencia: usuario → agente → skill → shell → CI → humano, con la parada antes del push. Rúbrica de la skill.

**Anti-patrones**
- Agente monolítico sin skills. Skill que publica o force-pushea sin parada. Restaurar a la punta de `main` en vez del pin `demo/start`.

---

## Capítulo 4 — Tool & connector surface

**Objetivos**
- Inventariar herramientas legítimas del repo (Make, Yarn, Go, Playwright, lefthook, GitHub bots).
- Diseñar “sanctioned paths” para agentes: preferir targets documentados vs inventar pipelines.

**Deep-dive**
- Makefile como API operativa; `lefthook.yml` como enforcement local.
- OpenAPI / swagger artifacts en `public/` como conectores de contrato.
- `.github/bot.md`, `commands.json`, `pr-commands.json`: automatización fuera del IDE.
- Selectores E2E (`packages/grafana-e2e-selectors`) como interfaz estable UI↔tooling.

**Lab**
- Dado un ticket “añadir endpoint”, listar tools en orden: gen → test → lint → e2e selector → PR template.
- Comparar “agente usa `gh`” (mencionado en alerting AGENTS) vs “agente usa Makefile”.

**Evaluación**
- Policy de 1 página: tools permitidos / prohibidos / requieren aprobación (alineada a Human Review Gates).

**Anti-patrones**
- Bypass de lefthook/`--no-verify`; scrapear secretos para hablar con APIs internas; regenerar OpenAPI a mano sin `make swagger-gen`.

---

## Capítulo 5 — Change pipelines: PR, CI, ownership

**Objetivos**
- Encajar el agente en el pipeline real: plantilla PR → checks → CODEOWNERS → merge policy.
- Entender sharding, compatibility workflows y breaking-change detection.

**Deep-dive**
- `contribute/create-pull-request.md` / `merge-pull-request.md` / `WORKFLOW.md`.
- Muestra representativa de workflows: `pr-checks`, unit FE/BE, `pr-e2e-tests`, `pr-unified-storage-compatibility`, `detect-breaking-changes-levitate`, `proto-breaking-changes`.
- CODEOWNERS como routing de review (y de contexto agentic implícito).
- Backports, security branches, release workflows: el agente no es el release engineer.

**Lab**
- Para un PR hipotético que toca `pkg/storage/unified` + caller en `pkg/api`: predecir qué workflows fallan y por qué (compat).
- Redactar el cuerpo del PR usando la plantilla oficial, con sección “agent-assisted / human-verified”.

**Evaluación**
- Checklist pre-push del agente alineada a CI (no solo “tests pasan en local”).

**Anti-patrones**
- Abrir PR sin gates humanos; ignorar CODEOWNERS; “arreglar” CI con skips; mezclar security patch en PR público.

---

## Capítulo 6 — Durable state: config, migraciones, schemas, flags

**Objetivos**
- Identificar estado que sobrevive al chat: DB, config, CUE kinds, feature toggles, provisioning.
- Diseñar cambios agentic que respeten migraciones y codegen.

**Deep-dive**
- `conf/defaults.ini` / `sample.ini` / `provisioning/`.
- SQL migrations (`pkg/services/sqlstore/migrations/`) y unified storage migrations.
- `kinds/` → `make gen-cue` → `packages/grafana-schema`.
- Feature toggles (`contribute/feature-toggles.md`, `pkg/services/featuremgmt/`, OpenFeature en alerting AGENTS).
- Wire DI: `pkg/server/wire.go` + `make gen-go`.

**Lab**
- Plan de cambio “nuevo campo en dashboard schema”: orden de archivos y comandos de generación.
- Plan de migración SQL + test con `make devenv sources=postgres_tests`.

**Evaluación**
- “State impact statement” obligatorio en cada propuesta de agente.

**Anti-patrones**
- Editar `wire_gen.go` / types generados; feature toggle leído vía `config.featureToggles` donde está prohibido; migración no reversible sin plan.

---

## Capítulo 7 — Demo y multi-env: harnesses reproducibles

**Objetivos**
- Construir y operar entornos multi-config para demos y agentes (local SQLite, devenv docker, Cursor Cloud).
- Separar “estado de demo” de “estado de producto”.

**Deep-dive**
- Reset harness: `.cursor/reset-demo.sh` / skill `reset-demo`.
- Restricciones del demo en `AGENTS.md` (Jira SDFD, ramas, Node nvm 24.11.0, quién arranca Grafana).
- `devenv/`: compose dinámico, datasources locales vs docker, secrets fixtures.
- Toolchain pins: `.nvmrc`, `mise.toml`, gotchas PATH en Cursor Cloud documentados en AGENTS.

**Lab**
- Diseñar un “demo day runbook” de 10 pasos con restore point y ownership humano/agente.
- Comparar tres envs: SQLite embebido, `make devenv sources=influxdb`, Docker image.

**Evaluación**
- Runbook que un segundo instructor puede ejecutar sin el primero (reproducibilidad).

**Anti-patrones**
- Demo en `main` sin branch; agente dispara webhook 101; drift de Node entre shell del agente y shell de login.

---

## Capítulo 8 — Evaluation & failure modes

**Objetivos**
- Evaluar trabajo de agentes más allá de “compila”: tests de comportamiento, smoke, CUJ, security scanners.
- Catalogar failure modes típicos (slop tests, flakes, secret leaks, breaking APIs).

**Deep-dive**
- Skills de testing como *eval harness instructions*.
- Pirámide: Jest unit → Go unit/integration → Playwright suites (`smoke-tests-suite`, `dashboard-cujs`) → journey smokes.
- CI evaluators: coverage, perf, govulncheck, trufflehog, reject-gh-secrets.
- Failure modes de contexto: agente ignora globs / lee AGENTS equivocado.

**Lab**
- Revisar un test “AI-slop” ficticio contra `frontend-testing-strategy` y reescribir criterios.
- Definir scorecard de PR agent-assisted: correctness, blast radius, test quality, security, docs.

**Evaluación**
- Scorecard aplicado a 2 PRs de ejemplo (uno bueno, uno malo).

**Anti-patrones**
- Medir solo LOC generadas; snapshots frágiles como única red; silenciar scanners.

---

## Capítulo 9 — Team operating model

**Objetivos**
- Integrar agentes en el modelo de squads: ownership, triage, governance, notificaciones.
- Definir roles: author agent, reviewer humano, CODEOWNER, release, security.

**Deep-dive**
- `CONTRIBUTING.md`, `GOVERNANCE.md`, `MAINTAINERS.md`.
- CODEOWNERS + `team-notifications-config.json`.
- Issue triage (`contribute/ISSUE_TRIAGE.md`) y bots de labelling.
- Cadencias FE/BE y “separate PRs” como política de equipo, no preferencia estética.
- Cómo un squad (alerting) externaliza normas en AGENTS scoped + skills.

**Lab**
- Escribir un RFC corto: “Agent operating model for Squad X” (cuando el agente puede mergear drafts, quién aprueba, qué skills son canónicas).
- Mapear 5 paths a owners y notificaciones.

**Evaluación**
- RFC revisado por pares con criterios de accountability.

**Anti-patrones**
- Agente como “owner” informal; notificaciones a todos; AGENTS.md sin maintainer humano.

---

## Capítulo 10 — Security & blast radius

**Objetivos**
- Acotar el radio de daño de agentes: secrets, auth, supply chain, policy-as-code, security branches.
- Traducir “dependency security” de alerting AGENTS a política general.

**Deep-dive**
- `.policy.yml` como control plane.
- Workflows de security branch / patch mirror; SBOM; relyance.
- Superficies calientes: `apps/secret`, `apps/iam`, `pkg/login`, `pkg/middleware`, `devenv/secrets`.
- Secret scanning en CI (`trufflehog`, `reject-gh-secrets`).
- Reglas agentic: no nuevas deps sin aprobación; yarn immutable; cuarentena 7 días.

**Lab**
- Threat model de un agente con acceso a `Makefile` + `gh` + editores: STRIDE ligero sobre el monorepo.
- Propuesta de “blast radius labels” en PRs (low/med/high) con ejemplos de paths.

**Evaluación**
- Threat model + política de aprobación por clase de cambio.

**Anti-patrones**
- Agente con credenciales de producción en demo; `yarn add` silencioso; security fix en issue público.

---

## Capstone — Diseñar la arquitectura Cursor de *vuestro* monorepo

**Entregable (equipo)**
1. Mapa de fronteras (análogo a Cap. 1).
2. Árbol de contexto: root AGENTS + 2 scoped + 2 skills.
3. Policy de tools + human gates.
4. Scorecard de evaluación + 3 failure modes mitigados.
5. Plan de demo/reset reproducible (aunque el producto no sea Grafana).

**Defensa**
- 20 min: walkthrough con paths reales del repo propio o del caso Grafana.
- Pregunta obligatoria del tribunal: “¿Qué no debe hacer nunca el agente, y dónde está escrito?”

---

## Apéndices

- **A.** Glosario ES/EN (AGENTS, skill, CODEOWNERS, Wire, CUE kinds, CUJ, SDFD demo).
- **B.** Rutas canónicas — ver `example-map.md`.
- **C.** Lecturas: `contribute/architecture/*`, `pkg/storage/unified/README.md` (selección), alerting `TESTING.md`.
- **D.** Checklist del instructor para el harness `novarz/grafana` (reset, ramas, Jira SDFD, Node pin).

---

## Principios pedagógicos del curso

1. **Principio, luego un ancla:** cada idea se entiende sin Grafana; como mucho un path por sección. El inventario de rutas vive en `example-map.md`.
2. **Sistemas > prompts:** se diseña contexto, topología, pipelines y evaluación.
3. **Human gates son feature, no bug:** la aprobación explícita es parte de la arquitectura.
4. **Demo ≠ prod:** el fork Cursor enseña harnesses; las lecciones de security/CI aplican al Grafana real.
5. **Profundidad:** se prioriza razonamiento de blast radius y failure modes sobre velocidad de codegen.

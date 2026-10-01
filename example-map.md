# Mapa de ejemplos — novarz/grafana (Cursor demo)

Fuente: [novarz/grafana](https://github.com/novarz/grafana) (`main`, demo Cursor 101/201).  
Uso: anclar cada tema del curso avanzado de arquitectura Cursor a rutas reales del monorepo.  
No se modificó el repositorio; solo lectura vía cursor-github.

## Top-level (referencia rápida)

Módulos y superficie agentic visibles en la raíz:

| Path | Rol |
|------|-----|
| `AGENTS.md` | Contrato global para agentes (demo + arquitectura + comandos) |
| `.cursor/` | Script 101, reset demo, skills |
| `pkg/` | Backend Go |
| `public/` | Frontend TS/React + assets |
| `packages/` | Librerías Yarn (`@grafana/*`) |
| `apps/` | Apps Go (App SDK) |
| `.github/workflows/` | ~99 workflows CI/CD |
| `devenv/` | Multi-env / datasources / docker |
| `docs/` | Docs de producto + `docs/AGENTS.md` |
| `contribute/` | Modelo operativo humano (PR, estilo, arquitectura) |
| `e2e-playwright/` | Suites E2E / evaluación |
| `conf/` | Config durable (defaults, sample, provisioning) |
| `kinds/` | Schemas CUE generativos |
| `go.work` / `package.json` / `nx.json` | Workspaces Go + Yarn + Nx |

---

## 1. System boundaries (módulos del monorepo)

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `pkg/` | Frontera del backend Go: API, services, plugins, storage, server. |
| `public/app/` | Frontera del frontend: `core/`, `features/`, `plugins/`, `store/`. |
| `packages/` | Paquetes compartidos (`grafana-data`, `grafana-ui`, `grafana-runtime`, `grafana-schema`, …) que acoplan FE y contratos. |
| `apps/` | Apps Go autónomas (dashboard, folder, alerting, iam, secret, …) vía App SDK — límite distinto a `pkg/services/`. |
| `go.work` + `package.json` | Workspaces duales (Go + Yarn) que definen qué “módulo” es buildable/testeable por separado. |
| `contribute/architecture/k8s-inspired-backend-arch.md` | Documenta el límite conceptual k8s-inspired entre API layer y storage. |

---

## 2. Context architecture (rules, AGENTS.md, docs vs code)

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `AGENTS.md` | Contexto raíz: principios, comandos, mapa de directorios, gates humanos, reglas del demo SDFD. |
| `docs/AGENTS.md` | Contexto scoped a documentación (estilo distinto al código). |
| `public/app/features/alerting/unified/AGENTS.md` | AGENTS de squad con `globs`, links a style guides y skills — patrón de contexto jerárquico. |
| `pkg/storage/unified/AGENTS.md` | Reglas de compatibilidad backwards entre capas con cadencias distintas. |
| `public/app/core/journeys/AGENTS.md` | Contexto de instrumentación CUJ separado del feature code. |
| `contribute/style-guides/` + `contribute/developer-guide.md` | Capa “docs humanos” que los AGENTS.md referencian en lugar de duplicar. |

---

## 3. Agent topology

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `.cursor/101-SCRIPT.md` | Talk track del presenter: orquesta humanos + agentes en 101/201. |
| `.cursor/skills/reset-demo/SKILL.md` | Skill invocable: topología “skill → script → git pin a `demo/start`”. |
| `.cursor/skills/frontend-testing-strategy/SKILL.md` | Skill de dominio (testing FE) referenciada desde AGENTS globales y de alerting. |
| `.cursor/skills/panel-testing-strategy/` | Skill especializada apilada sobre frontend-testing (topología en capas). |
| `.cursor/skills/add-e2e-selectors/` | Skill de tooling E2E: agente como especialista estrecho. |
| `AGENTS.md` (sección “Directory-scoped agent files”) | Declara explícitamente la topología: root agent + agentes scoped por directorio. |

---

## 4. Tool & connector surface

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `Makefile` | Superficie de comandos canónicos (`run`, `gen-go`, `devenv`, tests) que el agente debe preferir. |
| `lefthook.yml` | Hooks locales: tooling que el agente no debe saltarse (lint/format pre-commit). |
| `playwright.config.ts` + `e2e-playwright/` | Conector de evaluación E2E invocable vía `yarn e2e:playwright`. |
| `packages/grafana-e2e-selectors/` | Contrato de selectores entre UI y tests/herramientas. |
| `public/openapi3.json` / `public/api-merged.json` | Superficie API generada consumible por clientes/herramientas. |
| `.github/bot.md` + `.github/commands.json` | Bots/comandos GitHub: conectores automatizados del proceso (no solo CLI local). |

---

## 5. Change pipelines (PR, CI)

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `.github/PULL_REQUEST_TEMPLATE.md` | Checklist humano en el borde del pipeline. |
| `contribute/create-pull-request.md` + `contribute/merge-pull-request.md` | Política de cambio: cómo entra y sale código. |
| `.github/workflows/pr-checks.yml` + `pr-frontend-unit-tests.yml` + `backend-unit-tests.yml` | Gates automáticos FE/BE en PR. |
| `.github/workflows/pr-e2e-tests.yml` + `pr-unified-storage-compatibility.yml` | Pipelines de compatibilidad y E2E como parte del change path. |
| `.github/workflows/detect-breaking-changes-levitate.yml` + `proto-breaking-changes.yml` | Detección de blast radius API/proto en CI. |
| `WORKFLOW.md` + `.github/CODEOWNERS` | Routing de ownership y flujo de release/contribución. |

---

## 6. Durable state

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `conf/defaults.ini` + `conf/sample.ini` | Estado de configuración versionado (defaults vs overrides). |
| `conf/provisioning/` | Provisioning declarativo de datasources/dashboards/plugins. |
| `pkg/services/sqlstore/migrations/` (vía AGENTS: migraciones DB) | Evolución durable del schema SQL. |
| `pkg/storage/unified/` + `pkg/storage/unified/migrations/` | Estado unificado de recursos + migraciones con reglas de compatibilidad. |
| `kinds/dashboard/` + `kinds/gen.go` | Schemas CUE como fuente durable que genera Go/TS. |
| `pkg/services/featuremgmt/` (vía `contribute/feature-toggles.md`) | Feature flags como estado de producto versionado y codegen. |

---

## 7. Demo / multi-env

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `.cursor/reset-demo.sh` + `.cursor/skills/reset-demo/` | Reset reproducible del estado demo (`origin/demo/start`). |
| `AGENTS.md` (bloque “Cursor 101/201 demo”) | Contrato multi-env de la demo: ramas, Jira SDFD, no arrancar webpack desde el agente. |
| `devenv/README.md` + `devenv/create_docker_compose.sh` | Multi-backend compose (influx, postgres, …) vía `make devenv`. |
| `devenv/datasources.yaml` + `devenv/datasources_docker.yaml` | Provisioning distinto local vs docker. |
| `devenv/docker/` + `Dockerfile` | Contenedores como entorno paralelo al `make run` + SQLite embebido. |
| `.nvmrc` + `mise.toml` | Pinning de toolchains por entorno (Node/Go) usado en Cursor Cloud vs login shell. |

---

## 8. Evaluation & failure modes

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `.cursor/skills/frontend-testing-strategy/SKILL.md` | Criterios anti-slop / anti-flake: cómo evaluar calidad de tests generados por agentes. |
| `e2e-playwright/smoke-tests-suite/` + `dashboard-cujs/` | Suites de humo y CUJ como oráculo de regresión. |
| `public/app/core/journeys/*.smoke.ts` | Smoke de journeys embebido junto a instrumentación. |
| `.github/workflows/check-frontend-test-coverage.yml` + `frontend-perf-tests.yaml` | Evaluación cuantitativa (cobertura, perf) en CI. |
| `.github/workflows/govulncheck.yml` + `trufflehog.yml` + `reject-gh-secrets.yml` | Failure modes de seguridad: secretos y vulns como hard fail. |
| `pkg/tests/` + `jest.config.js` | Pirámide de tests unit/integration como harness de evaluación local. |

---

## 9. Team operating model

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `CONTRIBUTING.md` + `contribute/README.md` | Entrada del modelo operativo humano. |
| `.github/CODEOWNERS` | Ownership por path → quién aprueba (y qué contexto debe cargar el agente). |
| `GOVERNANCE.md` + `MAINTAINERS.md` | Gobernanza y maintainers. |
| `contribute/ISSUE_TRIAGE.md` + `.github/ISSUE_TEMPLATE/` | Triage y plantillas: trabajo compartido humano/bot. |
| `.github/team-notifications-config.json` | Routing de notificaciones por equipo. |
| `AGENTS.md` (“Human Review Gates” / “Separate PRs FE/BE”) | Normas agent-facing del mismo modelo operativo (push, cadencias). |

---

## 10. Security & blast radius

| Path | Por qué ilustra el tema |
|------|-------------------------|
| `.policy.yml` (+ `.policy.yml.tmpl`) | Policy-as-code: qué cambios/paths/permisos están acotados. |
| `.github/workflows/create-security-branch.yml` + `create-security-patch-from-security-mirror.yml` | Flujo de seguridad aislado del `main` público. |
| `apps/secret/` + `devenv/secrets/` | Superficie de secretos (app + fixtures de env) — blast radius alto. |
| `pkg/middleware/` + `pkg/login/` + `apps/iam/` | AuthN/AuthZ y middleware: cambios de radio amplio. |
| `.github/workflows/sbom-on-release.yml` + `relyance.yaml` | Supply-chain / compliance en release. |
| `public/app/features/alerting/unified/AGENTS.md` (sección Dependency Security) | Reglas agentic: no `yarn add` sin aprobación, cuarentena 7 días, lockfile immutable. |

---

## Cómo usar este mapa en clase

1. Empezar siempre por `AGENTS.md` + `.cursor/` (contexto y topología).
2. Elegir un cambio acotado (p.ej. solo `public/app/features/...`) y contrastar con un cambio transversal (`kinds/` → codegen → `packages/grafana-schema` → CI breaking).
3. Para cada tema, pedir al alumnado que trace: **quién escribe el contexto**, **qué tool se invoca**, **qué CI valida**, **qué estado durable se toca**, **cuál es el blast radius**.

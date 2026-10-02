# Arquitectura Cursor avanzada — Capítulo 2 (contexto)

El capítulo 0 te dejó un marco: un agente propone transiciones de estado sobre un sistema que ya tiene dueños, cadencias y validadores. El capítulo 1 te dejó el mapa de fronteras (`pkg/`, `public/app/`, `packages/`, `apps/`, `kinds/`). Este capítulo responde a la pregunta que esos dos abren y no cierran: **qué tiene que haber leído el agente antes de tocar una frontera**. La respuesta no es "más prompt". Es una cascada de ficheros versionados — raíz, directorio, skill, style guide — donde cada capa añade lo que la anterior no puede saber, y ninguna copia a las otras.

[novarz/grafana](https://github.com/novarz/grafana) sigue siendo el laboratorio, no Grafana de producción. Parte de [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md) describe el harness de las demos Cursor 101/201 (Jira SDFD, `demo/start`, quién arranca el servidor). Esa parte no es arquitectura del producto. El patrón de capas sí se transfiere a cualquier monorepo.

*Fuente: rama `main` de novarz/grafana, leída el 2 oct 2026 · Companion: [outline.md](../outline.md) (Cap. 2) y [example-map.md](../example-map.md) (sección 2)*

## Cómo leer este capítulo

Misma estructura que los capítulos 0–1: objetivos, un deep-dive anclado a rutas reales, un laboratorio que se hace sobre el árbol (sin modificar novarz/grafana), una rúbrica y los anti-patrones. El entregable del laboratorio es texto: un AGENTS.md ficticio y una review de tres inconsistencias. No se mergea nada en el fork de Grafana.

Prerrequisito: los capítulos 0 y 1, y haber leído entero el [AGENTS.md raíz](https://github.com/novarz/grafana/blob/main/AGENTS.md). Si tu matriz de fronteras del capítulo 1 está a mano, úsala: cada fila va a pedir un contexto distinto.

## Capítulo 2 · Objetivos

Al terminar deberías poder diseñar la cascada de contexto de un cambio concreto. Hay cuatro capas, y cada una tiene un trabajo que las otras no hacen:

1. **Raíz.** Contrato que aplica en cualquier path: principios, puertas humanas, comandos, mapa de arquitectura, restricciones del harness.
2. **Directory-scoped.** Reglas de una frontera, activadas por directorio o por `globs`, que la raíz no puede detallar sin hincharse.
3. **Skill.** Procedimiento de un dominio (cómo escribir un test de frontend, cómo resetear la demo), invocado cuando el trabajo lo pide, no pegado dentro del AGENTS.
4. **Style guide.** Norma humana en `contribute/`. El AGENTS enlace; no la duplica.

El segundo objetivo es evitar el drift entre lo que lee una persona y lo que lee un agente. Si el style guide y el AGENTS dicen cosas distintas, el agente va a seguir el fichero que tenga más cerca del path, y el revisor humano va a seguir el otro. Esa divergencia es un bug de arquitectura, no un mal prompt.

> **La idea central del capítulo.** El contexto correcto es el de la frontera en la que estás trabajando. Cargar el AGENTS de alerting para un cambio en unified storage es tan incorrecto como no cargar ninguno.

## Capítulo 2 · Capas de contexto

### La cascada mental

Memorízala como orden de carga, no como cuatro documentos sueltos:

**raíz siempre + scoped si el path encaja + skill cuando el dominio lo pide + style guide por enlace, nunca por copy-paste.**

| Capa | Dónde vive en este repo | Cuándo entra | Qué no debe hacer |
| --- | --- | --- | --- |
| Raíz | [`AGENTS.md`](https://github.com/novarz/grafana/blob/main/AGENTS.md) (`version: 2.0.0`) | En todo turno | Detallar el patrón de un squad |
| Scoped | `**/AGENTS.md` con o sin `globs` | Si el path cae en el directorio o en el glob | Copiar el style guide o repetir el harness SDFD |
| Skill | [`.cursor/skills/frontend-testing-strategy/SKILL.md`](https://github.com/novarz/grafana/blob/main/.cursor/skills/frontend-testing-strategy/SKILL.md) y hermanas | Cuando el trabajo es de ese dominio | Sustituir al AGENTS de la frontera |
| Style guide | [`contribute/style-guides/`](https://github.com/novarz/grafana/tree/main/contribute/style-guides) y [`contribute/backend/`](https://github.com/novarz/grafana/tree/main/contribute/backend) | Cuando un AGENTS (o la tarea) enlace el fichero | Vivir duplicado dentro de un AGENTS |

La raíz es el único fichero que puedes asumir leído. Todo lo demás es condicional. Un agente que "por si acaso" carga alerting, journeys, storage y el style guide de docs en el mismo turno diluye las reglas que importan: las de la frontera bajo el cursor.

Esto encaja con el capítulo 1. Una frontera agent-safe tiene lenguaje, dueño y validación propios. El contexto scoped es la cuarta pata: las restricciones que solo valen dentro de esa frontera. Unified storage se despliega a otra cadencia que la capa de API; esa frase está en la raíz, y el procedimiento está en [`pkg/storage/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md). El agente que trabaja en `pkg/api/` sin cruzar esa línea no necesita el procedimiento. El que cruza la línea, sí, y deja de estar en un cambio de un turno.

### Anatomía del AGENTS.md raíz v2.0.0

El fichero abre con un comentario de versión, no con un título de marketing: `<!-- version: 2.0.0 -->`. La versión existe para poder hablar de drift. Si un scoped contradice a la raíz, la pregunta útil es "¿contra qué versión?". El dueño en CODEOWNERS es `@grafana/docs-tooling` (`/AGENTS.md`). El contexto global se gobierna como cualquier otro artefacto.

Recórrelo como contrato, sección por sección.

**Cursor 101/201 demo.** Es el harness, y solo vive aquí. Talk track en `.cursor/101-SCRIPT.md`. Reset con la skill `reset-demo` o `bash .cursor/reset-demo.sh`, que fija `main` a `origin/demo/start`. Tracker Jira SDFD, no Linear. SDFD-1, SDFD-2 y SDFD-28 no se implementan en `main`. Nadie arranca webpack, `yarn start`, `make run` ni grafana-server desde un terminal de agente; lo hace una persona en un login shell, con Node de nvm **24.11.0** (el mismo pin que [`.nvmrc`](https://github.com/novarz/grafana/blob/main/.nvmrc), que dice `v24.11.0`). Un AGENTS de squad que copie este bloque se queda desfasado el día que cambie el harness, y un agente que lo aplique fuera de la demo está obedeciendo una regla de escenario.

**Directory-scoped agent files.** La raíz enumera cuatro ficheros y dice que se lean al trabajar en esos directorios:

- [`docs/AGENTS.md`](https://github.com/novarz/grafana/blob/main/docs/AGENTS.md)
- [`public/app/features/alerting/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/AGENTS.md)
- [`pkg/storage/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md)
- [`public/app/core/journeys/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/AGENTS.md)

Esa lista es el índice humano de la cascada. En `main`, el 2 oct 2026, también hay AGENTS.md que la raíz no nombra: [`public/app/features/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/AGENTS.md), [`public/app/plugins/panel/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/plugins/panel/AGENTS.md), [`packages/grafana-ui/AGENTS.md`](https://github.com/novarz/grafana/blob/main/packages/grafana-ui/AGENTS.md), [`pkg/router/AGENTS.md`](https://github.com/novarz/grafana/blob/main/pkg/router/AGENTS.md), [`e2e-playwright/alerting-suite/AGENTS.md`](https://github.com/novarz/grafana/blob/main/e2e-playwright/alerting-suite/AGENTS.md), [`e2e-playwright/dashboard-new-layouts/AGENTS.md`](https://github.com/novarz/grafana/blob/main/e2e-playwright/dashboard-new-layouts/AGENTS.md), [`e2e-playwright/plugin-e2e/plugin-e2e-api-tests/AGENTS.md`](https://github.com/novarz/grafana/blob/main/e2e-playwright/plugin-e2e/plugin-e2e-api-tests/AGENTS.md) y [`public/app/features/expressions/components/SqlExpressions/SqlEditor/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/expressions/components/SqlExpressions/SqlEditor/AGENTS.md). Un agente que trate la lista de la raíz como cerrada se saltará, por ejemplo, las reglas del router al editar `pkg/router/`. El índice manual es útil y se pudre. Quien lo mantiene es el dueño de `/AGENTS.md`.

**Project Overview y Principles.** Grafana como monorepo Go + TypeScript/React, Yarn por un lado y Go workspaces por el otro. Los principios que ya usaste en el capítulo 1 están aquí: seguir el patrón del código vecino, tests para lo nuevo, cambios pequeños, **PRs separados de frontend y backend** porque se despliegan a cadencias distintas, y seguridad (XSS, SQL injection, command injection; los avisos de seguridad no se abren como issue en el repo). Para tests de frontend, la raíz no explica la estrategia: manda a la skill `frontend-testing-strategy`, y a `panel-testing-strategy` encima cuando el código es de visualización. Eso es la cascada bien hecha. La raíz nombra la skill; no pega las 385 líneas de [`.cursor/skills/frontend-testing-strategy/SKILL.md`](https://github.com/novarz/grafana/blob/main/.cursor/skills/frontend-testing-strategy/SKILL.md).

**Comments.** Un comentario solo si explica un *porqué* no obvio. Nunca un enlace de Slack, GitHub o Jira dentro de un comentario de código. La misma norma, para humanos, está desarrollada en [`contribute/style-guides/code-comments.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/code-comments.md). La raíz se queda con la regla que un agente viola más a menudo; el style guide se queda con el resto (estabilidad de API, deprecación).

**Human Review Gates.** Antes de `git push`, parar y pedir aprobación explícita. "Open a PR" en la descripción de una tarea es intención, no permiso. Regla de este curso cuando dos capas discrepan: obedece la más restrictiva (esta puerta gana a un scoped que diga "cuando pasen los tests, haz push") y trata la discrepancia como bug del contexto. El repo no escribe esa precedencia en ningún sitio; si no la fijas tú, el fichero más cercano al path gana por accidente.

**Commands.** Superficie sancionada, con targets que existen en el [Makefile](https://github.com/novarz/grafana/blob/main/Makefile) y en los scripts de `package.json`:

| Intención | Comando que la raíz nombra | Nota de lectura |
| --- | --- | --- |
| Backend con reload | `make run` | Lo arranca una persona, no el agente, en la demo |
| Unitarios Go | `go test -run TestName ./pkg/services/myservice/` y `make test-go-unit` | El path del ejemplo es ilustrativo; sustitúyelo por el paquete real |
| Integración Go | `make test-go-integration` | Postgres: `make devenv sources=postgres_tests,mysql_tests` y luego `make test-go-integration-postgres` (Key Notes) |
| Frontend | `yarn test path/to/file` | El script `test` de `package.json` es `jest --notify --watch`. En Cloud hay que salir del watch: ver abajo |
| E2E | `yarn e2e:playwright path/to/test.spec.ts` | |
| Lint Go / JS | `make lint-go`, `yarn lint`, `yarn typecheck` | `make lint-go` es también lo que pide [`contribute/backend/style-guide.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/style-guide.md) |
| Codegen | `make gen-go`, `make gen-cue`, `make gen-apps`, `make swagger-gen`, `make gen-feature-toggles`, `make update-workspace` | Mismas cadenas que el capítulo 1 |
| Deps | `yarn install --immutable` | |

`make test` también existe: depende de `test-go` y de `test-js`, o sea la suite entera. La tabla Commands de la raíz no lo lista. Un cambio en un paquete se valida con el target estrecho. Nombrar `make test` dispara el monorepo para un diff de una carpeta.

**Architecture.** Tablas de `pkg/` y de `public/app/`, más los paquetes `@grafana/data`, `@grafana/ui`, `@grafana/runtime`, `@grafana/schema`, `@grafana/scenes`, y tres apps de ejemplo (`apps/dashboard/`, `apps/folder/`, `apps/alerting/`). `pkg/tsdb/` aparece en la tabla de backend como "Time series database query backends". No hay un AGENTS scoped para ese directorio: hoy hereda solo la raíz. Ese hueco es el laboratorio.

**Key Notes.** Wire, CUE en `kinds/` vía `make gen-cue`, feature toggles en `pkg/services/featuremgmt/`, `go.work`, build tags, `conf/defaults.ini` frente a `conf/custom.ini`, migraciones en `pkg/services/sqlstore/migrations/`, sharding de CI con `SHARD`/`SHARDS`, y el puntero de compatibilidad: unified storage puede desplegarse a otra cadencia, y el detalle está en `pkg/storage/unified/AGENTS.md`. La raíz detecta la frontera; el scoped explica las tres reglas. Así se reparte una restricción sin duplicarla entera.

**Cursor Cloud specific instructions.** Gotchas de este entorno, no de un portátil cualquiera. Node: `.nvmrc` fijado con nvm, pero la infra inyecta `/exec-daemon/node` por delante, así que un shell no-login puede resolver un Node distinto. Los shells de login (`bash -lc`, tmux) sí ven el pin, porque `~/.bashrc` antepone nvm. Go sale de `/usr/local/go` (el de la distro en `/usr/bin/go` es más viejo); la versión que hay que igualar está en [`go.mod`](https://github.com/novarz/grafana/blob/main/go.mod) (`go 1.26.6` en esta lectura). Yarn entra por corepack; la versión es el campo `packageManager` de `package.json` (`yarn@4.17.1`). `.yarnrc.yml` pone `enableScripts: false`. Hace falta GCC para CGo/SQLite. Los repos de Cloud viven en `/agent/repos/<repo>`, no en el `~/grafana` que asume otra guía. Y el gotcha de tests que más agentes ignoran: `yarn test` lleva `--watch`. Para una pasada que termina hay que usar `yarn jest --no-watch` o `--watchAll=false`. Los paquetes Go grandes (`pkg/api/`) tardan del orden de dos minutos en compilar tests; por eso el comando con `-run TestName`.

> **Fallo de lectura frecuente.** La tabla Commands dice `yarn test`. El bloque de Cloud dice que ese script no termina. Las dos frases están en el mismo fichero. Un scoped que necesite tests de frontend debe citar el comando que sale (`yarn jest --no-watch`), que es el que ya usa el AGENTS de journeys.

### El patrón alerting: globs, enlaces, skill, seguridad

El mejor ejemplo de capa scoped del repo es [`public/app/features/alerting/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/AGENTS.md). CODEOWNERS asigna `/public/app/features/alerting/` a `@grafana/alerting-squad`. El fichero no vive en la raíz de esa feature: vive en `unified/`. Su frontmatter declara un glob más ancho que su carpeta:

```yaml
globs:
  - 'public/app/features/alerting/**'
```

Trabajar en `public/app/features/alerting/routes.tsx` cae dentro del glob aunque el AGENTS esté un nivel más abajo, en `unified/`. La raíz dice "léelo cuando trabajes en esos directorios". El glob dice el conjunto real de paths. Si los dos discrepan, el glob es el que acota; la lista de la raíz es el índice, y ya vimos que el índice se queda corto.

Fíjate en lo que el fichero **no** hace. Bajo "Grafana Coding Standards" hay una instrucción explícita: *Do not duplicate standards here — reference the source files.* Los enlaces son relativos y resolubles:

| Qué necesitas | A dónde apunta |
| --- | --- |
| Nombres, componentes, TypeScript | [`contribute/style-guides/frontend.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/frontend.md) |
| React Testing Library, `*ByRole`, `userEvent.setup()` | [`contribute/style-guides/testing.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/testing.md) |
| Emotion y `useStyles2` | [`contribute/style-guides/styling.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/styling.md) |
| Redux Toolkit | [`contribute/style-guides/redux.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/redux.md) |
| MSW, permisos, datasources de test | [`./TESTING.md`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/TESTING.md) (sí existe, al lado del AGENTS) |
| Tests sin slop | skill `frontend-testing-strategy`, leída con la herramienta de skills, no incrustada |

Eso es contexto **referencial**. El style guide sigue teniendo un solo original. El día que `testing.md` cambie `userEvent.setup()`, no hay que cazar copias dentro de cada squad.

Lo que el fichero **sí** añade es lo que ningún style guide global sabe:

- Dirección del squad: datos nuevos con RTK Query ([`api/alertingApi.ts`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/api/alertingApi.ts), [`api/alertRuleApi.ts`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/api/alertRuleApi.ts)). El AGENTS marca los reducers como legado y los sitúa en `state/reducers/`. En el árbol el fichero es [`state/reducers.ts`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/state/reducers.ts): un fichero, no un directorio. Misma lección que `routes.tsx` más abajo. La raíz describe el frontend que ya existe ("Redux Toolkit with slices"); alerting pide no abrir otro reducer. Léelas en ese orden: mapa en la raíz, política de código nuevo en el scoped. Tratadas como órdenes del mismo rango, se pisan.
- Clientes generados: preferir `@grafana/api-clients/rtkq/<grupo>/<versión>` y, si el cliente viene cojo, `enhanceEndpoints` con un `TODO` para quitar el parche. El wrapper vive en `api/`.
- Tests de alerting: MSW (`mockApi`), RBAC por defecto (`grantUserPermissions`), factories. `alertingFactory` se exporta desde [`mocks/server/db.ts`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/mocks/server/db.ts); otras factories están en [`mocks.ts`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/mocks.ts). La skill prohíbe el test de existencia; TESTING.md dice cómo se monta el servidor de mocks. Capas distintas, unidas por un enlace.
- Feature flags: leer por OpenFeature (`useFlagAlertingTriage`, o `getFeatureFlagClient()` fuera de React, incluido `utils/pageAccess.ts`). Leer `config.featureToggles` lo bloquea el lint `@grafana/no-config-feature-toggles`, porque ese mapa va vacío en el frontend multi-tenant y el flag sale siempre `false`. El catálogo está en `pkg/services/featuremgmt/toggles_gen.csv`; la migración, en `contribute/feature-toggles.md`. En tests, `setTestFlags` de `@grafana/test-utils/unstable`. La raíz solo dice dónde se **definen** los toggles y que se regeneran con `make gen-feature-toggles`. El fallo de leer el mapa vacío solo aparece en la capa de alerting. Sin esa capa, el agente escribe un `if (config.featureToggles.foo)` que compila y nunca es verdad en el sitio que importa.
- Un failure mode de navegación, en tabla: `createRelativeUrl` solo con componentes que renderizan un `<a>` nativo (`LinkButton`). Con `locationService` o `TextLink` el prefijo se duplica. Esto no está en el style guide de frontend; es un bug que el squad ya se ha comido.
- `gh` para **leer** contexto (issue, diff, checks). No para publicar. Encaja con la puerta humana de la raíz.
- "Learning from Corrections": si la corrección es un patrón reusable, se *propone* un añadido a este AGENTS. No se aplica sin aprobación. Es una puerta humana de segundo orden, sobre el propio contexto.
- **Dependency Security**, que el capítulo 10 generalizará. Mientras tanto, las reglas están escritas aquí y son operativas: no `yarn add` sin aprobación (paquete, motivo, fecha de publicación); cuarentena de 7 días comprobada con `npm view <package> time --json`; `yarn install --immutable`; no editar `yarn.lock` a mano; no relajar `enableScripts: false` (está en [`.yarnrc.yml`](https://github.com/novarz/grafana/blob/main/.yarnrc.yml)) sin `@grafana/frontend-ops`; paquetes sospechosos se marcan en el PR.

Hay un desfase numérico que conviene ver ahora, porque es exactamente el ejercicio de review. El AGENTS de alerting exige 7 días. `.yarnrc.yml` tiene `npmMinimalAgeGate: 3d`. No se contradicen en el sentido de que cumplir 7 días también cumple el gate de Yarn: la instrucción del squad es más estricta que la herramienta. Sí se pudren por separado. Quien baje el gate de Yarn a un día y no toque el AGENTS deja una política mentira, o al revés. Un contexto bueno nombra las dos cifras y dice cuál manda.

Último detalle de paths relativos, porque duele en la práctica. El AGENTS dice que las rutas están "Defined in `routes.tsx`". Ese nombre, resuelto al lado del fichero, sería `public/app/features/alerting/unified/routes.tsx`, y ahí no está. El módulo está en [`public/app/features/alerting/routes.tsx`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/routes.tsx), un nivel por encima de `unified/`, dentro del glob. Un agente obediente crea el fichero que el nombre relativo sugiere. La regla de estilo de este capítulo: en un AGENTS, los paths van completos desde la raíz del repo, o relativos con `../` explícito. Un nombre suelto no es un path.

Los hermanos cortos enseñan la otra punta del patrón. [`public/app/features/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/AGENTS.md) (dueño `@grafana/dataviz-squad` sobre ese fichero) no tiene frontmatter y no repite convenciones: solo dice cuándo invocar `frontend-testing-strategy`, `panel-testing-strategy` y `add-e2e-selectors`. [`packages/grafana-ui/AGENTS.md`](https://github.com/novarz/grafana/blob/main/packages/grafana-ui/AGENTS.md) y [`public/app/plugins/panel/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/plugins/panel/AGENTS.md) hacen lo mismo. Tres ficheros, una skill, cero copias del texto de la skill. Alerting es el caso largo (la frontera tiene mucha política propia). Estos tres son el caso mínimo, y también son válidos.

### docs/AGENTS.md frente a los style guides: cuándo docs y cuándo código

[`docs/AGENTS.md`](https://github.com/novarz/grafana/blob/main/docs/AGENTS.md) se titula "Documentation Style Guide for AI Agents" y va envuelto en los marcadores `<!-- docs-ai-begin -->` … `<!-- docs-ai-end -->`. No tiene `globs`. La raíz lo activa por directorio: trabajo bajo `docs/`. CODEOWNERS da `/docs/` al mismo equipo que la raíz, `@grafana/docs-tooling`.

Su contenido es de redacción, no de compilación. Nombres de producto (Grafana Alloy, luego Alloy; "Grafana Cloud" siempre entero; "OpenTelemetry", no "OTel"). Estructura de artículo Hugo: front matter entre `---`, el `title` igual que el h1, texto entre encabezados, prerrequisitos, next steps. Voz activa, "you", contracciones, present simple, allowlist/blocklist, enlaces inline, alt text sin "Image of…", shortcodes de admonition que no se rompen al editar. Nada de `make gen-go`, nada de Jest, nada de Wire.

[`contribute/style-guides/`](https://github.com/novarz/grafana/tree/main/contribute/style-guides) es otra capa. En este árbol hay nueve ficheros: `accessibility.md`, `code-comments.md`, `e2e-playwright.md`, `frontend.md`, `redux.md`, `storybook.md`, `styling.md`, `testing.md`, `themes.md`. Los leen personas que contribuyen código, y los agentes cuando un AGENTS de código los enlaza. El estilo de **backend** no está en esa carpeta. Está en [`contribute/backend/style-guide.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/style-guide.md) (golangci-lint vía `make lint-go` y [`.golangci.yml`](https://github.com/novarz/grafana/blob/main/.golangci.yml), `testing` + testify, `TestMain` con `pkg/tests/testsuite`, integración con prefijo `TestIntegration` y `testutil.SkipIntegrationTestInShortMode`). Al lado, [`contribute/backend/package-hierarchy.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/package-hierarchy.md) y [`contribute/backend/services.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/services.md). Un AGENTS de Go que enlace `contribute/style-guides/frontend.md` está en la frontera equivocada.

La decisión práctica:

| La tarea | Contexto que manda | Lo que no aplica |
| --- | --- | --- |
| Arreglar el editor de reglas de alerta | Raíz + AGENTS de alerting + skill de testing + los style guides que ese AGENTS enlaza | La voz de `docs/AGENTS.md` (contracciones, Hugo, "you") |
| Escribir una página bajo `docs/sources/` | `docs/AGENTS.md` | RTK Query, `make lint-go`, globs de alerting |
| Cambiar un backend Go en `pkg/` | Raíz + `contribute/backend/style-guide.md` por enlace + el scoped si el path tiene uno | `frontend.md`, la skill de frontend |
| Cruzar API y `pkg/storage/unified/` | El AGENTS de unified storage, además de la raíz | El recetario de journeys |

`contribute/` es proceso y estilo para humanos (`developer-guide.md`, guías de PR). El agente lo abre cuando un AGENTS lo enlaza o cuando la tarea es el proceso de contribución. No lo vuelca en el prompt "para tenerlo todo".

> **Regla de lectura.** `docs/AGENTS.md` gobierna `docs/`. Que se llame AGENTS.md no lo convierte en el estilo del repositorio. El estilo del código sigue en `contribute/`, y entra por enlace desde el AGENTS de la frontera.

### Compatibilidad como contexto: pkg/storage/unified/AGENTS.md

[`pkg/storage/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md) cabe en una pantalla y no tiene `globs`. La raíz lo cita desde Key Notes, así que la activación es doble: el path `pkg/storage/unified/` y cualquier cambio que la raíz ya marcó como "spanning API-layer callers and unified storage". CODEOWNERS cubre `/pkg/storage/` con `@grafana/grafana-search-and-storage`. El workflow que la capa nombra, [`.github/workflows/pr-unified-storage-compatibility.yml`](https://github.com/novarz/grafana/blob/main/.github/workflows/pr-unified-storage-compatibility.yml), es del mismo equipo.

El fichero parte el paquete en tres lados, con paths reales:

- **Cliente** (corre dentro de Grafana): `apistore/`, `federated/`, `client.go` / `client_retry.go`, y callers como `pkg/registry/apis/`, `pkg/services/{apiserver,dashboards,folder,search,stats}/`, `pkg/services/team/search/`, `pkg/infra/leaderelection/kvlease/`, `pkg/storage/legacysql/`.
- **Servidor** (puede desplegarse aparte): `resource/`, `sql/`, `search/`, `migrations/`, `parquet/`.
- **Contrato** (los dos lados): `proto/`, `resourcepb/`.

Y tres reglas que son failure modes, no consejos:

1. No muevas una responsabilidad de cliente a servidor en un solo PR. Primero el servidor, y el cliente en un PR posterior, cuando el servidor ya está publicado.
2. El contrato solo crece. No borres ni reutilices campos o RPCs de `proto/` / `resourcepb/` que una versión desplegada todavía use.
3. Si el cliente nuevo espera algo del servidor nuevo, deja un fallback hasta que el rollout termine.

El check de CI falla si un PR toca los dos lados. Si de verdad son inseparables, la etiqueta es `no-check-unified-storage-compatibility` y la justificación va en la descripción del PR. El propio fichero avisa de que el check no ve todos los importadores y de que un PR solo de contrato no lo dispara: la regla 2 sigue valiendo igual. Eso es contexto con el modo de fallo escrito al lado de la regla. Sin esa frase, un agente "arregla" el CI quitando el cambio de un lado y deja el protocolo roto.

Compara el tamaño con alerting. Unified storage no necesita un style guide de React ni una skill de testing de UI. Su contexto cabe en las reglas de compatibilidad porque **esa es la frontera**. Añadirle el bloque SDFD, o el recetario de MSW, sería ruido de otra capa. El capítulo 1 ya dijo que cruzar API ↔ storage saca el cambio del "un turno". Este fichero es la razón, en formato que un agente puede obedecer.

### Journeys: instrumentación con receta corta y documento largo

[`public/app/core/journeys/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/AGENTS.md) sí trae frontmatter, y el glob coincide con la carpeta:

```yaml
globs:
  - 'public/app/core/journeys/**'
```

CODEOWNERS: `/public/app/core/journeys/` es de `@grafana/dashboards-squad`. El fichero no reescribe la arquitectura. Manda a leer, en este orden, ficheros que están en el árbol:

1. [`./journey-tracking.md`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/journey-tracking.md) — la referencia canónica.
2. [`./searchToResource.ts`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/searchToResource.ts) — la forma del wiring.
3. [`./searchToResource.test.ts`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/searchToResource.test.ts) — la forma del test.
4. [`./__test-utils__/journeyTestHarness.ts`](https://github.com/novarz/grafana/blob/main/public/app/core/journeys/__test-utils__/journeyTestHarness.ts) — la única forma legítima de mockear el tracker.

Los tipos públicos salen de `@grafana/runtime`, fichero [`packages/grafana-runtime/src/services/JourneyTracker.ts`](https://github.com/novarz/grafana/blob/main/packages/grafana-runtime/src/services/JourneyTracker.ts): `registerJourneyTriggers`, `onJourneyInstance`, `JourneyMeta`, `JourneyHandle`. No se importa `JourneyTrackerImpl` ni `JourneyRegistryImpl`. Esas implementaciones viven en `public/app/core/services/journey/` (`JourneyTrackerImpl.ts`, `JourneyRegistryImpl.ts`), fuera del glob. El AGENTS las nombra para prohibirlas. Eso es un failure mode explícito.

La receta (tipo en `snake_case`, owner, timeout, `reportInteraction` con `silent: true` si el evento es solo de CUJ, wiring, import en [`public/app/app.ts`](https://github.com/novarz/grafana/blob/main/public/app/app.ts), tests) es la versión corta de los pasos 0–7 de `journey-tracking.md`. El atajo sancionado es `yarn cuj:new`, script real de `package.json` (`node scripts/cuj-new.ts`). Los tests se corren con `yarn jest --no-watch <camelCase>.test.ts`, que es el comando que el bloque de Cloud de la raíz pide y que la tabla Commands no escribe. El scoped corrige la raíz en el único punto donde la raíz es ambigua, sin repetir el resto.

El checklist de pre-merge y la sección "Common Mistakes" son la parte que un style guide genérico no puede escribir: no dejes un `StepHandle` a nivel de módulo; toda suscripción de `onInteraction` pasa por `collectUnsubs`; `recordEvent` no mide duraciones; el camino `discarded` tiene que existir o el journey muere por timeout. Cada bala es un fallo ya visto.

Dos sitios donde este AGENTS pide un path y el árbol responde con otro, y por eso el capítulo insiste en paths completos:

- El paso 3 dice "Add an entry to `journeyRegistry.ts`". En `public/app/core/journeys/` no hay ese fichero. El que existe es [`public/app/core/services/journey/journeyRegistry.ts`](https://github.com/novarz/grafana/blob/main/public/app/core/services/journey/journeyRegistry.ts), que es la ruta que escribe `journey-tracking.md`. Un agente que resuelva el nombre suelto al lado del AGENTS crea un segundo registro. El documento canónico y el AGENTS no dicen el mismo path. Manda el documento que el propio AGENTS declara como lectura obligatoria, y el AGENTS se corrige.
- Los smokes se typechequean con `yarn typecheck:smoke`, que es `tsc -p tsconfig.smoke.json`. Ese [`tsconfig.smoke.json`](https://github.com/novarz/grafana/blob/main/tsconfig.smoke.json) está en la **raíz** del repo, no dentro de `journeys/`. Incluye `scripts/cuj-smoke.ts`, `public/app/core/journeys/__smoke__/**/*.ts` y `public/app/core/journeys/*.smoke.ts`. El comentario del tsconfig explica por qué existe: el tsconfig principal excluye los `.smoke.ts` porque importan con extensión `.ts`, y activar `allowImportingTsExtensions` en global rompe el checker del dev server. El AGENTS lo resume; el tsconfig tiene el porqué. Otra vez, enlace antes que copia.

El import en `public/app/app.ts` es el único cruce de frontera que la receta autoriza, y lo dice con path completo. Un buen scoped no prohíbe en abstracto "no toques otros paquetes": dice qué fichero de fuera sí se toca, y por qué.

### Qué capa corresponde a cada frontera

Junta el mapa del capítulo 1 con la cascada. Antes de editar, señala la fila. Si la fila no existe, el contexto es la raíz más el style guide del lenguaje, y el laboratorio es escribir la fila que falta.

| Frontera (cap. 1) | Contexto que manda | Skill | Style guide, por enlace |
| --- | --- | --- | --- |
| Feature de alerting | `public/app/features/alerting/unified/AGENTS.md` (glob `public/app/features/alerting/**`) | `frontend-testing-strategy` | `frontend.md`, `testing.md`, `styling.md`, `redux.md`, más `./TESTING.md` |
| Visualización en `public/app/features/` o paneles | `public/app/features/AGENTS.md`, `public/app/plugins/panel/AGENTS.md` | esa skill, más `panel-testing-strategy` y `add-e2e-selectors` | los que la skill y el style guide de frontend ya cubren; no se pegan |
| Unified storage y callers que cruzan | `pkg/storage/unified/AGENTS.md` | ninguna de UI | las tres reglas de compatibilidad *son* el contexto |
| CUJ en `public/app/core/journeys/` | el AGENTS de journeys | no la de paneles, salvo que el cambio sea de visualización | `journey-tracking.md`, no `docs/AGENTS.md` |
| Páginas de producto en `docs/` | `docs/AGENTS.md` | — | el propio fichero es el style guide de docs |
| Router | `pkg/router/AGENTS.md` (no está en el índice de la raíz) | — | el fichero describe el paquete; no hereda reglas de alerting |
| `pkg/tsdb/` | solo la raíz, hoy | — | `contribute/backend/style-guide.md` cuando toque escribir Go |

Ejemplo de cascada bien aplicada. Tarea: cambiar el polling del listado de reglas en `public/app/features/alerting/unified/api/`. Entra la raíz (PR solo de frontend, no hagas push, no comentes con enlaces). Entra el glob de alerting. Entra la skill porque vas a tocar tests. Abres `TESTING.md` y `testing.md` por los enlaces; no los pegas en el chat. No aplica `docs/AGENTS.md`, ni las reglas de cliente/servidor de storage, ni `panel-testing-strategy`.

Ejemplo de cascada mal aplicada. La misma tarea, y el agente "también" reescribe un caller de `pkg/services/ngalert/` "ya que está", carga el AGENTS de storage porque ha visto la palabra *unified*, y mete frontend y backend en un PR. Ha cruzado dos fronteras del capítulo 1 y ha obedecido el contexto de una tercera. El contexto de alerting no le dio permiso para eso. La raíz se lo prohíbe.

## Capítulo 2 · Laboratorio

Dos partes. Las dos se escriben en este repo de curso o en un bloc de notas. No se hace commit en novarz/grafana.

### Parte A — Un AGENTS.md scoped para `pkg/tsdb/`

En este fork, [`pkg/tsdb/`](https://github.com/novarz/grafana/tree/main/pkg/tsdb) no es el catálogo completo de datasources de Grafana upstream. El 2 oct 2026 el directorio contiene exactamente: `Magefile.go`, `azuremonitor/`, `cloudwatch/`, `grafana-testdata-datasource/`, `grafanads/`, `graphite/`. Escríbelo así. Inventar `prometheus/` o `loki/` dentro de este árbol es romper la regla del curso: paths reales.

No hay AGENTS.md ahí. La raíz solo dice "Time series database query backends". Redacta el fichero que faltaría, en inglés (el resto de AGENTS.md del repo están en inglés; mezclar idioma dentro del contrato es otro drift), siguiendo el patrón de alerting:

- Frontmatter con `title`, `description` y `globs` acotado a `pkg/tsdb/**`. Un glob `pkg/**` también matchea storage y el router, y les impone reglas de datasource.
- Enlaces a [`contribute/backend/style-guide.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/style-guide.md), [`contribute/backend/package-hierarchy.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/package-hierarchy.md) y, si hablas de Wire, [`contribute/backend/services.md`](https://github.com/novarz/grafana/blob/main/contribute/backend/services.md). Ni una sección copiada.
- Comandos que existen: `go test -run TestName ./pkg/tsdb/<backend>/`, `make test-go-unit`, `make test-go-integration`, `make lint-go`. Integración con la forma del style guide (`TestIntegration` + `testutil.SkipIntegrationTestInShortMode`). `pkg/tsdb/cloudwatch/cloudwatch_integration_test.go` está en el árbol; no inventes el nombre de las funciones.
- Dueños distintos, porque la frontera de código no es la frontera de ownership. CODEOWNERS: `cloudwatch/`, `azuremonitor/` y `graphite/` son `@grafana/data-sources-plugins`; `grafana-testdata-datasource/` y `Magefile.go` son `@grafana/grafana-catalog`; `grafanads/` es `@grafana/grafana-backend-services-squad`.
- El UI de estos backends, cuando existe en este fork, está en [`public/app/plugins/datasource/`](https://github.com/novarz/grafana/tree/main/public/app/plugins/datasource) (`azuremonitor`, `cloudwatch`, `grafana`, `grafana-testdata-datasource`, `graphite`, y otros que no son de `pkg/tsdb`). Un cambio que toque el plugin React y el backend Go son dos PRs. Lo dice la raíz. El scoped lo repite en una línea, no reabre el debate.
- Codegen: [`pkg/tsdb/cloudwatch/kinds/dataquery/types_dataquery_gen.go`](https://github.com/novarz/grafana/blob/main/pkg/tsdb/cloudwatch/kinds/dataquery/types_dataquery_gen.go) abre con `Code generated - EDITING IS FUTILE` y dice que lo genera `public/app/plugins/gen.go` al correr `make gen-cue`. Azure Monitor tiene el hermano `pkg/tsdb/azuremonitor/kinds/dataquery/types_dataquery_gen.go`. El scoped debe prohibir editar esos `_gen.go` a mano. No inventes un segundo comando de generación.
- `pkg/tsdb/Magefile.go` expone `BuildPlugin` (SDK de plugins). No es el runner de tests. No mandes a nadie a un target `mage test` que ese fichero no define.
- Failure modes en bala: push sin aprobación, `yarn add` / `npm test` (el repo es Yarn; los tests de este directorio son Go), editar generados, un solo PR con `public/app/plugins/datasource/<nombre>/`.
- No copies el bloque SDFD ni el ensayo de PATH de Cloud. Eso ya está en la raíz, que siempre se carga.

Cuando lo tengas, míralo con la rúbrica de la evaluación. Luego compara con la clave, al final del capítulo.

### Parte B — Tres inconsistencias

Este borrador pretende ser un AGENTS de `pkg/tsdb/` y no lo es. Encuentra **tres** inconsistencias con la raíz o con el patrón de alerting. Cita la frase de la raíz (o el path) que cada una rompe. Hay una cuarta de propina; si la ves, apúntala, no hace falta para aprobar.

```markdown
---
title: All backend data
globs:
  - 'pkg/**'
---

# TSDB and friends

Paste of contribute/backend/style-guide.md follows so the agent does not need
to open another file.

(… aquí irían las secciones Linting, Testing, Globals, Pointers, Database …)

## Commands

Run `npm test` and `make test-tsdb` before you finish.
Datasource UI lives in the same PR; reviewers prefer one diff.

## Ship it

When tests pass, `git push -u origin HEAD` and `gh pr create`.
```

Pistas, no la solución: ¿el glob es una frontera del capítulo 1 o el backend entero? ¿esos comandos están en el Makefile o en la tabla Commands? ¿qué dice Human Review Gates, y qué dice la raíz de los PRs de frontend y backend?

## Capítulo 2 · Evaluación

Un contexto bueno aprueba las cuatro filas. Una sola en rojo suspende el fichero, aunque las otras estén bien: un glob enorme con comandos perfectos sigue cargándose en la frontera equivocada.

| Criterio | Aprueba | No aprueba |
| --- | --- | --- |
| Accionable | Nombra comandos reales (`make lint-go`, `go test -run … ./pkg/tsdb/cloudwatch/`) y el siguiente paso (regenerar con `make gen-cue`, parar antes del push) | "Sigue las buenas prácticas y corre los tests" |
| Scoped | El glob es la frontera (`pkg/tsdb/**`, `public/app/features/alerting/**`) y dice qué hay fuera que no debe mezclarse | `pkg/**`, `**/*`, o un fichero que también legisla docs y storage |
| Referencial | Enlaces a style guides, skills y al doc canónico del área | El style guide pegado, o la skill resumida "para no tener que abrirla" |
| Failure modes | Dice qué se rompe: push sin aprobación, `_gen.go` editado a mano, contrato de storage no aditivo, `config.featureToggles` siempre falso, `yarn test` que no termina | Solo el camino feliz, o "ten cuidado" |

Aplica la rúbrica a tu parte A y al borrador de la parte B. El borrador suspende las cuatro filas; tu fichero tiene que aprobar las cuatro. Una frase que sirve de prueba: ¿podría un agente, con tu fichero y la raíz, decidir sin preguntar si el cambio cabe en un turno y qué comando corre? Si la respuesta es "depende de cómo se interprete", no está accionable.

## Capítulo 2 · Anti-patrones

**Pegar el style guide dentro del AGENTS.** Alerting lo prohíbe en la primera sección de standards. El original de backend son unos cientos de líneas con ejemplos de mocks, `TestMain` y XORM. Pegarlo produce dos copias que divergen el primer día que alguien corrija `contribute/backend/style-guide.md` y no el AGENTS. El agente no gana contexto; gana una versión vieja con más peso en el prompt.

**Reglas contradictorias entre raíz y squad.** La puerta de `git push` es de la raíz. Un scoped que diga "abre el PR en cuanto pase CI" no es autonomía del equipo: es un agente que empuja porque el fichero más específico parecía ganar. Cuando de verdad quieras una excepción, escríbela como excepción ("en este paquete el check X se etiqueta así") y no como una puerta humana distinta. La cuarentena de 7 días frente a `npmMinimalAgeGate: 3d` es el caso límite: no se contradicen, pero si nadie escribe cuál manda, el siguiente editor va a "arreglar" una de las dos cifras y romper la otra.

**Contexto que no nombra comandos reales del Makefile.** `make test-tsdb` no es un target. `npm test` no es el package manager de este repo (`packageManager` es Yarn 4.17.1). `mage test` no está en `pkg/tsdb/Magefile.go`. Un agente con un comando inventado no falla de forma limpia: busca un sustituto, y el sustituto suele ser la suite entera o un script que se ha inventado. Los comandos estrechos ya están escritos. Cítalos.

**Índice de la raíz tratado como inventario cerrado.** Ya está desfasado respecto del árbol. El anti-patrón gemelo es el contrario: dar por cargado un AGENTS que el path no matchea. Ni el de alerting para `pkg/tsdb/`, ni el de docs para un handler Go.

**Paths de una sola palabra.** `routes.tsx` y `journeyRegistry.ts` están escritos así en ficheros reales, y el árbol los tiene en otro sitio. Cada vez que tu AGENTS nombre un fichero, escribe el path desde la raíz del repo y comprueba que el fichero está ahí. El laboratorio de este capítulo existe porque esa comprobación no es opcional.

## Clave del laboratorio

Léela después de escribir. Si tu parte A aprueba la rúbrica y no coincide frase por frase con esto, está bien: la rúbrica es el contrato, este texto es una solución de referencia.

### Parte B — las tres inconsistencias

1. **Glob `pkg/**`.** No es la frontera de `pkg/tsdb/`. También cubre `pkg/storage/unified/` (que tiene sus propias reglas de compatibilidad) y `pkg/router/` (que tiene su propio AGENTS). El patrón de alerting usa el glob de la feature, no `public/app/**`.
2. **Comandos que no existen.** `npm test` no es un script de este repo. `make test-tsdb` no está en el Makefile. Los reales, para este directorio, son `go test -run TestName ./pkg/tsdb/<backend>/`, `make test-go-unit`, `make test-go-integration` y `make lint-go`. `make test` sí existe y corre Go **y** frontend: no es el comando de un cambio scoped.
3. **`git push` y `gh pr create` al pasar los tests.** Rompe Human Review Gates: antes de `git push` se para y se pide aprobación; "open a PR" no es permiso. El `gh` que documenta alerting es de lectura. Además, "el UI del datasource en el mismo PR" rompe el principio de PRs separados de frontend y backend.

De propina: pegar `contribute/backend/style-guide.md` es el anti-patrón de duplicación. El fichero se enlaza.

### Parte A — solución de referencia

No la copies en novarz/grafana. Es material del curso.

```markdown
---
title: TSDB query backends
description: Agent rules for Go query backends under pkg/tsdb in this repo
globs:
  - 'pkg/tsdb/**'
---

# TSDB query backends — agent configuration

Context for AI agents working under `pkg/tsdb/`. This fork's tree (checked
against `main`) contains only:

- `pkg/tsdb/azuremonitor/`
- `pkg/tsdb/cloudwatch/`
- `pkg/tsdb/grafana-testdata-datasource/`
- `pkg/tsdb/grafanads/`
- `pkg/tsdb/graphite/`
- `pkg/tsdb/Magefile.go`

Do not invent sibling backends that are not in the tree.

Demo harness rules (Jira SDFD, who starts Grafana, Node PATH) stay in the
root `AGENTS.md`. Do not copy them here.

## Required reading

Do not duplicate these. Open them.

1. Backend style guide: `contribute/backend/style-guide.md`
   (`make lint-go`, testify, `TestMain` + `pkg/tests/testsuite`,
   integration tests named `TestIntegration` with
   `testutil.SkipIntegrationTestInShortMode`).
2. Package shape: `contribute/backend/package-hierarchy.md`.
3. Wire, only if the change touches service init: `contribute/backend/services.md`
   and then `make gen-go` from the root `AGENTS.md`.

## Ownership

The directory is one code boundary and three owners (`.github/CODEOWNERS`):

| Path | Owner |
| --- | --- |
| `pkg/tsdb/cloudwatch/`, `pkg/tsdb/azuremonitor/`, `pkg/tsdb/graphite/` | `@grafana/data-sources-plugins` |
| `pkg/tsdb/grafana-testdata-datasource/`, `pkg/tsdb/Magefile.go` | `@grafana/grafana-catalog` |
| `pkg/tsdb/grafanads/` | `@grafana/grafana-backend-services-squad` |

## Commands

- `go test -run TestName ./pkg/tsdb/cloudwatch/` (narrow the package)
- `make test-go-unit`
- `make test-go-integration`
- `make lint-go`

Do not use `npm test`, `yarn test` (that script is Jest, and it watches),
`make test` (runs Go and frontend), or a mage test target.
`pkg/tsdb/Magefile.go` defines `BuildPlugin` for plugin packaging, not tests.

## Frontend is a different PR

Matching UI, when this fork has it, lives under
`public/app/plugins/datasource/` (`azuremonitor`, `cloudwatch`, `grafana`,
`grafana-testdata-datasource`, `graphite`). Root `AGENTS.md` requires separate
frontend and backend PRs. Do not add that UI to a `pkg/tsdb/` change.

## Generated code

`pkg/tsdb/cloudwatch/kinds/dataquery/types_dataquery_gen.go` is generated by
`public/app/plugins/gen.go`. Regenerate with `make gen-cue` from the repo root.
Do not hand-edit it. `pkg/tsdb/azuremonitor/kinds/dataquery/types_dataquery_gen.go`
is the same kind of file. `pkg/tsdb/grafana-testdata-datasource/kinds/query.go`
is not marked generated; do not "clean it up" by running a generator you have
not found in the file header.

## Failure modes

- `git push` / `gh pr create` without the human review gate in the root `AGENTS.md`.
- Editing `*_gen.go` instead of the CUE source and `make gen-cue`.
- One PR that also touches `public/app/plugins/datasource/<name>/`.
- New dependencies (`go get` of a module the task did not name) without asking.
  Root security rules still apply: no XSS, SQL injection, or command injection.
- Assuming a backend directory that is not in the list at the top of this file.
```

Si tu versión añade un failure mode que esta clave no tiene, y el path es real, es mejor que la clave. Si nombra un comando que no está en el Makefile ni en `package.json`, vuelve a la rúbrica.

## Lo que viene

El capítulo 3 baja de "qué fichero se lee" a "quién actúa". La raíz ya nombra skills (`frontend-testing-strategy`, `panel-testing-strategy`, `reset-demo`) y varios AGENTS scoped solo contienen punteros a esas skills. Falta la topología: cuándo el agente generalista basta, cuándo se invoca una skill, cuándo el humano arranca Grafana, y por qué un skill que hace `git` irreversible sin confirmación no es un especialista, es un accidente. Trae el AGENTS de `pkg/tsdb/` que acabas de escribir. En el capítulo 3 se le va a pedir que declare qué skill no debe invocar, y esa frase es más difícil de lo que parece.

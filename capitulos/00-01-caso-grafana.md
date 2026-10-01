# Arquitectura Cursor avanzada — Capítulos 0–1 (caso Grafana)

Estos dos capítulos sientan la base de todo el curso: un agente en un monorepo no es un autocompletado con esteroides, es un componente más de un sistema con fronteras, estado durable, pipelines y personas que aprueban. El capítulo 0 cambia el marco mental (de "asistente de código" a "sistema agentic") usando [novarz/grafana](https://github.com/novarz/grafana) como laboratorio; el capítulo 1 convierte ese marco en algo operable, leyendo los módulos del monorepo (`pkg/`, `public/app/`, `packages/`, `apps/`, `kinds/`) como contratos con un radio de impacto y un camino de codegen propios. La conclusión práctica: antes de pedirle nada a un agente, Sergio, decide qué frontera va a cruzar, qué estado toca y quién tiene que aprobarlo.

*Fuente: rama `main` de novarz/grafana, leída el 1 oct 2026 · Basado en el outline y el mapa de ejemplos del curso*

## Cómo leer estos capítulos

Cada capítulo sigue la misma estructura: objetivos, un deep-dive largo con rutas reales del repo, un laboratorio que se hace sobre el árbol de ficheros (sin modificar el repositorio), criterios de evaluación y los anti-patrones que más daño hacen. Los laboratorios están pensados para hacerse con Cursor abierto sobre el repo, pero el entregable siempre es razonamiento escrito, no código mergeado. Prerrequisito: haber leído entero [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md) y hojeado [.cursor/101-SCRIPT.md](https://github.com/novarz/grafana/blob/main/.cursor/101-SCRIPT.md).

## Capítulo 0 · Objetivos

Al terminar este capítulo deberías ser capaz de distinguir con precisión entre dos formas de trabajar con IA en un repositorio. La primera es el *tool-use ad-hoc*: abres el chat, describes un cambio, el modelo lee algunos ficheros, edita, y tú juzgas el resultado a ojo. La segunda es una *arquitectura de agentes*: el agente opera dentro de fronteras declaradas, consume contexto diseñado a propósito, deja estado en sitios conocidos y entrega su trabajo a un pipeline (CI, CODEOWNERS, revisión humana) que lo valida. El segundo objetivo es mapear Grafana como sistema de estudio: un monorepo grande y real, una capa de demo de Cursor encima, y el proceso OSS de Grafana alrededor.

## Capítulo 0 · De asistente a sistema agentic

Empecemos por la diferencia que importa. Un asistente responde a una petición; un sistema agentic tiene **fronteras** (qué puede tocar), **estado** (qué sobrevive al chat) y **pipelines** (por dónde sale su trabajo y quién lo valida). En el modo asistente, la calidad depende casi por completo del prompt y de tu revisión. En el modo sistema, la calidad la sostienen piezas que existen fuera de la conversación: ficheros de contexto versionados, comandos canónicos, hooks, workflows de CI y reglas de ownership. Lo interesante es que en Grafana casi todas esas piezas ya existen, escritas para humanos, y el trabajo de arquitectura consiste en hacerlas legibles y vinculantes para un agente.

Mira cómo arranca [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md): *"This file provides guidance to AI agents when working with code in the Grafana repository."* No es un README. Lleva versión (`version: 2.0.0`), declara principios ("Separate PRs for frontend and backend changes (deployed at different cadences)"), fija una puerta humana explícita ("Before running `git push`, stop and get explicit human approval") y enumera comandos reales del Makefile y de Yarn. Cada una de esas líneas es una pieza de arquitectura: la versión permite razonar sobre drift, el principio FE/BE codifica una restricción de despliegue, la puerta humana define el borde del sistema autónomo y los comandos son la superficie de herramientas sancionada.

> **La idea central del capítulo.** Un agente en Grafana no "escribe código"; propone transiciones de estado sobre un sistema que ya tiene dueños, cadencias y validadores. Diseñar para eso es lo que separa un sistema agentic de un chat con permisos de escritura.

### Por qué un monorepo de ~3M LoC es el laboratorio correcto

Los repos pequeños engañan. En un proyecto de juguete todo cabe en contexto, no hay dueños distintos, el CI tarda segundos y un cambio "full stack" no rompe nada. Grafana, con del orden de tres millones de líneas, te quita esas muletas. Tiene límites reales: backend Go en [pkg/](https://github.com/novarz/grafana/blob/main/pkg/), frontend React en [public/app/](https://github.com/novarz/grafana/blob/main/public/app/), librerías publicadas en [packages/](https://github.com/novarz/grafana/blob/main/packages/), apps del App SDK en [apps/](https://github.com/novarz/grafana/blob/main/apps/) y schemas generativos en [kinds/](https://github.com/novarz/grafana/blob/main/kinds/). Tiene CI real: casi un centenar de workflows en [.github/workflows/](https://github.com/novarz/grafana/blob/main/.github/workflows/). Y tiene ownership real: [.github/CODEOWNERS](https://github.com/novarz/grafana/blob/main/.github/CODEOWNERS) ocupa unas 1.450 líneas y reparte el repo entre decenas de squads. Un agente que funciona bien aquí funciona bien en casi cualquier sitio; uno que solo funciona en repos pequeños no está diseñado, está teniendo suerte.

Hay además un detalle que hace de Grafana un caso especialmente didáctico: las fronteras no son solo de carpetas, son de **tiempo**. AGENTS.md avisa de que unified storage ([pkg/storage/unified/](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/)) "can be deployed as separate services at a different cadence than the Grafana API layer", y de que los cambios que cruzan ambas capas "must be backwards compatible in both directions". Es decir, dos ficheros en el mismo commit pueden acabar ejecutándose en versiones distintas en producción. Un agente que no sabe esto escribirá código que compila y pasa los tests locales y aun así rompe un despliegue.

### novarz/grafana como harness pedagógico, no como Grafana production

El fork novarz/grafana no es el Grafana que despliega Grafana Labs; es un *staging environment* de agentes montado para las demos Cursor 101/201. Conviene tenerlo muy claro, porque una parte de AGENTS.md describe ese harness y no el producto. El bloque "Cursor 101/201 demo" establece que el tracker es Jira SDFD ("Do not use Linear"), que SDFD-1, SDFD-2 y SDFD-28 no se implementan en `main` sino en una rama demo, que el punto de restauración es `demo/start` y que el reset se hace con la skill `reset-demo` o con [.cursor/reset-demo.sh](https://github.com/novarz/grafana/blob/main/.cursor/reset-demo.sh), que "pins `main` to `origin/demo/start`". También fija quién arranca qué: "Do not start webpack, `yarn start`, `make run`, or grafana-server from an agent terminal. The human starts Grafana in a login shell."

Lee esas reglas como lo que son: un **harness**. Un harness tiene un estado inicial conocido (la rama `demo/start`), una forma de volver a él (el script de reset), una división de responsabilidades (el humano levanta el servidor, el agente edita) y un sistema de tickets acotado (SDFD). Es exactamente lo que necesitas para que un experimento con agentes sea reproducible. La lección transferible no es "usa Jira SDFD", sino "todo entorno donde operan agentes necesita un punto de restauración, una frontera de procesos y un tracker con estados controlados". Lo que aprendas sobre seguridad, CI y ownership sí aplica al Grafana real; lo que aprendas sobre SDFD y ramas demo aplica al diseño de harnesses.

### Las cuatro capas del sistema

Para el resto del curso vamos a mirar el repo en cuatro capas que se superponen sobre el mismo árbol. La capa de **código** es lo que se compila y se despliega. La capa de **contexto agentic** es lo que el agente lee para decidir: AGENTS.md raíz, los AGENTS.md por directorio que la raíz enumera (como [docs/AGENTS.md](https://github.com/novarz/grafana/blob/main/docs/AGENTS.md) o [pkg/storage/unified/AGENTS.md](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md)) y las skills de [.cursor/](https://github.com/novarz/grafana/blob/main/.cursor/). La capa de **proceso humano** es cómo entra y sale el cambio: [contribute/](https://github.com/novarz/grafana/blob/main/contribute/), CODEOWNERS, las plantillas de PR. La capa de **CI y entorno** es lo que valida y ejecuta: los workflows, [devenv/](https://github.com/novarz/grafana/blob/main/devenv/), [conf/](https://github.com/novarz/grafana/blob/main/conf/), [e2e-playwright/](https://github.com/novarz/grafana/blob/main/e2e-playwright/). Un agente bien diseñado sabe en cada momento en qué capa está actuando y cuál le toca solo leer.

| Capa | Rutas representativas | El agente… |
| --- | --- | --- |
| Código | `pkg/`, `public/app/`, `packages/`, `apps/`, `kinds/` | Edita, dentro de una frontera |
| Contexto agentic | `AGENTS.md`, `.cursor/`, AGENTS.md por directorio | Lee siempre; edita solo si se le pide |
| Proceso humano | `contribute/`, `.github/CODEOWNERS` | Respeta; nunca sustituye |
| CI y entorno | `.github/workflows/`, `devenv/`, `conf/`, `e2e-playwright/` | Predice y ejecuta lo sancionado |

## Capítulo 0 · Laboratorio: inventario del top-level

Abre la raíz del repo en Cursor y haz un inventario completo de lo que cuelga de ella. Para cada carpeta y fichero relevante, asígnale una de cinco categorías: código, contexto agentic, proceso humano, CI o entorno. La gracia del ejercicio está en los casos ambiguos, así que no los esquives: [package.json](https://github.com/novarz/grafana/blob/main/package.json) es a la vez código (dependencias) y entorno (fija `packageManager: yarn@4.17.1` y `engines.node: ">= 22 <25"`); [conf/](https://github.com/novarz/grafana/blob/main/conf/) es configuración versionada que se comporta como estado durable; [docs/](https://github.com/novarz/grafana/blob/main/docs/) es producto para usuarios pero contiene su propio AGENTS.md. Cuando dudes, decide por el criterio "¿quién se rompe si esto cambia?" y escribe una línea justificando la elección.

Hazlo en dos pasadas. En la primera, clasifica sin ayuda del agente, solo mirando nombres y abriendo lo imprescindible. En la segunda, pídele a Cursor que haga la misma clasificación leyendo AGENTS.md primero, y compara. Las discrepancias son el resultado interesante: te dicen dónde el contexto actual del repo no basta para que un agente entienda el sistema, y eso alimentará directamente el capítulo 2 (arquitectura de contexto). Anota también qué carpetas el agente no menciona en absoluto; lo que el contexto no nombra, el agente tiende a tratarlo como si no existiera.

## Capítulo 0 · Evaluación

El entregable es un diagrama de una sola página con cuatro actores: **humano**, **agente**, **CI** y **estado durable**. Las flechas deben describir transiciones reales (el agente lee contexto, edita código, el humano aprueba el push, CI valida, el merge altera el estado) y cada flecha importante debe anclarse en al menos una ruta del repo. Exige un mínimo de cinco paths reales; una buena selección sería AGENTS.md (contrato del agente), `.cursor/reset-demo.sh` (restauración del harness), `.github/CODEOWNERS` (routing de aprobación), `.github/workflows/` (validación) y `kinds/` o `conf/` (estado que sobrevive). El diagrama aprueba si alguien que no conoce Grafana puede responder, solo mirándolo, a dos preguntas: dónde se detiene el agente y quién decide que un cambio entra.

## Capítulo 0 · Anti-patrones

El primer anti-patrón es **tratar AGENTS.md como un README de marketing**: leerlo una vez, encontrarlo razonable y no volver a mirarlo. AGENTS.md es un contrato operativo. Cuando dice "Separate PRs for frontend and backend changes" o "Before running `git push`, stop and get explicit human approval", no está sugiriendo estilo, está declarando restricciones del sistema. Un agente que lo ignora no es "creativo", está fuera de especificación. Fíjate además en que el propio fichero tiene dueño en CODEOWNERS (`/AGENTS.md @grafana/docs-tooling`): el contexto agentic se gobierna como cualquier otro artefacto.

El segundo es **ignorar CODEOWNERS y las cadencias FE/BE**. CODEOWNERS no es burocracia: es el mapa de quién responde de cada frontera, y por tanto quién va a revisar lo que el agente produce. Las cadencias distintas de frontend y backend (y de unified storage frente a la capa de API) significan que un cambio "coherente" en un commit puede no serlo en producción. Si tu flujo agentic produce PRs que mezclan squads y cadencias, estás generando trabajo de revisión que nadie puede aprobar limpiamente.

---

## Capítulo 1 · Objetivos

El capítulo 1 convierte el marco anterior en un mapa operativo. El primer objetivo es definir **fronteras agent-safe**: qué conjunto de ficheros puede tocar un agente en un solo turno sin romper el sistema ni obligar a coordinar varios equipos. El segundo es relacionar cada módulo principal (`pkg/`, `public/app/`, `packages/`, `apps/`, `kinds/`) con su radio de impacto y con su cadena de generación de código, porque en Grafana una parte importante del código no se escribe, se genera, y editar en el lado equivocado de esa cadena es uno de los errores más caros que comete un agente.

## Capítulo 1 · Módulos del monorepo como contratos

Una frontera es útil para un agente cuando cumple tres condiciones: tiene un lenguaje y un toolchain propios, tiene dueños identificables y tiene una forma de validarse por separado. Con ese criterio, el monorepo de Grafana se descompone así.

| Frontera | Rol | Toolchain | Radio de impacto |
| --- | --- | --- | --- |
| `pkg/` | Backend Go | Go, Wire, Make | Servidor completo |
| `public/app/` | Frontend | TS/React, Jest | La UI de Grafana |
| `packages/` | Librerías compartidas | Yarn workspaces, Nx | Grafana y plugins externos |
| `apps/` | Apps del App SDK | Go, CUE, App SDK | Un recurso y su API |
| `kinds/` | Schemas CUE | CUE, `make gen-cue` | Go y TS generados |

[pkg/](https://github.com/novarz/grafana/blob/main/pkg/) es el backend Go. AGENTS.md lo describe con su tabla interna (`pkg/api/` para handlers, `pkg/services/` para lógica de negocio por dominio, `pkg/server/` para la inicialización y Wire, `pkg/storage/`, `pkg/plugins/`…) y con una regla de diseño clara: "business logic in `pkg/services/<domain>/` not in API handlers". Para un agente esto define una frontera interna: un cambio de comportamiento de negocio debería vivir en `pkg/services/<dominio>/` y no extenderse a `pkg/api/` salvo que cambie el contrato HTTP. Y añade una trampa de codegen: si cambias la inicialización de servicios, hay que ejecutar `make gen-go` para regenerar Wire.

[public/app/](https://github.com/novarz/grafana/blob/main/public/app/) es el frontend: `core/` para servicios compartidos, `features/` para código por dominio, `plugins/` para plugins integrados y `store/` para Redux. La frontera agent-safe más natural del repo es un subdirectorio de `public/app/features/`, porque suele tener un único squad dueño. Por ejemplo, CODEOWNERS asigna `/public/app/features/dashboard/` y `/public/app/features/dashboard-scene/` a `@grafana/dashboards-squad`, aunque con excepciones finas (el `TransformationsEditor` es de `@grafana/datapro`, `ShareModal` de `@grafana/sharing-squad`). Incluso dentro de una "feature" hay sub-fronteras de ownership; el agente debería saber en cuál está.

[packages/](https://github.com/novarz/grafana/blob/main/packages/) es donde viven las librerías `@grafana/*`: `grafana-data`, `grafana-ui`, `grafana-runtime`, `grafana-schema`, `grafana-e2e-selectors`, `grafana-i18n` y otras. Aquí el radio de impacto cambia de naturaleza: estos paquetes no solo los consume `public/app/`, también los consumen plugins de terceros. Son la **API interna entre equipos** y, en parte, una API pública.

[apps/](https://github.com/novarz/grafana/blob/main/apps/) contiene apps autónomas del App SDK: `advisor`, `alerting`, `annotation`, `dashboard`, `folder`, `iam`, `playlist`, `provisioning`, `secret`, `shorturl` y más. Cada una es un módulo Go propio con sus schemas CUE. Y [kinds/](https://github.com/novarz/grafana/blob/main/kinds/) contiene los schemas CUE de dashboard y panel, junto a [kinds/gen.go](https://github.com/novarz/grafana/blob/main/kinds/gen.go); según AGENTS.md, "Dashboard/panel schemas in `kinds/` generate both Go and TS code via `make gen-cue`". Es la frontera con mayor radio de impacto relativo a su tamaño: unas pocas líneas de CUE se propagan a Go, a TypeScript y a `@grafana/schema`.

### Workspaces duales: go.work frente a Yarn y Nx

El monorepo tiene dos sistemas de workspaces que no se conocen entre sí, y entender ambos es entender qué es "un módulo" para cada toolchain. En el lado Go, [go.work](https://github.com/novarz/grafana/blob/main/go.work) declara `go 1.26.6` y una lista `use (...)` con el módulo raíz (`.`), cada app de `apps/` (incluidas cuatro sub-apps de alerting: `alertenrichment`, `alertrulequality`, `notifications`, `rules`) y varios módulos de `pkg/` que se versionan por separado, como `pkg/apimachinery`, `pkg/apiserver`, `pkg/plugins`, `pkg/codegen` o `pkg/storage/unified/resourcepb`. Cada entrada es una frontera de compilación y de dependencias: un agente que añade un módulo Go nuevo debe ejecutar `make update-workspace`, tal como recuerda AGENTS.md.

En el lado JavaScript, [package.json](https://github.com/novarz/grafana/blob/main/package.json) declara los workspaces `packages/*`, `public/app/plugins/*/*` y `e2e-playwright/test-plugins/*`, y [nx.json](https://github.com/novarz/grafana/blob/main/nx.json) añade caché para los targets `build` (con salidas en `{projectRoot}/dist`) y `generate`, con `defaultBase: "main"`. Esa última línea importa: Nx calcula qué proyectos están "afectados" comparando con `main`, lo cual da al agente una forma sancionada de preguntar "¿qué toca mi cambio?" en vez de adivinarlo. Hay además plugins integrados que necesitan build propio (`azuremonitor`, `loki`, `grafana-testdata-datasource`), un recordatorio de que "frontend" no es un bloque único.

La consecuencia arquitectónica es que la pregunta "¿esto es un cambio pequeño?" no tiene respuesta en líneas de código, sino en cuántos workspaces cruza. Un cambio que vive en un solo workspace se puede compilar, testear y revisar por separado. Uno que cruza `go.work` y Yarn a la vez ya es un cambio FE+BE, y AGENTS.md pide separarlos en PRs distintos.

### Apps SDK frente a pkg/services

Grafana conserva dos formas de construir backend, y un agente necesita saber cuál está tocando. La tradicional es `pkg/services/<dominio>/`: servicios Go inyectados con Wire, consumidos por handlers de `pkg/api/` y persistidos vía `sqlstore`. La nueva es `apps/<nombre>/`: módulos autocontenidos que definen sus recursos en CUE y se registran mediante el App SDK. Que convivan no es desorden, es una migración en curso, y la documentación lo explica con mucho detalle.

La lectura guiada del capítulo es [contribute/architecture/k8s-inspired-backend-arch.md](https://github.com/novarz/grafana/blob/main/contribute/architecture/k8s-inspired-backend-arch.md). Su tesis: Grafana adopta un modelo orientado a recursos al estilo Kubernetes, con Resource APIs bajo `/apis/<group>/<version>/namespaces/<namespace>/<resource>/<name>`, y "the end goal is for the Resource APIs to become the only interface for managing core Grafana resources". Distingue dos enfoques de implementación. El *Registry Approach* (`pkg/registry/apis/...`) es el original, en Go. El *Apps Approach* (`apps/...`) es "the direction Grafana is moving towards for all resources": cada app define recursos en `kinds/*.cue` y los registra vía App SDK. El ejemplo canónico es la app de playlist: `apps/playlist/kinds/playlist.cue` define el schema, `pkg/apis/playlist/v1/` contiene los tipos Go generados, un reconciler actúa sobre los recursos y `pkg/registry/apps/playlist/register.go` registra el kind.

Para nuestro propósito, la parte más importante es la separación entre la **capa de API** y **unified storage**. El documento define unified storage (`pkg/storage/unified/...`) como "Grafana's internal persistence layer backing the Resource API", que "decouples API handlers from the storage backend", implementa la `storage.Interface` de Kubernetes sobre SQL (PostgreSQL, MySQL, SQLite, no etcd) y usa `resourceVersion` para concurrencia optimista: si la versión almacenada difiere, la actualización se rechaza con un 409. También describe la migración por recurso en nueve pasos, con feature flags (por ejemplo `kubernetesClient<Resource>`) y versiones alpha, beta y GA. Si juntas esto con la advertencia de AGENTS.md sobre cadencias de despliegue distintas, el resultado es una frontera dura: cualquier cambio que toque a la vez callers de la capa de API y `pkg/storage/unified/` es, por definición, un cambio de compatibilidad, y debe leerse [pkg/storage/unified/AGENTS.md](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md) antes de escribir una línea.

> **Regla práctica.** Si el cambio cruza la línea API ↔ unified storage, deja de ser un cambio "de un turno". Necesita plan de compatibilidad en ambas direcciones y una revisión de quien es dueño de esa capa.

### Paquetes compartidos como API interna

Los paquetes `@grafana/data` (estructuras de datos), `@grafana/ui` (componentes), `@grafana/runtime` (servicios en ejecución) y `@grafana/schema` (tipos generados desde CUE) son la frontera más engañosa para un agente, porque parecen código de la casa y se comportan como una API pública. Un cambio de firma en un componente de `@grafana/ui` puede compilar perfectamente dentro del monorepo y aun así romper plugins externos que dependen de la versión publicada. Por eso el capítulo 5 hablará del workflow de detección de breaking changes; aquí basta con interiorizar que tocar `packages/` es cambiar un contrato.

El ownership confirma esa lectura. CODEOWNERS asigna `/packages/grafana-ui/` a `@grafana/grafana-frontend-platform`, pero componentes concretos como `BarGauge`, `DataLinks` o `RadialGauge` son de `@grafana/dataviz-squad`. `@grafana/data` está repartido por subcarpetas entre más de media docena de squads (`src/panel/` y `src/field/` de `@grafana/dashboards-squad`, `src/transformations/` de `@grafana/datapro`, `src/rbac/` de `@grafana/access-squad`…). Y `/packages/grafana-schema/` es de `@grafana/grafana-app-platform-squad`, con la parte de dashboard delegada a `@grafana/dashboards-squad`. Un paquete compartido es, literalmente, un sitio donde varios equipos firman el mismo contrato.

`@grafana/schema` merece mención aparte porque es la desembocadura de la cadena `kinds/ → make gen-cue → packages/grafana-schema`. Sus tipos no se escriben a mano. Un agente que "arregla" un tipo directamente ahí está editando un artefacto generado que el siguiente `make gen-cue` sobrescribirá, y probablemente rompiendo la coherencia entre el schema y lo que valida el backend.

## Capítulo 1 · Laboratorio: el título del panel

Plantéate un bug ficticio: "cambiar cómo se muestra o se edita el título de un panel". Es deliberadamente ambiguo, porque la misma frase describe cambios con radios de impacto muy distintos. Tu trabajo es trazar dos versiones del cambio y listar los ficheros y comandos que tocaría cada una.

En la **versión FE-only**, el título ya existe en el modelo (el schema de dashboard en [kinds/dashboard/dashboard_kind.cue](https://github.com/novarz/grafana/blob/main/kinds/dashboard/dashboard_kind.cue) define un campo `title?: string` para paneles) y solo cambia la presentación o la edición. El cambio vive en `public/app/features/dashboard-scene/` o `public/app/features/dashboard/`, tiene un único squad dueño, se valida con tests Jest del área (aplicando la skill `frontend-testing-strategy`, como pide AGENTS.md) y es un buen candidato para un único turno de agente. Identifica los componentes concretos que tocarías y qué tests mínimos añadirías.

En la **versión schema change**, el cambio exige un campo nuevo o un cambio de semántica del título en el modelo persistido. Ahora la traza empieza en `kinds/`, pasa por `make gen-cue`, regenera tipos en Go y en `packages/grafana-schema`, probablemente afecta a la app de dashboard en [apps/dashboard/](https://github.com/novarz/grafana/blob/main/apps/dashboard/) (de `@grafana/grafana-app-platform-squad` y `@grafana/dashboards-squad`) y solo al final llega a la UI. Escribe el orden exacto de ficheros y comandos, marca cuáles son generados y cuáles escritos a mano, y separa lo que irá en un PR de backend y lo que irá en uno de frontend.

Como segunda parte, compara dos cambios aparentemente parecidos: modificar un componente de `packages/grafana-ui` (por ejemplo, el comportamiento de un input que se usa para títulos) frente a modificar directamente el editor de paneles en `public/app/features/dashboard`. Para cada uno, responde a tres preguntas: quién lo consume además de Grafana, qué squad lo revisa según CODEOWNERS y qué podría romperse fuera del repo. La diferencia que deberías encontrar es que el primero cambia un contrato con consumidores externos y el segundo cambia una implementación con un solo consumidor.

## Capítulo 1 · Evaluación

El entregable es una matriz con una fila por cambio y cuatro columnas: frontera cruzada, tests mínimos, owners según CODEOWNERS y codegen requerido. Debe cubrir al menos los dos casos del laboratorio y añadir tres más de tu elección, uno de los cuales debe cruzar la línea API ↔ unified storage.

| Cambio | Frontera | Tests mínimos | Owners |
| --- | --- | --- | --- |
| Título de panel, solo UI | `public/app/features/dashboard-scene/` | Jest del área | `@grafana/dashboards-squad` |
| Campo nuevo en schema | `kinds/` → `apps/dashboard/` → UI | Go + Jest, en PRs separados | App platform + dashboards |
| Componente en grafana-ui | `packages/grafana-ui/` | Jest + revisión de API | `@grafana/grafana-frontend-platform` |

*Las tres filas son el arranque; complétalas con codegen requerido y añade tus propios casos.*

La matriz aprueba si cada fila permite a un agente decidir, sin preguntar, si el cambio cabe en un turno, cuántos PRs necesita y a quién va a llegar la revisión. Una fila que dice "tests: los necesarios" o "owners: el equipo" no aprueba.

## Capítulo 1 · Anti-patrones

El primer anti-patrón es el **PR "full stack"** cuando AGENTS.md pide explícitamente separar frontend y backend porque se despliegan con cadencias distintas. Es tentador, porque al agente le cuesta lo mismo editar un fichero Go que uno TSX, y el resultado parece más completo. Pero ese PR mezcla revisores, mezcla pipelines de CI y crea una ventana en la que el backend nuevo convive con el frontend viejo (o al revés) sin que nadie haya pensado en ello. La separación no es una molestia de proceso: es la forma que tiene el sistema de hacer explícita la compatibilidad.

El segundo es **editar artefactos generados a mano**. En Grafana hay al menos cinco cadenas de generación documentadas en AGENTS.md: Wire (`make gen-go`), CUE (`make gen-cue`), App SDK (`make gen-apps`), OpenAPI (`make swagger-gen`) y feature toggles (`make gen-feature-toggles`). Cada una tiene una fuente de verdad y unos ficheros de salida. Un agente que corrige un tipo en `packages/grafana-schema` o en `wire_gen.go` produce un diff que parece correcto, pasa la revisión superficial y desaparece (o se corrompe) en la siguiente regeneración. La regla es simple: si un fichero es generado, el cambio se hace en su fuente y se regenera con el comando sancionado.

## Lo que viene

Con el marco del capítulo 0 y el mapa de fronteras del capítulo 1, el capítulo 2 entra en la arquitectura de contexto: cómo se escriben y se escalonan los AGENTS.md (raíz, por directorio, skills) para que un agente cargue exactamente las restricciones de la frontera en la que está trabajando, ni más ni menos. Guarda el inventario y la matriz que has producido aquí: son la materia prima del siguiente laboratorio.

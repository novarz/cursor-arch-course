# Capítulo 2 — Apilar contexto para un agente

Este capítulo se lee sin conocer Grafana. Al terminar tienes que poder explicar en voz alta cómo apilar contexto: raíz, zona, procedimiento, guía por enlace. Grafana es un ejemplo por principio, no un recorrido de ficheros. Las rutas de apoyo siguen en [example-map.md](../example-map.md).

Cada idea usa el mismo bloque: **qué es → por qué importa → best practice → ejemplo corto → ejercicio mental**.

Hace falta el capítulo 1: una frontera es un contrato, y el contexto que cargas es el contrato de esa frontera.

*Hechos de Grafana comprobados en `main` de novarz/grafana el 2 oct 2026.*

**Objetivos.** Diseñar la pila de contexto de un turno. Escribir reglas de una zona que no copien la guía humana ni choquen con la raíz. Revisar contexto ajeno con la rúbrica de abajo.

### El contexto se apila en cuatro capas

1. **Qué es** — El contexto son las instrucciones que el agente carga antes de editar. Se apilan de lo general a lo específico. (1) **Raíz:** el contrato de cualquier carpeta del repo. (2) **Zona:** un fichero que se enciende con un glob — un glob es un patrón de rutas, por ejemplo `equipo/pagos/**`, y solo entonces esas reglas entran en el turno. (3) **Skill:** un procedimiento que se abre cuando la tarea lo pide (cómo testear, cómo resetear un ensayo), no en cada edición. (4) **Guía humana:** el documento de estilo del equipo, al que se llega por un enlace. Cada capa añade. Ninguna repite a la de arriba.
2. **Por qué importa** — Un fichero único y enorme se desfasa, y el agente se trae al PR las normas de otra zona. Cargar de menos deja el turno solo con las reglas generales y pisa las del equipo dueño. En los dos casos el review discute un contrato que no es el de esa frontera, y el blast radius del diff sorprende a quien aprueba.
3. **Best practice** — Antes de editar, escribe la pila del turno en una línea: raíz + fichero de zona + skill si la tarea lo pide + enlaces. Nombra también las capas que dejas cerradas.
4. **Ejercicio mental** — Vas a cambiar un test de una sola feature. ¿Qué capas abres y cuál dejas cerrada?

### La raíz es el contrato de cualquier PR

1. **Qué es** — El fichero raíz dice lo que es verdad en todas las carpetas: principios, puertas humanas, comandos canónicos y un índice corto de dónde viven las reglas locales. Conviene que lleve versión, para poder hablar de si está desfasado. El manual de un equipo no va aquí.
2. **Por qué importa** — Casi todo turno empieza en la raíz. Si la raíz está llena de normas de un squad, esas normas contaminan PRs de otros squads y el review hereda un contrato ajeno. Si en la raíz faltan la puerta humana y los comandos, el agente publica y valida con lo que invente.
3. **Best practice** — En la raíz solo entran reglas que aplicarías a un PR de cualquier carpeta. Lo demás baja a un fichero de zona o a una skill.
4. **Ejemplo (Grafana, opcional)** — [`AGENTS.md`](https://github.com/novarz/grafana/blob/main/AGENTS.md) (versión 2.0.0) es ese contrato: PRs separados cuando las cadencias difieren, aprobación humana antes de `git push`, y los comandos reales del repo. Yarn, que aparece en esos comandos, es el gestor de paquetes del frontend; el Makefile es la lista de tareas canónicas (compilar, testear, generar).
5. **Ejercicio mental** — Cita tres reglas que deban estar en la raíz de tu repo y una que estaría de más ahí.

### La zona se carga solo dentro de su glob

1. **Qué es** — El fichero de zona añade el contrato de una frontera: patrones locales, enlaces y modos de fallo de ese trozo. El glob lo enciende cuando los ficheros del turno caen dentro del patrón. Fuera del patrón, esas reglas no forman parte del turno.
2. **Por qué importa** — Sin glob, la norma de un equipo se vuelve norma del monorepo y el review de otro equipo la hereda. Un glob más ancho que la frontera agranda el blast radius de la regla. Un glob más estrecho deja al agente editando al lado de la frontera sin verla. El PR sale con la norma equivocada y el dueño real no estaba en la conversación.
3. **Best practice** — El glob coincide con la frontera del capítulo 1: la carpeta del contrato, ni el repo entero ni un fichero suelto si la norma es de la carpeta.
4. **Ejemplo (Grafana, opcional)** — [`public/app/features/alerting/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/public/app/features/alerting/unified/AGENTS.md) declara el glob `public/app/features/alerting/**`. Esas normas entran al trabajar en esa feature.
5. **Ejercicio mental** — Si el glob fuera `**/*`, ¿qué PR de otro equipo quedaría sujeto a reglas que no le tocan?

### Una skill es un procedimiento con disparador

1. **Qué es** — Una skill es un procedimiento con un disparador claro: cómo escribir tests, cómo volver a un estado de demo, cómo añadir un selector de prueba. El agente la abre cuando la tarea coincide con el disparador. No viaja en el contexto de cada edición.
2. **Por qué importa** — Si el procedimiento vive en la raíz, se carga en turnos que no lo necesitan y envejece lejos de quien lo mantiene. Si vive solo en la cabeza del revisor, el agente entrega trabajo que el review rechaza por un criterio que no estaba escrito. La skill es el sitio del criterio que se repite cada vez que la tarea aparece.
3. **Best practice** — «Cuando hagas X, sigue estos pasos» es una skill, enlazada desde el fichero de la frontera. «En esta carpeta no hagas Y» es una regla de zona, no una skill.
4. **Ejemplo (Grafana, opcional)** — Para tests de frontend, el contrato manda invocar la skill [`frontend-testing-strategy`](https://github.com/novarz/grafana/blob/main/.cursor/skills/frontend-testing-strategy/SKILL.md). El procedimiento vive en la skill.
5. **Ejercicio mental** — Una checklist de tests de tu equipo. ¿Aplica en cada edición o solo cuando alguien escribe tests? Eso decide si es regla o skill.

### La guía humana se enlaza

1. **Qué es** — La guía de estilo la mantienen las personas, en la documentación del repo. El fichero del agente guarda el enlace y, como mucho, la excepción local que la guía no cubre. El texto largo tiene un solo sitio.
2. **Por qué importa** — Copiar la guía dentro de las instrucciones duplica la norma. El día que la guía cambia, el agente obedece la copia vieja y el review obedece la nueva. El PR tiene dos contratos. El blast radius de ese desfase es cada turno hasta que alguien nota la contradicción.
3. **Best practice** — Una regla, un sitio. El fichero del agente enlaza. Ahí solo se escribe lo que la guía no dice y esta frontera necesita.
4. **Ejemplo (Grafana, opcional)** — La guía de frontend está en [`contribute/style-guides/frontend.md`](https://github.com/novarz/grafana/blob/main/contribute/style-guides/frontend.md). El fichero de zona del principio del glob enlaza esa guía y pide dejar los estándares en ella.
5. **Ejercicio mental** — Si mañana cambia una convención de nombres, ¿cuántos ficheros habría que tocar para que el agente se entere? La respuesta sana es uno.

### La zona especializa; no revoca en silencio

1. **Qué es** — La capa de abajo puede concretar la raíz. No puede anularla sin decirlo. Si la raíz pide separar cambios que se despliegan en ritmos distintos y la zona pide subirlos juntos, el agente cumple una norma y el review exige la otra.
2. **Por qué importa** — La contradicción no se ve en el diff. Se ve con el PR ya abierto, cuando dos textos versionados discrepan. El equipo deja de fiarse del contexto y vuelve a pegar reglas en el chat. A partir de ahí cada turno tiene el blast radius de una norma oral.
3. **Best practice** — Al añadir una regla de zona, busca en la raíz la frase del mismo tema. Si chocan, o la raíz nombra la excepción, o reescribes la regla local. Dos verdades a la vez no son un contrato.
4. **Ejemplo (Grafana, opcional)** — [`pkg/storage/unified/AGENTS.md`](https://github.com/novarz/grafana/blob/main/pkg/storage/unified/AGENTS.md) concreta la cadencia de la raíz: cliente y servidor pueden salir por separado, así que cualquier mezcla de versiones tiene que seguir funcionando y el cambio de responsabilidad va en PRs partidos.
5. **Ejercicio mental** — Inventa una regla de squad que choque con «separa por cadencia». ¿Cuál gana, y qué frase habría que añadir a la raíz para que la excepción sea legal?

### El contexto nombra el comando y el modo de fallo

1. **Qué es** — Un contexto accionable dice el comando que ya existe en el repo para validar, y el modo de fallo: qué se rompe, y en qué orden, si lo ignoras. «Sigue las convenciones» no cambia ninguna decisión. «Ejecuta este comando y no edites la salida» sí.
2. **Por qué importa** — En el hueco, el agente inventa un comando. El review recibe un PR que no corre lo que corre el CI. El modo de fallo es lo que permite parar antes de cruzar la frontera: compatibilidad entre versiones, fichero generado, secreto, test que no prueba comportamiento.
3. **Best practice** — Cada fichero de zona incluye al menos un comando copiado de la lista canónica del repo y dos frases del tipo «si haces X, se rompe Y».
4. **Ejemplo (Grafana, opcional)** — La raíz documenta `make gen-go` para regenerar el arranque de servicios después de cambiar cómo se inicializan. Wire es el generador que escribe ese arranque a partir de la lista de dependencias. Sin el comando escrito, el arreglo acaba en la salida generada. [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md)
5. **Ejercicio mental** — Una instrucción que diga «lanza los tests». ¿Sabrías el comando exacto sin preguntarle a nadie? Si no, el agente tampoco.

## Capítulo 2 · Laboratorio

Escribe un fichero de contexto de zona, ficticio, para una frontera de un solo equipo. Secciones obligatorias:

- glob que coincida con esa frontera y con nada más
- enlaces a la guía humana (el texto de la guía no se pega)
- una skill, o en su defecto un comando de test sacado de la lista canónica del repo
- tres reglas que no estén ya en la raíz
- dos modos de fallo («si haces X, se rompe Y»)
- el comando real que regenera, si la frontera tiene salida generada; si no lo tiene, una línea que diga que no aplica

Segunda parte, de review. Redacta tú mismo un párrafo de raíz y un párrafo de zona que choquen en tres sitios: cadencia, comando de validación, y qué fichero es la fuente de algo generado. Para cada choque, di cuál gana y la frase que falta para que dejen de contradecirse.

Si haces el laboratorio sobre Grafana, el destino es `pkg/tsdb/` (backends de consulta de series temporales; hoy no tiene fichero de zona). La forma es la de este capítulo: glob de esa carpeta, enlace a la guía, skill de la tarea, y reglas de esos backends.

## Capítulo 2 · Rúbrica

Un contexto bueno se puede obedecer sin preguntar y se puede revisar como un contrato.

| Criterio | Aprueba | No aprueba |
| --- | --- | --- |
| Accionable | Dice qué hacer y con qué comando del repo | «Sigue las convenciones del equipo» |
| Acotado | El glob es la frontera | El glob es el repo entero, o no hay glob |
| Referencial | Enlaza la guía humana | Pega la guía, o la parafrasea entera |
| Modos de fallo | Nombra qué se rompe si se ignora la regla | Solo lista preferencias de estilo |
| Compatible | Especializa la raíz, o la raíz nombra la excepción | Dos frases opuestas, ninguna marcada como excepción |
| Pila | Se ve qué es regla permanente y qué es skill | El procedimiento de un solo tipo de tarea está en la raíz |

La rúbrica se aplica al fichero del laboratorio y a los tres choques de la segunda parte. Una frase que no cambia una decisión del agente no puntúa en «accionable».

## Capítulo 2 · Anti-patrones

### Pegar la guía dentro de las instrucciones

1. **Qué es** — El fichero del agente reproduce páginas de la guía de estilo «para que no tenga que buscar».
2. **Por qué importa** — Hay dos fuentes. El review usa la guía viva y el agente usa la copia. El PR cumple un contrato caducado.
3. **Best practice** — Un enlace y, si hace falta, una excepción de pocas líneas que la guía no contempla.
4. **Ejercicio mental** — ¿Cuánto de tu fichero de instrucciones seguiría siendo verdad si la guía cambiara mañana?

### Dos verdades para el mismo PR

1. **Qué es** — La raíz y la zona dan órdenes opuestas sobre cadencia, comandos o ficheros generados, y ninguna dice que es una excepción.
2. **Por qué importa** — El agente elige una, el revisor exige la otra, y el equipo acaba escribiendo la norma real en el chat. El siguiente turno no la ve.
3. **Best practice** — Cada regla de zona se lee al lado de la raíz antes de mergear el fichero de contexto. El choque se resuelve en el texto, no en el review del PR de producto.
4. **Ejercicio mental** — ¿Qué regla ganarías tú en un desempate, y dónde quedaría escrita la que pierde?

### Comandos que el repo no tiene

1. **Qué es** — El contexto dice «corre los tests» o «regenera» sin el comando que el Makefile, el gestor de paquetes o el CI ya usan.
2. **Por qué importa** — El agente rellena el hueco. El PR local queda en verde con una orden distinta a la del pipeline, y el review descubre el fallo cuando el check ya ha caído.
3. **Best practice** — Copia el comando desde la lista canónica. Si el comando no existe, el trabajo es añadirlo al sitio de comandos del repo, no inventarlo en las instrucciones.
4. **Ejercicio mental** — ¿El comando que recordarías de memoria es el mismo que corre el PR?

## Límite de este capítulo

La pila dice qué cargar. No elige la frontera: eso es el capítulo 1. Un fichero de zona perfecto sobre la frontera equivocada sigue teniendo el blast radius equivocado. Y una plantilla llena de frases que no cambian decisiones («ve con cuidado», «sigue los patrones») aprueba una lectura superficial y suspende la rúbrica.

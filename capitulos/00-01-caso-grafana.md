# Capítulos 0–1 — Framing y fronteras

Estos dos capítulos se leen sin conocer Grafana. Al terminarlos tienes que poder explicar en voz alta qué es una frontera y qué es el blast radius. Grafana aparece solo como ejemplo breve; el inventario de rutas está en [example-map.md](../example-map.md).

Cada idea sigue el mismo bloque: **qué es → por qué importa → best practice → ejemplo corto → ejercicio mental**. El ejemplo se puede saltar.

*Los hechos de Grafana salen de `main` de [novarz/grafana](https://github.com/novarz/grafana), comprobados el 2 oct 2026.*

## Capítulo 0 — De asistente a sistema agentic

**Objetivos.** Distinguir un chat con permiso de escritura de un sistema con fronteras, estado y pipeline. Nombrar en qué capa actúa el agente. Separar las reglas del ensayo de las reglas del producto.

### Un sistema agentic tiene frontera, estado y salida

1. **Qué es** — Un asistente contesta un mensaje y el resultado se juzga a ojo. Un sistema agentic mete a ese modelo dentro de piezas que ya viven en el repositorio: una frontera que acota qué puede tocar, un estado que sigue ahí cuando el chat se cierra, y un pipeline por el que sale el cambio (tests, dueños, una persona que aprueba).
2. **Por qué importa** — Sin frontera, el blast radius del turno es el repositorio entero. Sin estado durable, el agente vuelve a decidir cosas que el repo ya decidió (cómo se genera código, cómo se despliega). Sin pipeline, el review llega cuando el PR ya mezcló cadencias y revisores.
3. **Best practice** — Antes de encargar el cambio, escribe tres líneas: frontera, estado que toca, quién aprueba. Si una línea queda vacía, ese turno no es trabajo autónomo.
4. **Ejemplo (Grafana, opcional)** — El fichero raíz de instrucciones declara una puerta humana: antes de `git push`, parar y pedir aprobación. Que la tarea diga «abre un PR» no es permiso para empujar. [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md)
5. **Ejercicio mental** — En el último diff que aceptaste desde un chat, ¿dónde estaba escrita la frontera?

### La calidad vive en el repositorio

1. **Qué es** — Lo que sostiene al agente está versionado junto al código: instrucciones, comandos canónicos, hooks, checks del PR y el mapa de quién revisa cada ruta. El chat consulta esas piezas. El chat no las guarda.
2. **Por qué importa** — Un prompt bueno no llega al siguiente agente ni al siguiente revisor. Si la regla vive solo en la conversación, el turno siguiente la rompe y el review discute gustos. El blast radius de una norma no escrita es todo PR futuro.
3. **Best practice** — Toda restricción que el agente deba cumplir mañana se commitea en la capa que le toca: instrucciones, proceso humano o CI.
4. **Ejercicio mental** — Elige una norma que tu equipo «ya sabe». ¿Un agente nuevo la encontraría sin que se la cuentes?

### Cuatro capas sobre el mismo árbol

1. **Qué es** — Las mismas carpetas sirven a cuatro capas. **Código:** lo que se compila y se despliega. **Contexto:** lo que el agente lee para decidir. **Proceso humano:** cómo entra un cambio y quién lo firma. **CI y entorno:** lo que valida y lo que ejecuta. En un turno el agente escribe en una capa y las otras solo las lee, salvo que el encargo sea cambiarlas.
2. **Por qué importa** — Confundir la capa equivoca el blast radius. Retocar dueños, plantillas o un check para «que el PR pase» cambia las reglas del sistema, y el review de producto no está mirando eso.
3. **Best practice** — Clasifica cada fichero con una pregunta: ¿quién se rompe si esto cambia? La respuesta dice la capa y si este turno puede editarlo.
4. **Ejercicio mental** — Un fichero de valores por defecto de configuración. ¿Es código, estado durable o entorno, y quién se rompe si un agente lo reescribe?

### El ensayo tiene punto de restauración

1. **Qué es** — Un harness es un entorno para practicar con agentes: estado inicial conocido, una forma de volver a él, y un reparto de quién arranca procesos y quién edita. El producto es lo que ejecutan los usuarios. Fronteras, CI y ownership se aprenden en el ensayo y valen en el producto. Las reglas del ensayo (tickets de la demo, rama de reset) se quedan en el ensayo.
2. **Por qué importa** — Si el agente trata el ensayo como producción, dispara acciones de demo sobre la rama real. Si tú tratas el producto como ensayo, cuelas un atajo de demo en un PR que va a mergearse. El review no distingue las dos cosas si el fichero de instrucciones tampoco las distingue.
3. **Best practice** — En las instrucciones, marca qué reglas son del ensayo y cuáles son del sistema. Deja escrito el comando que devuelve el entorno a un estado conocido y quién puede ejecutarlo.
4. **Ejemplo (Grafana, opcional)** — [`.cursor/reset-demo.sh`](https://github.com/novarz/grafana/blob/main/.cursor/reset-demo.sh) fija `main` en el pin `origin/demo/start`. Eso es harness. La regla de PRs separados por cadencia, en el mismo repo, es del producto.
5. **Ejercicio mental** — ¿Qué comando devolvería tu repo a un estado conocido después de una demo, y quién tiene permiso de lanzarlo?

## Capítulo 0 · Laboratorio

En papel, dibuja cuatro cajas: código, contexto, proceso humano, CI y entorno. Coloca cinco artefactos de un sistema que conozcas. Uno tiene que ser ambiguo: justifica la caja con «quién se rompe». En una segunda pasada, tacha lo que un agente no vería si solo leyera el README. Lo tachado es contexto que falta (capítulo 2).

Si anclas el diagrama a Grafana, usa un solo fichero de [example-map.md](../example-map.md). Este laboratorio no es un inventario de carpetas.

## Capítulo 0 · Rúbrica

Entregable: un diagrama de una página con cuatro actores (humano, agente, CI, estado durable) y flechas que sean transiciones (lee contexto, edita, una persona aprueba, CI valida, el merge cambia estado).

| | Aprueba | No aprueba |
| --- | --- | --- |
| Lectura | Alguien que no conoce el producto dice dónde se detiene el agente y quién decide que un cambio entra | Hace falta conocer el repo para entender el dibujo |
| Flechas | Cada flecha importante nombra un tipo de artefacto (instrucciones, dueños, check, config o schema) | Las flechas dicen «colabora» o «usa IA» |
| Ancla | Como mucho un path real, si usas el caso Grafana | Una lista de carpetas en el margen |

## Capítulo 0 · Anti-patrones

### Leer las instrucciones una vez y archivarlas

1. **Qué es** — El fichero de instrucciones del agente es un contrato operativo: puertas, cadencias, comandos. Tratarlo como un manifiesto es leerlo, asentir y no volver a usarlo al abrir un PR.
2. **Por qué importa** — El agente y el review quedan sin norma común. El PR mezcla squads y el blast radius se discute en comentarios, tarde.
3. **Best practice** — En cada PR agentic, cita la regla del contrato que el cambio cumple (cadencia, puerta humana, comando). Si no hay regla que citar, el contrato está incompleto.
4. **Ejercicio mental** — ¿Qué frase del contrato citarías para frenar un push automático?

### Saltarse dueños y cadencias

1. **Qué es** — El mapa de dueños (en muchos repos, un fichero `CODEOWNERS`: quién revisa cada ruta) dice qué persona puede aceptar el cambio. La cadencia dice cuándo llega a producción. Ignorar los dos es tratar el review como un trámite.
2. **Por qué importa** — Un PR con tres dueños no lo aprueba nadie entero. Un PR que mezcla dos cadencias pide al review que firme un estado que producción no va a tener.
3. **Best practice** — Si no puedes nombrar un dueño y una cadencia, parte el cambio antes de llamar al agente.
4. **Ejercicio mental** — ¿Quién aprobaría hoy un cambio que toca a la vez la UI y el servicio que se despliega otro día?

---

## Capítulo 1 — La frontera es un contrato

**Objetivos.** Definir una frontera en la que un agente pueda trabajar un turno. Estimar el blast radius sin contar líneas. Decidir cuántos PRs hace falta y si hay algo que regenerar.

### Una frontera es un contrato

1. **Qué es** — Una frontera es un trozo del repositorio con contrato propio: lenguaje o cadena de herramientas, dueños identificables, y una forma de validarse sin arrastrar el resto. Una frontera segura para un agente cabe en un turno: se edita dentro, los tests de esa frontera responden, y un solo grupo puede revisar.
2. **Por qué importa** — Si el turno cruza el contrato, el PR junta revisores y el blast radius deja de ser el que el autor creía. El agente puede editar ambos lados con el mismo esfuerzo; el review no puede aprobarlos con el mismo criterio.
3. **Best practice** — Antes de editar, escribe la frontera en una frase: qué entra, qué queda fuera, quién revisa, qué comando la valida. Si la frase no sale, el cambio todavía no es un turno.
4. **Ejemplo (Grafana, opcional)** — Un cambio de presentación que se queda en [`public/app/features/dashboard/`](https://github.com/novarz/grafana/tree/main/public/app/features/dashboard) se valida con los tests de esa feature. Esa carpeta es la frontera del turno.
5. **Ejercicio mental** — El último cambio que describiste como «el botón y la API». ¿Son dos fronteras con dueños distintos?

### El blast radius es quién se rompe

1. **Qué es** — El blast radius es el conjunto de personas, sistemas y versiones que se rompen si el cambio va mal o si solo se despliega una parte. Se cuenta en consumidores, no en líneas. Una pantalla que solo usa tu equipo tiene radio chico. El mismo componente, si lo consumen otros equipos o gente fuera del repo, tiene radio grande.
2. **Por qué importa** — El agente optimiza el diff que ve. Quien no está en el diff —el servicio que sale la semana que viene, el paquete de terceros, el fichero que el próximo build regenera— solo entra en el review si alguien lo nombra. Un radio grande pide más de un revisor y, a menudo, más de un PR.
3. **Best practice** — En la descripción del cambio, una línea obligatoria: «si esto falla, se rompe X y no se rompe Y». Si X incluye consumidores fuera del repo, es un cambio de API.
4. **Ejemplo (Grafana, opcional)** — [`packages/grafana-ui`](https://github.com/novarz/grafana/tree/main/packages/grafana-ui) lo usa la aplicación y también plugins de terceros. Cambiar ahí una firma cambia un contrato, aunque el monorepo siga compilando.
5. **Ejercicio mental** — Cambias el redondeo de un helper de fechas en una librería compartida y los tests del producto siguen en verde. ¿Quién se entera?

### Un PR por cadencia de despliegue

1. **Qué es** — La cadencia es el ritmo al que una parte llega a producción. Dos carpetas del mismo commit pueden desplegarse en días distintos. En esa ventana, la versión nueva de un lado convive con la versión vieja del otro, así que cada lado tiene que seguir entendiendo al otro.
2. **Por qué importa** — Un PR «completo» esconde la ventana. Los tests locales corren las dos mitades juntas, que es justo el estado que producción no tiene. El agente no ve el fallo; el review firma un estado ficticio.
3. **Best practice** — Si dos partes no salen a la vez, van en PRs distintos. El primero en desplegarse no puede exigir que el segundo ya esté fuera. Cada lado tolera la versión anterior del otro.
4. **Ejemplo (Grafana, opcional)** — El contrato raíz pide PRs distintos para frontend y backend porque se despliegan con cadencias distintas. [AGENTS.md](https://github.com/novarz/grafana/blob/main/AGENTS.md)
5. **Ejercicio mental** — Tu API sale los martes y la web los jueves. El martes, ¿qué hace el cliente viejo contra la API nueva?

### Se cambia la fuente y se regenera

1. **Qué es** — Un artefacto generado es un fichero que un comando escribe a partir de una fuente. La fuente es el contrato. El fichero generado es una salida versionada. Editar la salida a mano produce un diff que el siguiente comando deshace o deja a medias.
2. **Por qué importa** — El review ve un tipo «arreglado» y lo aprueba. La siguiente generación vuelve a la fuente vieja y el arreglo desaparece, o fuente y salida divergen y el próximo agente repara el lado equivocado. El blast radius incluye todo lo que cuelga de esa cadena, a veces en más de un lenguaje.
3. **Best practice** — Si un comando documentado reescribe el fichero, el cambio va en la fuente, se regenera con ese comando, y el PR trae fuente y salida y nombra el comando.
4. **Ejemplo (Grafana, opcional)** — Los schemas de dashboard y panel viven en [`kinds/`](https://github.com/novarz/grafana/tree/main/kinds). `make gen-cue` escribe a partir de ellos el código del backend y del frontend. CUE es un lenguaje para describir schemas; Go es el lenguaje del backend y TypeScript el del frontend. Un arreglo a mano en la salida no sobrevive a la siguiente corrida.
5. **Ejercicio mental** — El diff del agente solo toca la salida. ¿Dónde está la fuente y qué comando la vuelve a escribir?

### El tamaño se cuenta en fronteras

1. **Qué es** — Para un agente, el tamaño de un cambio es el número de fronteras que cruza. Diez líneas dentro de una frontera caben en un turno. Diez líneas que cruzan dos cadenas de herramientas, dos cadencias y dos dueños son un proyecto.
2. **Por qué importa** — «Terminar la feature» en un solo diff junta radios de impacto que el review no puede firmar juntos. Partir por frontera deja cada PR con tests, dueño y cadencia propios.
3. **Best practice** — Si cuentas dos fronteras, escribe dos planes antes de editar. Uno de los dos puede ser «este turno no lo toco».
4. **Ejercicio mental** — Coge el último ticket que parecía «un rato». Cuenta fronteras, no líneas. ¿Sigue siendo un rato?

## Capítulo 1 · Laboratorio

Bug ficticio, a propósito ambiguo: «cambiar cómo se muestra el título de un panel». Escribe dos planes, sin abrir el repo.

**Solo presentación.** El título ya existe en el modelo guardado. El plan nombra una frontera, un dueño, los tests de esa frontera y un solo PR.

**Cambio de modelo.** Hace falta un campo nuevo o un significado nuevo en lo que se persiste. El plan ordena: fuente del schema, comando que regenera, consumidores, qué ficheros son salida generada, y qué va en el PR que se despliega primero.

Tercera pregunta, para los dos planes: ¿quién consume esto además del equipo que lo escribe, y qué se rompe fuera de ese equipo?

Si quieres el ancla de Grafana, reutiliza el ejemplo de [`kinds/`](https://github.com/novarz/grafana/tree/main/kinds) del principio anterior: el campo nuevo empieza en la fuente y se regenera; la presentación sola no entra ahí.

## Capítulo 1 · Rúbrica

Entregable: una matriz. Una fila por cambio. Columnas: frontera, blast radius (quién se rompe), tests mínimos, dueños, número de PRs, comando de generación o «no aplica».

Mínimo: las dos filas del laboratorio, más una fila que cruce dos cadencias.

| | Aprueba | No aprueba |
| --- | --- | --- |
| Decisión | Un agente, leyendo la fila, sabe si cabe en un turno, cuántos PRs abre y a quién le llega el review | «Tests: los necesarios», «dueños: el equipo» |
| Radio | La columna de blast radius nombra consumidores, incluidos los de fuera del repo cuando existan | La columna repite el número de líneas o de ficheros |
| Generación | La fila dice la fuente y el comando, o dice explícitamente que no hay nada generado | El diff toca una salida y la fila no lo marca |

## Capítulo 1 · Anti-patrones

### Un solo PR para dos cadencias

1. **Qué es** — El agente edita los dos lados en un turno y abre un PR «full stack» porque el diff parece una feature cerrada.
2. **Por qué importa** — Mezcla revisores, mezcla checks y crea la ventana en la que un lado nuevo convive con el otro viejo sin que nadie la haya diseñado.
3. **Best practice** — Parte por cadencia en el plan, antes de la primera edición. El PR que sale primero tiene que funcionar con el lado que todavía no ha cambiado.
4. **Ejercicio mental** — Si el backend mergea el lunes y el frontend el jueves, ¿qué comportamiento ves el martes?

### Editar la salida a mano

1. **Qué es** — El agente «arregla» un tipo en un fichero generado porque es el fichero que el compilador señaló.
2. **Por qué importa** — El diff parece correcto, pasa un review superficial y desaparece o se corrompe en la siguiente generación. El turno siguiente repara otra vez la salida.
3. **Best practice** — Ante un error en un fichero generado, el arreglo se escribe en la fuente y se regenera con el comando que ya documenta el repo.
4. **Ejercicio mental** — ¿Qué comentario o qué comando te haría soltar el fichero que tienes abierto y buscar la fuente?

## Lo que viene

El capítulo 2 apila el contexto para que el agente cargue el contrato de la frontera en la que está: raíz, fichero de la zona, procedimiento invocable, guía humana por enlace. La matriz de este capítulo es la entrada de ese laboratorio.

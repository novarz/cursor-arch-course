# Principios del curso (una página)

Se leen sin Grafana. En los capítulos, cada uno se despliega igual: **principio → por qué → best practice → ejemplo corto**.

### 1. Sistema agentic

**Qué.** Frontera (qué puede tocar) + estado que sobrevive al chat + pipeline por el que sale el cambio (tests, dueños, una persona que aprueba).

**Por qué.** Si falta una pata, el blast radius del turno es «lo que el modelo decidió en este chat».

**Práctica.** Tres líneas antes de encargar nada: frontera, estado, quién aprueba.

### 2. La norma vive en el repo

**Qué.** Instrucciones, comandos, hooks, checks y dueños están versionados. El chat los consulta.

**Por qué.** El modelo no retiene memoria entre respuestas. Un prompt no llega al siguiente agente ni al siguiente review.

**Práctica.** Toda restricción de mañana se commitea en su capa. El chat no es un almacén.

### 3. Una frontera es un contrato

**Qué.** Trozo del repo con lenguaje o cadena de herramientas propios, dueños, y validación que no arrastra al resto. Un turno de agente cabe en una frontera.

**Por qué.** Cruzar el contrato junta revisores y mezcla criterios de aprobación en un solo PR.

**Práctica.** Una frase antes de editar: qué entra, qué queda fuera, quién revisa, qué comando valida.

### 4. Blast radius

**Qué.** Quién y qué se rompe si el cambio va mal o si solo se despliega una parte. Se cuenta en consumidores y en versiones que conviven, no en líneas.

**Por qué.** El agente ve el diff. El review tiene que cubrir a quien no está en el diff.

**Práctica.** Una línea en el PR: «si esto falla, se rompe X y no se rompe Y».

### 5. Un PR por cadencia

**Qué.** Dos partes del mismo commit pueden llegar a producción en días distintos. En medio, lo nuevo convive con lo viejo.

**Por qué.** Los tests del turno corren las dos mitades juntas: el único estado que producción no tiene.

**Práctica.** Quien sale primero funciona con el lado que todavía no ha cambiado.

### 6. Se edita la fuente

**Qué.** Lo generado es la salida de un comando. La fuente es el contrato.

**Por qué.** El arreglo a mano en la salida no sobrevive a la siguiente generación, y el turno siguiente repara otra vez el lado equivocado.

**Práctica.** Cambio en la fuente, regenerar con el comando documentado, PR con las dos cosas y el comando escrito.

### 7. El tamaño son las fronteras cruzadas

**Qué.** Diez líneas en una frontera son un turno. Diez líneas en dos cadencias, dos dueños y dos cadenas de herramientas son un proyecto.

**Por qué.** «Cerrar la feature» en un diff pide al review que firme radios distintos a la vez.

**Práctica.** Dos fronteras, dos planes. Uno puede ser «este turno no».

### 8. El contexto se apila

**Qué.** Raíz (vale para cualquier PR) → zona de la frontera (`AGENTS.md` de la carpeta, o regla con glob) → skill (procedimiento que se abre solo si la tarea lo pide) → guía humana por enlace.

**Por qué.** Un solo fichero o se desfasa o impone a todo el repo las normas de un equipo. Cargar de menos pisa el contrato de la frontera.

**Práctica.** Una línea por turno: raíz + zona + skill si aplica + enlaces. Lo que no aplica, se dice.

### 9. Una regla, un sitio

**Qué.** La guía de estilo vive en la documentación humana. La zona enlaza. La raíz no se anula en silencio: la zona especializa, o la raíz nombra la excepción.

**Por qué.** Dos copias, o dos órdenes opuestas, hacen que agente y review firmen contratos distintos.

**Práctica.** Al escribir una regla de zona, buscar la frase hermana en la raíz y en la guía. Si chocan, resolverlo en el texto antes del PR de producto.

### 10. Accionable significa comando y fallo

**Qué.** El contexto dice el comando canónico del repo y qué se rompe si se ignora.

**Por qué.** El hueco lo rellena el agente con una orden que el CI no corre.

**Práctica.** Cada fichero de zona: un comando copiado de la lista real y dos frases «si haces X, se rompe Y».

### 11. Puerta humana antes de publicar

**Qué.** Empujar, abrir el PR o disparar un efecto difícil de deshacer es un borde del sistema, no un detalle del prompt.

**Por qué.** «Abre un PR» en la tarea expresa intención. La aprobación es otra instrucción, explícita. La frase no bloquea el comando.

**Práctica.** La raíz nombra la puerta y el resumen que hay que enseñar. El mecanismo que hace esperar —hook, lista de aprobación, o la persona que lanza el push— es el capítulo 3.

### 12. Cada fichero tiene una capa

**Qué.** Código, contexto del agente, proceso humano, CI y entorno. Un turno escribe en una.

**Por qué.** Cambiar un check o los dueños para que el PR pase cambia las reglas del sistema, y el review de producto no mira eso.

**Práctica.** Quién se rompe si este fichero cambia. Si la respuesta no es el producto de este turno, no se edita.

### 13. Cada necesidad tiene una pieza

**Qué.** Regla: verdad en cada edición de un ámbito. Skill: procedimiento con disparador, cargado por capas. Subagente: otro bucle con ventana propia, para aislar contexto o ir en paralelo. Hook o check de CI: se ejecuta aunque el modelo se olvide. Puerta humana: lo irreversible o lo que sale del repo.

**Por qué.** El procedimiento dentro de la regla se carga siempre. La frase «nunca hagas push» no impide el comando. Un subagente para un checklist parte el contexto sin ganar nada.

**Práctica.** Cinco preguntas, en este orden: ¿tiene que ocurrir aunque el modelo se olvide? ¿es irreversible o sale fuera? ¿hace falta otra ventana o ir en paralelo? ¿es un cómo con disparador? ¿es verdad en cada edición de este ámbito?

### 14. El texto no es un candado

**Qué.** Reglas y skills entran en el prompt. Un hook (programa enganchado a un momento del bucle) o un check de CI se ejecuta. Los modos de ejecución deciden cuándo una llamada a terminal o MCP pide aprobación. Las frases de Auto-review orientan al clasificador.

**Por qué.** Una política escrita y un push en la lista de comandos permitidos conviven. El review cree que hay puerta y el comando sale solo. Un agente de nube no usa esos modos locales.

**Práctica.** Lo que debe cumplirse siempre vive en un hook o en CI. Lo irreversible, además, espera a una persona. En la nube, la puerta está en el PR, en el hook del repo o en la rama.

### 15. Se compone de lo simple a lo compuesto

**Qué.** La topología por defecto es un agente y sus reglas. Se añade una skill cuando el procedimiento se repite. Se añaden subagentes cuando el padre no puede prever las subtareas o el ruido no cabe en su ventana. El hook y la persona van en el borde.

**Por qué.** Cada pieza extra gasta contexto y crea otro sitio donde dos instrucciones pueden chocar.

**Práctica.** Dibuja primero una caja. La segunda caja nombra el fallo repetido que la justifica.

---

**En voz alta, sin nombrar un producto.** Una frontera es un contrato: cadena de herramientas, dueños y validación propia; un turno de agente cabe en una. El blast radius es quién se rompe si el cambio va mal o llega solo a medias. El contexto se apila: contrato del repo, la zona de la frontera (carpeta o glob), un procedimiento solo si la tarea lo pide, y la guía humana por un enlace. Para cada necesidad, una pieza: regla si es verdad en cada edición, skill si es un procedimiento con disparador, subagente si hace falta otra ventana, hook o CI si tiene que ocurrir igual, persona si el efecto sale del repo.

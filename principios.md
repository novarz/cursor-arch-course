# Principios del curso (una página)

Se leen sin Grafana. En los capítulos, cada uno se despliega igual: **principio → por qué → best practice → ejemplo corto**.

### 1. Sistema agentic

**Qué.** Frontera (qué puede tocar) + estado que sobrevive al chat + pipeline por el que sale el cambio (tests, dueños, una persona que aprueba).

**Por qué.** Si falta una pata, el blast radius del turno es «lo que el modelo decidió en este chat».

**Práctica.** Tres líneas antes de encargar nada: frontera, estado, quién aprueba.

### 2. La norma vive en el repo

**Qué.** Instrucciones, comandos, hooks, checks y dueños están versionados. El chat los consulta.

**Por qué.** Un prompt no llega al siguiente agente ni al siguiente review.

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

**Qué.** Raíz (vale para cualquier PR) → zona activada por un glob (patrón de rutas) → skill (procedimiento que se abre solo si la tarea lo pide) → guía humana por enlace.

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

**Por qué.** «Abre un PR» en la tarea expresa intención. La aprobación es otra instrucción, explícita.

**Práctica.** El agente se detiene, enseña el resumen y espera. La puerta está escrita en la raíz.

### 12. El ensayo no es el producto

**Qué.** Un harness tiene estado inicial conocido, forma de volver a él, y un reparto de quién arranca procesos. Esas reglas no viajan al producto. Fronteras, CI y ownership sí.

**Por qué.** Un atajo de demo en la rama real, o una regla de producto tratada como ensayo, cambian el blast radius sin que el review lo vea.

**Práctica.** En las instrucciones, marcar qué es ensayo y qué es sistema. El punto de restauración es un pin con nombre y un comando. Mover el pin lo hace una persona.

### 13. Un turno tiene cuatro papeles

**Qué.** Agente generalista (edita dentro de la frontera), skill (procedimiento con disparador), humano (puertas y procesos que no arranca el agente), harness (vuelve a un pin conocido).

**Por qué.** Un solo agente que hace de los cuatro elige el procedimiento, publica y deja el repo sin forma de volver.

**Práctica.** Una línea antes del turno: quién edita, qué skill si aplica, qué espera a la persona, cuál es el pin.

### 14. Regla, skill o persona

**Qué.** La regla vale en cada edición de su ámbito. La skill se abre cuando la tarea coincide con el disparador. La persona entra si el paso publica, mergea, force-pushea o dispara otro sistema.

**Por qué.** Meter el procedimiento en la raíz lo carga siempre. Meter la puerta dentro de la skill hace que un disparador publique. Pedir aprobación para cada test esconde la puerta de verdad.

**Práctica.** Tres preguntas: ¿vale en cada edición? ¿es un cómo con disparador? ¿publica, mergea o dispara otro sistema? Regla, skill, persona.

### 15. Una skill llama; no copia

**Qué.** La skill especializada manda a leer la general y solo escribe lo suyo. La general dice en qué caso hace falta la especializada.

**Por qué.** Dos copias del mismo procedimiento: el review exige una y el agente obedece la otra.

**Práctica.** Si la skill nueva repite más de un paso de otra, ese paso es un enlace.

---

**En voz alta, sin nombrar un producto.** Una frontera es un contrato: cadena de herramientas, dueños y validación propia; un turno de agente cabe en una. El blast radius es quién se rompe si el cambio va mal o llega solo a medias. El contexto se apila: contrato del repo, reglas de la zona cuando el glob coincide, un procedimiento solo si la tarea lo pide, y la guía humana por un enlace. Una skill se abre cuando la tarea coincide con su disparador; una regla vale en cada edición de su zona; publicar o un efecto difícil de deshacer espera a una persona. El restore point es el pin al que el harness vuelve, no la punta de la rama de trabajo.

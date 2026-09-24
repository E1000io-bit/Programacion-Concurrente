# Clase 3 · Programación Concurrente y Paralela

> Resumen armado a partir de la transcripción automática de las grabaciones. Algunos nombres propios o términos pueden no ser exactos.

## Teórico

### Administrativo

- **Grupos para los TP**: de 4 o 5 integrantes (3 o 6 son excepciones). Hay **dos TP** en el semestre. Hay que elegir un **nombre de grupo** y mandarlo por mail al profe Mauri, con copia al profe de teoría.
- **Defensa del TP**: son coloquios de 15 a 20 minutos por grupo, con 2 o 3 preguntas por persona. Cada uno tiene que conocer el TP completo, porque el TP está pensado para que trabajen **todos** (el profe advierte sobre los "mochilas").
- **Parciales**: dos fechas, aproximadamente entre las clases 8 y 9 y entre las 14 y 15 (son 16 clases). Hay un **recuperatorio** la semana siguiente al segundo.
- **Último viernes de cada mes**: es una instancia extra de mutuo acuerdo (no está en el reglamento) para rendir lo que falte, recuperar o dar el coloquio. Queda la nota de la última vez que se rinde. No hay viernes extra el mes en que hay turno de examen.
- **IDE y versión de Java**: libres. JDK 10, 11 o 12 alcanza; no se usan features de la última versión.
- ⚠️ **No usar librerías externas que "resuelvan" la concurrencia**: la idea es aprender a manejar las secciones críticas a mano.
- Para dudas: hay clase de consulta los martes o se puede escribir por mail. El profe insiste en no acumular dudas sobre los conceptos básicos.

### Repaso: interleaving y por qué nos abstraemos

- **Interleaving**: los hilos intercalan sus instrucciones atómicas de formas impredecibles. Con 2 hilos de 2 instrucciones ya hay varias combinaciones posibles; con 14 hilos es imposible seguirlo.
- 🎓 **Pregunta típica de coloquio**: *¿qué hace la materia frente a este problema?* Se **abstrae**: sube un nivel, mira **estados y cambios de estado** en vez de líneas de código y **modela** el sistema.
- **Concurrencia**: según la RAE, "acaecimiento o concurso de varios sucesos en un mismo tiempo". En la práctica son ejecuciones que **comparten recursos** y parecen simultáneas, aunque internamente se intercalen.

### Ciclo de vida de un hilo

- La analogía del profe es un **joystick**: `new Thread()` no crea un hilo físico, crea el **control** de un hilo. Los hilos físicos están en el procesador (núcleos o hilos de hardware) y se van turnando.
- **NEW**: recién creado.
- **`start()` → READY-TO-RUN**: queda *listo para ejecutarse*, pero todavía no se ejecuta. **Depende del scheduler.**
- **RUNNING**: el scheduler le dio el procesador. El hilo alterna entre *ready* y *running*. Que dos hilos estén "listos" no significa que se estén ejecutando los dos.
- Estados de **inactividad**: *waiting* o *blocked* (esperando un recurso) y *sleeping*.
- 🎓 **Pregunta segura de coloquio**: **¿qué diferencia hay entre un hilo dormido y uno bloqueado esperando un recurso?**
  - **Dormido** (`sleep(ms)`): se duerme solo por un tiempo y **se despierta solo**, sin depender de nadie, como una alarma del celular.
  - **Bloqueado**: **depende de que otro** libere el recurso o lo despierte, como cuando uno espera que le traigan el auto.
- Conceptos a tener claros (vistos la clase anterior): **indeterminismo**, **deadlock**, **livelock** y **starvation** (inanición).

### Beneficio de la concurrencia y cuántos hilos crear

- Mientras un hilo espera memoria, disco o un sensor, **otro usa el procesador**. Así el procesador pasa el **menor tiempo ocioso posible**.
- ¿Cuántos hilos conviene crear? Con más hilos que núcleos se aprovechan los "huecos" de espera, pero **hay un límite**: si hay demasiados, se pierde más tiempo en **cambios de contexto** que procesando. El número óptimo depende del hardware.

### Paralelismo vs. concurrencia

- **Paralelismo**: muchos núcleos avanzan al mismo tiempo **sin compartir nada**, como al partir una suma de matrices en pedazos independientes. Depende de la naturaleza del problema.
- **Concurrencia**: hay un **recurso compartido**. Uno lo toma, lo usa y lo devuelve, y recién después otro puede tomarlo.

### Sección crítica y condición de carrera

- **Sección crítica**: parte del flujo del programa que **debe ser ejecutada por un solo hilo a la vez**. Es uno de los conceptos más importantes de la materia.
  - Tiene que ser **lo más chica posible pero bien protegida**. Si es grande, el programa concurrente se vuelve secuencial.
  - Un programa puede tener muchas secciones críticas (el proyecto final va a tener varias).
- **Race condition** (condición de carrera): **dos o más procesos o hilos quieren operar sobre un recurso compartido al mismo tiempo**, con más de una unidad de procesamiento.
  - Ejemplo del estacionamiento con **dos entradas** y un cartel de lugares libres: si entran dos autos a la vez, el contador puede marcar 9 en vez de 8.
  - Ejemplo del banco: sacar del cajero y transferir **al mismo tiempo** para retirar el doble.
  - La sección crítica es el momento de "consultar saldo → descontar → escribir". Tiene que hacerse **de a uno**.
- **En sistemas distribuidos** (varios pods o nodos, Kubernetes) no alcanza con resolverlo en un solo proceso: hay que **centralizar**, en general en la **base de datos**, configurando que esas transacciones se bloqueen. La concurrencia pesa sobre todo en **fintech**, y es una pregunta clásica en entrevistas laborales.

### Ejemplo: red de Petri del TP 2024

- Modelaba un sistema con **dos procesadores** que procesan tareas, cada uno con su buffer, y **recursos compartidos**. Tenía unos **12 hilos** que debían funcionar bien entre sí.
- El TP final va a ser algo parecido. La materia trabaja **del lado abstracto** (modelos), no sobre el hardware.
- Las redes de Petri sirven para modelar casi cualquier cosa: procesos industriales, ingeniería química, semáforos de tránsito, etc.

### Tema nuevo: sistemas reactivos y dirigidos por eventos

- **Sistema reactivo**: funciona a la espera de que algo pase y **reacciona con el entorno**. No es "entrada → cálculo → salida" como una calculadora.
- De ahí salen las **arquitecturas dirigidas por eventos**, muy usadas en backends distribuidos (Kafka, SQS, SNS, etc.):
  - Lo que debe ser **síncrono** se hace con llamadas que esperan respuesta. Por ejemplo, **descontar plata de una billetera**.
  - Lo **asíncrono** se publica como **evento** y lo procesan consumidores. Por ejemplo, estadísticas o actualizar un panel.

### Lenguajes formales (base para modelar)

La idea: **cada evento del sistema es un símbolo** y **una secuencia válida de eventos es una palabra**. Por ejemplo: tocan el timbre (c), suena la chicharra (a), se saca la foto (s), se abre la puerta (a).

- **Símbolo**: entidad primitiva que no se define; la elige uno para su sistema. Pueden ser letras, dígitos o incluso algo como `T14`, si en el sistema tiene sentido como una unidad.
- **Alfabeto o vocabulario (Σ)**: **conjunto** de símbolos.
- **Cadena o palabra**: **secuencia** de símbolos del alfabeto. ⚠️ La diferencia sutil es que en un conjunto no importa el orden y en una secuencia sí.
- **Cadena vacía (λ)**: longitud 0.
- **Longitud**: cantidad de símbolos de la cadena.
- **Concatenación**: pegar dos cadenas. La cadena vacía es el **elemento neutro** (como el 0 en la suma).
- 🎓 **Universo del discurso (Σ\*)**: **todas** las combinaciones posibles de los símbolos, en cualquier orden y longitud. Es la *clausura* o *cierre* e incluye λ; la *clausura positiva* (Σ⁺) no incluye λ.
- **Lenguaje**: **subconjunto** del universo del discurso, formado por las palabras que nos interesan o que son válidas.

### Gramática y autómata

- **Gramática**: ente formal que **especifica el conjunto de cadenas que forman un lenguaje**. Se define como una cuádrupla:
  - Conjunto de símbolos **terminales** (≈ eventos).
  - Conjunto de símbolos **no terminales** (≈ estados).
  - **Símbolo inicial** (estado inicial).
  - **Reglas de producción**: "estoy en este estado, pasa esto y voy a aquel".
- **Autómata**: sistema que **recibe entradas y produce salidas**. Se define con:
  - Conjunto de **entradas**, conjunto de **salidas** y conjunto de **estados**.
  - Función **f** (de transición): estado + entrada → **próximo estado**.
  - Función **g** (de salida): estado + entrada → **salida**.
  - Estado y salida son cosas distintas. Por ejemplo: en "esperando timbre", si tocan el timbre, el próximo estado es "sonando campana" y la salida es la señal que activa el timbre.
- **Relación clave**: la **gramática genera un lenguaje** y **el autómata reconoce (acepta) ese lenguaje**. Hay una relación uno a uno, así que se puede pasar de uno al otro.
- Ejemplo de la **máquina expendedora** con estados Q1, Q2 y Q3: con la entrada `a` (plata que no alcanza) la salida es 0 y la máquina queda en el mismo estado; con `b` (un billete) la salida es 1. Se arman **tablas de f y de g**.
- **Estado final**: se dibuja con **doble círculo**. Una palabra es aceptada si termina en un estado final.
- Ejemplos para practicar: el lenguaje `x · c^(3m)` con x ∈ {a, b}, cantidad de b par y m ≥ 0; y `a^(2n) · b^(2k+1)`.
- **JFLAP** (herramienta universitaria gratuita): se dibujan estados y transiciones, se prueba el autómata con palabras y **genera la gramática** automáticamente.

### ¿Para qué sirve todo esto en concurrencia?

- Con 14 hilos no se puede seguir la ejecución línea por línea ni "debuggear" el interleaving.
- En cambio, se ve el sistema **como un autómata**: recibe eventos y **cambia de estado**. Cada hilo hace lo suyo y **se validan los cambios de estado**.
- Se guarda un **log** con la secuencia de estados. Después se analiza como una cinta: si la palabra aceptada es `1-2-3-8`, se buscan esas secuencias. Si aparece un orden invertido (por ejemplo `8` antes que `3`), **algo se rompió**.
- Así se puede **prescindir del interleaving**, siempre que en las secciones críticas y condiciones de carrera no se violen los invariantes.
- 🎓 Posible pregunta de coloquio: **¿por qué vemos autómatas en concurrente?** Porque interpretamos el sistema como una sucesión de cambios de estado disparados por eventos. En la red de Petri, **los símbolos del alfabeto son las transiciones**.

---

## Práctico

> Según el profe, es **la clase más importante de la materia**: sección crítica, cómo proteger un recurso y las primitivas de Java para hacerlo (`synchronized` y `Lock`). Los semáforos se ven más adelante.

### Repaso del cuestionario de la clase anterior

- **`start()` vs `run()`**: `start()` crea un hilo nuevo y lo pone a correr. Llamar a `run()` directamente ejecuta el código en el **mismo** hilo que lo llama, sin concurrencia.
- **Dos formas de crear un hilo**: extender `Thread` o implementar `Runnable`. La ventaja de `Runnable` es que la clase puede seguir extendiendo otra clase (Java no tiene herencia múltiple).
- **¿Los atributos `static` son thread-safe?** No. **Nada es thread-safe salvo que la documentación de Java lo diga explícitamente.**
- **¿Y los `final`?** El concepto de thread-safe no aplica: una variable que nunca se modifica no puede corromperse. Los problemas aparecen con **variables compartidas que se modifican**.
- **Estado inicial de un hilo** (antes de `start()`): `NEW`. "Pausado" no es un estado válido.
- **`setPriority()`**: solo le **sugiere** una prioridad al scheduler; la JVM decide. En la materia no conviene intentar resolver cosas con prioridades.
- **Algoritmo de Lamport simplificado**: la variable `inCritical` funciona como contador de procesos que quieren entrar a la sección crítica.

### Motivación: recursos compartidos

- En entornos multihilo y multiprocesador hay concurrencia **y** paralelismo al mismo tiempo, y muchos hilos quieren acceder al mismo recurso.
- **Solo lectura**: no hay riesgo de corrupción, aunque sí de saturar CPU o memoria.
- **Lectura y escritura**: el acceso compartido puede generar **inconsistencias**. El ejemplo del profe es una billetera virtual: rendimientos, tarjeta, suscripciones y transferencias usan la misma cuenta a la vez. Los fraudes típicos consisten en lanzar muchas transferencias simultáneas para sacar más plata de la disponible. Otros ejemplos: vender stock que ya no hay o reservar dos veces el mismo asiento de avión.
- Regla de la materia: **"no debería pasar" no existe**. Hay que *asegurar* con un mecanismo que no pase. Si falla 1 vez en 100.000 corridas, la solución no sirve.

### Sección crítica

- Es una **parte del programa que protege un recurso compartido**, de modo que **un solo hilo por vez** puede ejecutarla.
- Se entra a través de un **mecanismo de sincronización**. Sin mecanismo no hay sección crítica, aunque el código se parezca.
- Los hilos esperan su turno y **la JVM decide quién entra** (el orden no está garantizado).
- **Tiene que ser lo más chica posible**: sincronizar implica secuencializar y perder performance. Adentro va solo el acceso al recurso compartido; el resto del trabajo va afuera.

### `synchronized` en métodos

- Un método `synchronized` genera una sección crítica **sobre la instancia** del objeto.
- ⚠️ **Todos los métodos `synchronized` de una misma instancia comparten UNA sola sección crítica**, no una por método. Si hay un hilo en `carComeIn()`, ningún otro puede estar en `carGoOut()` (ni en ningún otro método sincronizado de esa instancia).
- Los **métodos `static synchronized`** comparten otra sección crítica, la **de la clase**. Por eso un método estático sincronizado y uno de instancia sincronizado son **dos** secciones críticas distintas.
- En resumen: una clase puede tener **N secciones críticas de instancia (una por objeto) + 1 de clase**.
- Un método **sin** `synchronized` no participa de ninguna sección crítica.

### Ejemplo 1: playa de estacionamiento

- `main` crea un `ParkingCash`, un `ParkingStats` y `2 × nº de procesadores` hilos `Sensor` (24 en la máquina del profe), todos con **la misma instancia** de `ParkingStats`.
- Cada sensor repite 10 veces: entran 2 autos y 1 moto, salen 1 moto y 2 autos (con `sleep` en el medio). Resultado esperado: **0 autos, 0 motos y cash = 1440** (2 × 3 × 24 × 10).
- **Recursos compartidos**: `numberCars`, `numberMotorcycles` y `cash`.
- **Sin sincronizar**: los resultados son incorrectos y cambian en cada corrida (−3 autos, 5 motos, cash 1408...). Es el **interleaving**: dos hilos leen el mismo valor viejo, escriben lo mismo y se pierde una escritura (condición de carrera).
- **Todos los métodos `synchronized`**: funciona, pero no es óptimo, porque autos y motos quedan en **una sola barrera** aunque sean independientes. La analogía del profe: dos puertas del estadio que llevan a tribunas distintas y se habilita una sola.
- **Bloques sincronizados con dos llaves** (`controlCars` y `controlMotorcycles`, dos `Object`): dos secciones críticas independientes, así puede entrar un auto y una moto a la vez. ✅ 0 / 0 / 1440.
- Combinar métodos `synchronized` (llave `this`) con un bloque sobre otra llave también funciona, porque son llaves distintas.

```java
private final Object controlCars = new Object();
private final Object controlMotorcycles = new Object();

public void carComeIn() {
    synchronized (controlCars) { numberCars++; }
}
public void carGoOut() {
    synchronized (controlCars) { numberCars--; }
    cash.vehiclePay();          // fuera de la sección crítica
}
```

- **Trampa**: si `cash` se protege dentro del bloque de autos y *también* dentro del de motos, **sigue rompiéndose**, porque son dos llaves distintas y un auto y una moto pueden pagar al mismo tiempo.
- **Responsabilidades**: cada clase protege **sus propias** variables. `ParkingStats` protege autos y motos; `ParkingCash` protege `cash` (con `synchronized` en `vehiclePay()`). No hay que sincronizar "por las dudas" desde afuera.
- Anidar secciones críticas no está mal en sí, pero es redundante si ya se garantiza un solo hilo, y **puede llevar a deadlock** (un hilo tiene A y espera B mientras otro tiene B y espera A).
- `AtomicInteger`, `ConcurrentHashMap`, etc. son thread-safe y se usan en el trabajo real, pero en la materia la idea es **entender por qué se rompe y cómo arreglarlo a mano**.

### Bloques `synchronized` y qué usar como llave

- `synchronized (obj) { ... }`: la sección crítica es **el bloque**, y hay **una sección crítica por cada llave**. Mientras se use la misma llave, es la misma sección crítica.
- Lo más común es `synchronized (this)`, equivalente al método sincronizado. Otro objeto se usa cuando hay **recursos independientes**.

### Ejemplo 2: cine

- Dos ventanillas (hilos `TicketOffice`) venden y devuelven entradas para dos salas de un único `Cinema`.
- `Cinema` usa `controlCinema1` y `controlCinema2` para proteger las vacantes de cada sala. Resultado correcto: **5 y 6** vacantes.
- Al quitar la sincronización **parece** funcionar (hay poco interleaving con solo 2 hilos). Con 1000 repeticiones se rompe (3 y 4). Con los bloques restaurados, nunca se rompe.

### ⚠️ No usar wrappers (`Integer`, etc.) como llave

- Todo objeto hereda de `Object` y podría ser llave, pero **un `Integer` es inmutable**: al modificarlo (`vacancies--`) la variable apunta a **otro objeto**. Es como si cambiaran la cerradura con alguien adentro: entra otro hilo con la llave "nueva" y **deja de haber sección crítica**.
- Lo mismo pasa si se reasigna la llave con `new`.
- En la prueba del profe con 100.000 corridas, sincronizar sobre un `Integer` falló 9 veces y sobre un elemento de un `int[]` falló 1 vez. **Una sola falla alcanza para descartar la solución.**
- **Buena práctica**: `private final Object control = new Object();`, un objeto exclusivo para cada sección crítica. Es barato crearlo.

### `wait()`, `notify()` y `notifyAll()`

- Se usan **dentro de un bloque o método sincronizado** para que los hilos se comuniquen. Están definidos en `Object`, así que cualquier clase los tiene.
- **`wait()`**: si la condición que necesita el hilo no se cumple, este **libera la sección crítica** y se duerme (pasa al *wait set*). El que llama a `wait()` se duerme a sí mismo.
- **`notify()`**: despierta a **un hilo arbitrario** del wait set y lo pasa a *listo*.
- **`notifyAll()`**: pasa a *listo* a **todos**. Igual entran de a uno, porque siguen dentro de la sección crítica, y no hace falta volver a notificarlos.
- Al salir de un bloque sincronizado, el hilo **no** despierta a nadie: la JVM es la que deja pasar al siguiente.
- El profe los usa poco: "a veces complican más de lo que ayudan".

### Locks (`java.util.concurrent.locks`)

- `Lock` es una **interfaz**; la implementación típica es `ReentrantLock`. *Reentrante* significa que el hilo que ya tiene el lock puede volver a tomarlo.
- Son **más flexibles** que `synchronized`:
  - La sección crítica puede estar **distribuida** en el código o en distintas clases.
  - **`tryLock()`**: si está ocupado, el hilo hace otra cosa en vez de esperar. También existe en versión **con timeout**.
  - **Fairness**: `new ReentrantLock(true)` hace que la cola de espera sea **FIFO**. `synchronized` no ofrece esto.
  - **`ReadWriteLock`**: permite muchos lectores simultáneos y un solo escritor, que tiene prioridad al llegar. Sirve para cargas intensivas en lectura.
- ⚠️ **Siempre liberar en `finally`**. Si salta una excepción no controlada antes del `unlock()`, el lock queda tomado para siempre y el programa se bloquea. Se pone el `try` aunque no haya nada que capturar.

```java
lock.lock();
try {
    // sección crítica
} finally {
    lock.unlock();
}
```

### Ejemplo 3: cola de impresión (`PrintQueue`)

- 10 `Job` comparten una `PrintQueue`. `printJob()` hace `lock`, imprime la parte 1, `unlock`, y después `lock`, imprime la parte 2, `unlock`.
- **Con `fair = false`**: el hilo que acaba de hacer `unlock` y vuelve a hacer `lock` **les gana a los que ya estaban esperando**, porque sigue en *running* mientras los demás están *blocked*. Resultado: hilo 0 parte 1, hilo 0 parte 2, hilo 1 parte 1, hilo 1 parte 2...
- Un `sleep(1)` entre las dos partes **fuerza un cambio de contexto** y el orden cambia por completo: todos imprimen la parte 1 y después todos la parte 2.
- **Con `fair = true`**: se respeta el orden de llegada (FIFO) aunque no esté el `sleep`.
- Moraleja: **no confiar en el orden** de la JVM. El programa tiene que ser correcto sea cual sea el orden.

### ¿`synchronized` o `Lock`?

- **Keep it simple**: usar la solución más sencilla que resuelva el problema. No hay una receta que sirva para todo.
- `synchronized`: para secciones críticas simples y localizadas.
- `Lock`: cuando hace falta `tryLock`, timeout, fairness, secciones críticas repartidas o muchos recursos (por ejemplo, diez secciones críticas con diez objetos de control ya se vuelve difícil de manejar).
- Con locks sobre varios recursos, **cuidar el orden en que se toman** para evitar deadlocks.

### Administrativo

- Hay que armar **grupos de 4 o 5 integrantes** (máximo 6) para el trabajo práctico. Un integrante manda por mail el nombre del grupo y los integrantes. Debería quedar cerrado la semana siguiente.
- Bibliografía de los ejercicios: **Java 9 Concurrency Cookbook** (también hay algunos del de Java 7).
- Recomendación del profe: **correr los ejemplos uno mismo**, sacando y agregando `synchronized`, `sleep`, etc., para ver cómo se rompen.

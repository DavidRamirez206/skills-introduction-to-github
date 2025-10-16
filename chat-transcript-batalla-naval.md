# Transcript — Batalla Naval: ThreadPool, Semaphore, Server/Client (Git-friendly Markdown)

> Export generated: 2025-10-16 04:55:48 (America/Bogota)
> Repo-ready: UTF-8, LF line endings, no trailing spaces.

---

## Table of Contents
- [Contexto](#contexto)
- [Mensajes](#mensajes)
  - [Usuario → Asistente](#usuario--asistente)
- [Snippets de Código](#snippets-de-código)
- [Puntos Clave / TL;DR](#puntos-clave--tldr)
- [Glosario](#glosario)
- [Créditos](#créditos)

---

## Contexto

Este archivo reúne el hilo de conversación donde se explica, paso a paso y con analogías, cómo usar **ThreadPool** y **Semaphore** en un servidor de **Batalla Naval** (2 clientes), junto con dudas frecuentes:
- ¿Qué es `ExecutorService`? ¿Viene en Java?
- ¿Qué ocurre si el pool tiene 1 solo hilo?
- ¿Se bloquea la segunda conexión?
- ¿Cuándo usar `Semaphore` y `ThreadPool` juntos?
- ¿Para qué sirve el `Semaphore` en la **lógica de turnos** y (opcionalmente) en **acceso a base de datos**?

## Mensajes

### Usuario → Asistente

**Usuario:** *Hey, debo terminar una implementación...*  
- Pide explicación integral: semáforo, thread pool, PostgreSQL, lógica server/cliente, ClientHandler, DAO, pruebas, etc.

**Asistente:** Explica arquitectura base, dónde encaja `ClientHandler`, `GameService`, `GameDao`, y las rutas `gradlew :gameServer/:gameClient`. Introduce **ThreadPool** y **Semaphore** con ejemplos y propósito.

**Usuario:** *Sigo sin entender muy bien qué hace el thread pool y el clienthandler. Si puedes usar analogías...*

**Asistente:** Analogías del restaurante:  
- **ThreadPool** = grupo de camareros reutilizables.  
- **ClientHandler** = camarero atendiendo a un cliente (hilo por cliente).

**Usuario:** *En el threadpool no se destruyen nunca los hilos? Muestra la lógica...*

**Asistente:** Detalla ciclo de vida del pool: creación, `submit`, reutilización, `shutdown`, diferencias `newFixedThreadPool` vs `ThreadPoolExecutor` (keep-alive).

**Usuario:** *¿ExecutorService ya viene en Java? ¿Y si configuro 1 hilo?*

**Asistente:** Sí, `java.util.concurrent`. Con 1 hilo, ejecución **secuencial**: tareas en cola, sin concurrencia real.

**Usuario:** *Si hay un cliente A conectado y abro otro `:gameClient:run`, ¿me saldrá error?*

**Asistente:** No error de conexión; el segundo cliente **espera** en cola hasta que haya hilo disponible.

**Usuario:** *Entonces el intento de conexión estará en espera hasta que el otro cliente se desconecte?*

**Asistente:** Exacto: conexión aceptada, procesamiento diferido; atención secuencial con 1 hilo.

**Usuario:** *¿Qué es mejor, semáforo o threadpool? Diferencias.*

**Asistente:** No son excluyentes:  
- **ThreadPool** gestiona ejecución concurrente de tareas.  
- **Semaphore** limita acceso a **recursos**/secciones críticas.  
Se pueden combinar.

**Usuario:** *¿Cuándo usarlos juntos?*

**Asistente:** Ejemplo: servidor maneja muchas solicitudes (pool) pero limita acceso concurrente a **DB** con `Semaphore` (p.ej., 5 permisos).

**Usuario:** *Si el pool es 2 (dos jugadores), ¿para qué el semáforo?*

**Asistente:** Para **turnos** y **secciones críticas** del estado del juego: solo un disparo se procesa a la vez (contador=1).

**Usuario:** *¿Para qué manejar semáforo además en DB?*

**Asistente:** Limitar conexiones/operaciones concurrentes a DB; evitar sobrecarga y condiciones de carrera en escrituras.

**Usuario:** *Ah, entonces no es necesariamente importante??*

**Asistente:** Correcto: no obligatorio si la DB y transacciones manejan concurrencia y la carga es baja; útil si hay límites/concurrencia alta.


## Snippets de Código

### Pool fijo y envío de ClientHandler
```java
ExecutorService threadPool = Executors.newFixedThreadPool(10);

try (ServerSocket serverSocket = new ServerSocket(PORT)) {
    while (true) {
        Socket client = serverSocket.accept();
        threadPool.submit(new ClientHandler(client)); // en vez de new Thread(...).start()
    }
}
```

### ClientHandler básico
```java
public class ClientHandler implements Runnable {
    private final Socket client;
    public ClientHandler(Socket client) { this.client = client; }
    public void run() { /* leer comandos, procesar, responder, cerrar recursos */ }
}
```

### Turnos con Semaphore (contador=1)
```java
private final Semaphore turn = new Semaphore(1);
private volatile int currentTurn = 1; // 1 ó 2

void handleAction(int playerId, Runnable action) {
    try {
        turn.acquire();
        if (playerId == currentTurn) {
            action.run();
            currentTurn = (currentTurn == 1) ? 2 : 1;
        } else {
            // fuera de turno
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        turn.release();
    }
}
```

### Límite a la base de datos (opcional)
```java
private static final Semaphore dbSem = new Semaphore(5); // p.ej., 5 operaciones simultáneas

void persist(Runnable dbWork) {
    try {
        dbSem.acquire();
        dbWork.run();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        dbSem.release();
    }
}
```

## Puntos Clave / TL;DR

- **ThreadPool**: ejecuta tareas en paralelo con hilos reutilizables; evita crear/destruir hilos por cada cliente.
- **ClientHandler**: la tarea que atiende a un cliente; se ejecuta dentro del pool.
- **1 hilo** en el pool ⇒ atención **secuencial**; los demás clientes esperan en cola (sin error de conexión).
- **Semaphore**: controla acceso a **secciones críticas** (turnos, estado de juego) o **recursos limitados** (DB).
- En **Batalla Naval (2 jugadores)**: pool de 2 hilos + `Semaphore(1)` para **turnos** es una receta simple y segura.
- Usar semáforo para DB es **opcional**: útil con alta concurrencia o límites de conexiones; no imprescindible si la DB/ORM maneja transacciones y la carga es baja.

## Glosario

- **ExecutorService**: Interfaz de Java (`java.util.concurrent`) para ejecutar tareas asíncronas con pools de hilos.
- **ThreadPool**: Conjunto de hilos listos para ejecutar tareas; típicamente `Executors.newFixedThreadPool(n)`.
- **ClientHandler**: `Runnable` que gestiona I/O y comandos de un cliente.
- **Semaphore**: Contador de permisos; limita accesos concurrentes a una sección o recurso.
- **Sección crítica**: Bloque de código que no debe ser ejecutado simultáneamente por múltiples hilos.

## Créditos

- Conversación entre **Usuario** y **Asistente (GPT-5 Thinking)**.
- Código de ejemplo: Java 8+ (`java.util.concurrent`, `java.net`).


# Guía Completa para Examen de Concurrencia, Redes y Persistencia en Java

##  Temas Cubiertos
1. [JDBC (Conexión Java – Base de Datos Relacional)](#1-jdbc-conexión-entre-java-y-bases-de-datos)
2. [Sockets TCP](#2-sockets-tcp)
3. [Thread Pools y Sincronización](#3-thread-pools-y-sincronización)
4. [Semáforos](#4-semáforos)
5. [Serialización JSON con GSON](#5-serialización-json-con-gson)
6. [Simulacro de Examen (Ejercicios Integradores)](#6-simulacro-de-examen-ejercicios-integradores)

---

## JDBC (Conexión entre Java y Bases de Datos)

### ¿Qué es?
**JDBC (Java Database Connectivity)** permite conectar una aplicación Java con bases de datos relacionales como **PostgreSQL, MySQL u Oracle**.

### Pasos de conexión
```java
Class.forName("org.postgresql.Driver");
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/empresa", "usuario", "clave");
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM empleados");

while (rs.next()) {
    System.out.println(rs.getString("nombre"));
}

rs.close(); st.close(); conn.close();

```

### Estrategias de examen

-   Usa `PreparedStatement` para prevenir **inyección SQL**.
    
-   Cierra SIEMPRE los recursos (`close()`).
    
-   Preguntas típicas:
    
    -   Diferencias entre `Statement` y `PreparedStatement`.
        
    -   Cómo conectar Java con PostgreSQL.
        
    -   Cómo leer datos desde `ResultSet`.
        

----------

##  Sockets TCP

### ¿Qué es?

Un **socket TCP** permite la comunicación confiable entre un cliente y un servidor mediante el **protocolo TCP**.

### Servidor TCP

```java
ServerSocket server = new ServerSocket(5000);
Socket socket = server.accept();
BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

String mensaje = in.readLine();
out.println("Servidor recibió: " + mensaje);

socket.close();
server.close();

```

### Cliente TCP

```java
Socket socket = new Socket("localhost", 5000);
PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));

out.println("Hola desde cliente!");
System.out.println(in.readLine());

socket.close();

```

### Estrategias de examen

-   Cliente usa `new Socket(host, port)`.
    
-   Servidor usa `ServerSocket.accept()`.
    
-   Si hay varios clientes, usa **hilos o ThreadPool**.
    

----------

## Thread Pools y Sincronización

### ¿Qué es?

Un **Thread Pool** administra un conjunto fijo de hilos reutilizables para ejecutar múltiples tareas concurrentes.

### Ejemplo de Thread Pool

```java
ExecutorService pool = Executors.newFixedThreadPool(3);

for (int i = 0; i < 5; i++) {
    final int id = i;
    pool.execute(() -> {
        System.out.println("Tarea " + id + " ejecutada por " + Thread.currentThread().getName());
    });
}

pool.shutdown();

```

### Sincronización

Controla el acceso a recursos compartidos.

```java
public class Contador {
    private int valor = 0;

    public synchronized void incrementar() {
        valor++;
    }

    public int getValor() { return valor; }
}

```

###  Estrategias de examen

-   `synchronized` bloquea métodos u objetos.
    
-   `ExecutorService` administra tareas concurrentes.
    
-   Evita condiciones de carrera y deadlocks.
    

----------

##  Semáforos

### ¿Qué es?

Un **Semaphore** controla cuántos hilos pueden acceder a un recurso al mismo tiempo.

### Ejemplo

```java
Semaphore semaforo = new Semaphore(2);

for (int i = 0; i < 5; i++) {
    new Thread(() -> {
        try {
            semaforo.acquire();
            System.out.println(Thread.currentThread().getName() + " accede al recurso");
            Thread.sleep(1000);
            semaforo.release();
        } catch (InterruptedException e) {}
    }).start();
}

```

### Estrategias de examen

-   `acquire()` bloquea si no hay permisos.
    
-   `release()` libera un permiso.
    
-   `Semaphore(1)` → mutex (exclusión mutua).
    

----------

## Serialización JSON con GSON

### ¿Qué es?

**GSON** convierte objetos Java a JSON y viceversa.

### Ejemplo

```java
import com.google.gson.Gson;

class Persona {
    String nombre;
    int edad;
    Persona(String n, int e) { this.nombre = n; this.edad = e; }
}

public class EjemploGson {
    public static void main(String[] args) {
        Gson gson = new Gson();
        Persona p = new Persona("David", 25);

        String json = gson.toJson(p);
        System.out.println("JSON: " + json);

        Persona copia = gson.fromJson(json, Persona.class);
        System.out.println("Objeto: " + copia.nombre + " - " + copia.edad);
    }
}

```

### Estrategias de examen

-   `toJson(obj)` → convierte a JSON.
    
-   `fromJson(json, Clase.class)` → reconstruye el objeto.
    
-   Se usa mucho para **guardar configuración o enviar datos por red.**
    

----------

## Simulacro de Examen (Ejercicios Integradores)

### 🧩 Ejercicio 1 – Servidor TCP con Hilos

**Enunciado:** Implementa un servidor TCP que atienda varios clientes a la vez. Cada cliente envía un mensaje que el servidor imprime.

**Solución corta:**

```java
ExecutorService pool = Executors.newFixedThreadPool(3);
ServerSocket server = new ServerSocket(5000);

while (true) {
    Socket cliente = server.accept();
    pool.execute(() -> {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(cliente.getInputStream()))) {
            System.out.println("Mensaje: " + in.readLine());
        } catch (IOException e) {}
    });
}

```

**Conceptos usados:** Sockets TCP + ThreadPool

----------

### Ejercicio 2 – Control de acceso con Semáforo

**Enunciado:** Tres hilos simulan usuarios queriendo imprimir documentos, pero solo una impresora está disponible.

**Solución corta:**

```java
Semaphore sem = new Semaphore(1);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            sem.acquire();
            System.out.println(Thread.currentThread().getName() + " imprime...");
            Thread.sleep(1000);
            sem.release();
        } catch (InterruptedException e) {}
    }).start();
}

```

**Conceptos usados:** Semáforos, sincronización

----------

### Ejercicio 3 – Conexión JDBC y persistencia

**Enunciado:** Crea un programa que conecte a PostgreSQL y registre un nuevo cliente.

**Solución corta:**

```java
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/empresa", "user", "pass");
PreparedStatement ps = conn.prepareStatement("INSERT INTO cliente(nombre, correo) VALUES (?, ?)");
ps.setString(1, "David");
ps.setString(2, "david@mail.com");
ps.executeUpdate();
ps.close(); conn.close();

```

**Conceptos usados:** JDBC + PreparedStatement

----------

### Ejercicio 4 – Guardar datos recibidos por red en JSON

**Enunciado:** Un servidor TCP recibe objetos en formato JSON, los convierte a objeto Java y los guarda en archivo.

**Solución corta:**

```java
ServerSocket server = new ServerSocket(6000);
Socket socket = server.accept();

BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
String json = in.readLine();

Gson gson = new Gson();
Persona p = gson.fromJson(json, Persona.class);
System.out.println("Recibido: " + p.nombre);

socket.close(); server.close();

```

**Conceptos usados:** Sockets + GSON + serialización

----------

### Ejercicio 5 – Sistema concurrente con BD y Thread Pool

**Enunciado:** Crea un servidor TCP que recibe mensajes de clientes y los guarda en la base de datos usando un pool de hilos.

**Solución corta:**

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
ServerSocket server = new ServerSocket(7000);
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/logs", "user", "pass");

while (true) {
    Socket cliente = server.accept();
    pool.execute(() -> {
        try {
            BufferedReader in = new BufferedReader(new InputStreamReader(cliente.getInputStream()));
            String msg = in.readLine();

            PreparedStatement ps = conn.prepareStatement("INSERT INTO mensajes(texto) VALUES (?)");
            ps.setString(1, msg);
            ps.executeUpdate();
            ps.close();
        } catch (Exception e) { e.printStackTrace(); }
    });
}

```

**Conceptos usados:** Sockets TCP + JDBC + Thread Pool

----------

## Recomendaciones Finales

### Antes del examen

-   Memoriza los métodos clave:
    
    -   JDBC → `getConnection`, `executeQuery`, `executeUpdate`
        
    -   Sockets → `accept`, `readLine`, `write`
        
    -   GSON → `toJson`, `fromJson`
        
    -   Semáforos → `acquire`, `release`
        
    -   ThreadPool → `execute`, `shutdown`
        
-   Practica **escribir código completo sin copiar y pegar.**
    
-   Identifica **palabras clave del enunciado** (concurrencia, red, sincronización).
    

### Durante el examen

-   Siempre **cierra recursos** (try-with-resources o `close()`).
    
-   Usa **mensajes de consola** para verificar flujo de ejecución.
    
-   Si piden “control de acceso”, piensa en **semáforo**.
    
-   Si piden “múltiples tareas simultáneas”, piensa en **thread pool**.
    
 ----
# REMATE


Perfecto ✅ — Aquí tienes un **`README.md` completo, detallado y claro**, que explica **cada uno de los 5 temas** (JDBC, Sockets TCP, Thread Pools y sincronización, Semáforos, y Serialización JSON con GSON), **todo orientado a Java**, e incluyendo **estrategias prácticas para examen** (qué estudiar, cómo reconocer preguntas típicas, y qué escribir o implementar bajo presión).

----------

## 1️⃣ JDBC (Conexión entre Java y Bases de Datos)

### 🧩 ¿Qué es?
**JDBC (Java Database Connectivity)** es una API que permite a Java conectarse con **bases de datos relacionales** (MySQL, PostgreSQL, Oracle, etc.) para ejecutar consultas SQL.

### ⚙️ Pasos de una conexión JDBC
1. **Registrar el driver**

   ```java
   Class.forName("org.postgresql.Driver");

   ```

2.  **Establecer conexión**
    
    ```java
    Connection conn = DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/mi_bd",
        "usuario",
        "contraseña"
    );
    
    ```
    
3.  **Crear y ejecutar sentencia**
    
    ```java
    Statement st = conn.createStatement();
    ResultSet rs = st.executeQuery("SELECT * FROM clientes");
    
    ```
    
4.  **Procesar resultados**
    
    ```java
    while (rs.next()) {
        System.out.println(rs.getString("nombre"));
    }
    
    ```
    
5.  **Cerrar recursos**
    
    ```java
    rs.close();
    st.close();
    conn.close();
    
    ```
    

### 🧠 Estrategias de examen

-   Aprende **las clases principales**: `Connection`, `Statement`, `PreparedStatement`, `ResultSet`.
    
-   Recuerda que `PreparedStatement` se usa para evitar **inyección SQL**.
    
-   Si te piden leer o escribir datos, **muestra siempre el cierre de recursos**.
    
-   Preguntas típicas:
    
    -   “¿Qué pasos sigues para conectar Java a una BD?”
        
    -   “¿Diferencias entre Statement y PreparedStatement?”
        
    -   “Escribe un código que inserte y consulte registros usando JDBC.”
        

----------

## 2️⃣ Sockets TCP

### 🧩 ¿Qué es?

Un **socket TCP** permite comunicación **bidireccional** entre dos programas (cliente-servidor) mediante el **protocolo TCP**.

### 🖥️ Servidor TCP

```java
import java.io.*;
import java.net.*;

public class ServidorTCP {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(5000);
        System.out.println("Servidor esperando conexión...");
        Socket socket = server.accept();

        BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

        String mensaje = in.readLine();
        System.out.println("Cliente dice: " + mensaje);
        out.println("Mensaje recibido: " + mensaje);

        socket.close();
        server.close();
    }
}

```

### 💻 Cliente TCP

```java
import java.io.*;
import java.net.*;

public class ClienteTCP {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 5000);

        BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

        out.println("Hola desde el cliente!");
        System.out.println("Servidor dice: " + in.readLine());

        socket.close();
    }
}

```

### 🧠 Estrategias de examen

-   Entiende **la diferencia entre TCP (orientado a conexión)** y **UDP (sin conexión)**.
    
-   Recuerda:
    
    -   Servidor usa `ServerSocket.accept()`.
        
    -   Cliente usa `new Socket(host, port)`.
        
-   Si te piden “multicliente”, usa **threads o thread pools** para manejar cada conexión.
    

----------

## 3️⃣ Thread Pools y Sincronización

### 🧩 ¿Qué es?

Un **Thread Pool (pool de hilos)** es un conjunto de hilos pre-creados que ejecutan tareas concurrentemente.  
Se usan para evitar el **costo de crear hilos nuevos constantemente**.

### ⚙️ Ejemplo de Thread Pool

```java
import java.util.concurrent.*;

public class EjemploThreadPool {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 5; i++) {
            final int tarea = i;
            pool.execute(() -> {
                System.out.println("Ejecutando tarea " + tarea + " en " + Thread.currentThread().getName());
            });
        }

        pool.shutdown();
    }
}

```

### 🔒 Sincronización

Evita **condiciones de carrera** (cuando dos hilos acceden al mismo recurso a la vez).

#### 🔹 Bloque sincronizado

```java
public synchronized void incrementar() {
    contador++;
}

```

o:

```java
synchronized (this) {
    contador++;
}

```

#### 🔹 Bloque con `ReentrantLock`

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    contador++;
} finally {
    lock.unlock();
}

```

### 🧠 Estrategias de examen

-   Identifica problemas de **race condition** o **deadlock**.
    
-   Recuerda: `synchronized` bloquea a nivel de método u objeto.
    
-   Si ves código con `ExecutorService`, explica que **gestiona tareas con hilos reutilizables**.
    
-   Preguntas típicas:
    
    -   “¿Qué ventaja tiene un thread pool sobre crear hilos manualmente?”
        
    -   “¿Qué hace synchronized?”
        
    -   “¿Qué pasa si dos hilos modifican la misma variable sin sincronización?”
        

----------

## 4️⃣ Semáforos

### 🧩 ¿Qué es?

Un **semáforo** controla el acceso a un recurso compartido limitando el número de hilos que pueden usarlo simultáneamente.

### ⚙️ Ejemplo de semáforo

```java
import java.util.concurrent.Semaphore;

public class EjemploSemaforo {
    static Semaphore semaforo = new Semaphore(2);

    public static void main(String[] args) {
        for (int i = 1; i <= 5; i++) {
            new Thread(new Trabajador(i)).start();
        }
    }

    static class Trabajador implements Runnable {
        int id;
        Trabajador(int id) { this.id = id; }

        public void run() {
            try {
                semaforo.acquire();
                System.out.println("Trabajador " + id + " accede al recurso");
                Thread.sleep(1000);
                System.out.println("Trabajador " + id + " libera el recurso");
                semaforo.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### 🧠 Estrategias de examen

-   Recuerda los métodos:
    
    -   `acquire()` → pide permiso (puede bloquear)
        
    -   `release()` → libera permiso
        
-   Si hay `Semaphore(1)`, se comporta como **un candado (mutex)**.
    
-   Preguntas típicas:
    
    -   “¿Qué diferencia hay entre synchronized y Semaphore?”
        
    -   “¿Qué pasa si no se llama release()?”
        
    -   “Simula una impresora compartida usando semáforos.”
        

----------

## 5️⃣ Serialización JSON con GSON

### 🧩 ¿Qué es?

**GSON** (Google Serialization/Deserialization) permite **convertir objetos Java a JSON y viceversa**.

### ⚙️ Ejemplo básico

```java
import com.google.gson.Gson;

class Persona {
    String nombre;
    int edad;

    Persona(String n, int e) { this.nombre = n; this.edad = e; }
}

public class EjemploGSON {
    public static void main(String[] args) {
        Gson gson = new Gson();

        Persona p = new Persona("David", 25);
        String json = gson.toJson(p);
        System.out.println("JSON: " + json);

        Persona p2 = gson.fromJson(json, Persona.class);
        System.out.println("Objeto: " + p2.nombre + " - " + p2.edad);
    }
}

```

### 🧠 Estrategias de examen

-   Recuerda los métodos:
    
    -   `toJson(obj)` → convierte objeto a cadena JSON
        
    -   `fromJson(cadena, Clase.class)` → convierte JSON a objeto
        
-   Posibles usos en examen:
    
    -   Guardar configuración en JSON.
        
    -   Serializar objetos a archivo.
        
-   Preguntas típicas:
    
    -   “Convierte una lista de objetos a JSON usando GSON.”
        
    -   “¿Qué ventaja tiene GSON frente a la serialización binaria?”
        
    -   “Explica cómo leer un archivo JSON y convertirlo en objetos Java.”
        

----------

## 🧩 Estrategias Globales para el Examen

### 📑 Teoría

-   Aprende **definiciones y ventajas** de cada tecnología.
    
-   Memoriza las **clases principales y sus métodos clave**.
    
-   Dibuja diagramas de flujo simples (por ejemplo, cliente-servidor o sincronización de hilos).
    

### 🧪 Práctica

-   Escribe **microprogramas de ejemplo** (10-20 líneas) para cada tema.
    
-   Practica **ejecutar y depurar**: ver cuándo un hilo se bloquea, o cuándo falla una conexión JDBC.
    
-   Simula preguntas tipo examen:
    
    -   “Implementa un servidor TCP que atienda múltiples clientes con un ThreadPool.”
        
    -   “Conecta a PostgreSQL, inserta un registro y muestra todos los resultados.”
        

### ⚡ Recomendación final

> **Domina los patrones:** conexión → procesamiento → cierre.  
> (se aplica a JDBC, sockets, hilos y recursos compartidos)


# 🧠 Parcial - Programación Concurrente y Redes

## Temas evaluados:
1. Uso de **Sockets TCP**
2. Uso de **Thread Pools** y **sincronización**
3. Uso de **Semáforos**
4. Uso de **Serialización JSON con GSON**

---

## 🛰️ 1. Uso de Sockets TCP

### 🔹 Cliente TCP básico
```java
import java.io.*;
import java.net.*;

public class TCPClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 5000);

        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));

        out.println("Hola servidor!");
        System.out.println("Servidor responde: " + in.readLine());

        socket.close();
    }
}
🔹 Servidor TCP básico
java
Copiar código
import java.io.*;
import java.net.*;

public class TCPServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(5000);
        System.out.println("Servidor esperando conexión...");

        Socket clientSocket = serverSocket.accept();
        System.out.println("Cliente conectado.");

        BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);

        String mensaje = in.readLine();
        out.println("Mensaje recibido: " + mensaje);

        clientSocket.close();
        serverSocket.close();
    }
}
✅ Recordar:

El cliente inicia la conexión.

El servidor escucha en un puerto fijo.

Usar flush() o true en PrintWriter para enviar datos.

Siempre cerrar sockets después de usarlos.

⚙️ 2. Uso de Thread Pools y Sincronización
🔹 Creación de un ThreadPool
java
Copiar código
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(4);

        for (int i = 1; i <= 8; i++) {
            int taskId = i;
            pool.execute(() -> {
                System.out.println("Ejecutando tarea " + taskId + " en " + Thread.currentThread().getName());
            });
        }

        pool.shutdown();
    }
}
✅ Recordar:

newFixedThreadPool(n) limita los hilos.

shutdown() detiene la entrada de nuevas tareas.

Ideal para manejar múltiples clientes en un servidor TCP.

🧵 3. Sincronización
🔹 Uso del modificador synchronized
java
Copiar código
public class Contador {
    private int valor = 0;

    public synchronized void incrementar() {
        valor++;
    }

    public synchronized int obtener() {
        return valor;
    }
}
🔹 Bloque sincronizado
java
Copiar código
synchronized (objeto) {
    // Bloque protegido
}
✅ Recordar:

Evita condiciones de carrera.

Bloquea acceso simultáneo al mismo recurso.

Solo un hilo a la vez puede ejecutar código sincronizado sobre el mismo objeto.

🚦 4. Semáforos
🔹 Ejemplo con Semaphore
java
Copiar código
import java.util.concurrent.Semaphore;

public class EjemploSemaforo {
    private static Semaphore sem = new Semaphore(2); // Máximo 2 hilos a la vez

    public static void main(String[] args) {
        for (int i = 1; i <= 5; i++) {
            new Thread(new Tarea(i)).start();
        }
    }

    static class Tarea implements Runnable {
        private int id;
        Tarea(int id) { this.id = id; }

        public void run() {
            try {
                sem.acquire();
                System.out.println("Tarea " + id + " entrando a la sección crítica.");
                Thread.sleep(1000);
                System.out.println("Tarea " + id + " saliendo.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                sem.release();
            }
        }
    }
}
✅ Recordar:

acquire() bloquea hasta que haya permiso.

release() libera un permiso.

Controla cuántos hilos acceden simultáneamente a un recurso.

💾 5. Serialización JSON con GSON
🔹 Agregar la librería
java
Copiar código
import com.google.gson.Gson;
🔹 Serializar (Objeto → JSON)
java
Copiar código
class Persona {
    String nombre;
    int edad;

    Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }
}

public class EjemploGson {
    public static void main(String[] args) {
        Gson gson = new Gson();
        Persona p = new Persona("Ana", 25);
        String json = gson.toJson(p);
        System.out.println(json);
    }
}
Salida:

json
Copiar código
{"nombre":"Ana","edad":25}
🔹 Deserializar (JSON → Objeto)
java
Copiar código
String json = "{\"nombre\":\"Ana\",\"edad\":25}";
Persona p2 = gson.fromJson(json, Persona.class);
System.out.println(p2.nombre + " tiene " + p2.edad + " años.");
✅ Recordar:

toJson(obj) convierte a JSON.

fromJson(json, Clase.class) convierte a objeto.

Ideal para enviar/recibir objetos por sockets.

💡 Tips para el examen
Usa try-with-resources para cerrar sockets automáticamente.

Sincroniza accesos compartidos (colecciones, archivos).

Usa Thread.sleep() con precaución (maneja InterruptedException).

Recuerda que Gson no serializa campos transient.

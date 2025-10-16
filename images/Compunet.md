<details>

<summary>UDP Communication Example</summary>


# UDP Communication Example

This document contains two Java programs that demonstrate basic UDP communication. One program listens for incoming messages (`Reciver`), while the other sends a message (`Sender`). Both are implemented using Java's `DatagramSocket` and `DatagramPacket` classes.

---

## Reciver.java

This program listens for UDP messages on port 5000. When a message is received, it decodes the message and prints it along with the sender's address.

### Code:

```java
import java.io.IOException;
import java.net.DatagramSocket;
import java.net.SocketException;
import java.net.DatagramPacket;

public class Reciver {

    public static void main(String[] args){
        // Initialize a UDP socket on port 5000
        try (DatagramSocket socket = new DatagramSocket(5000)) {

            // Create a buffer to store incoming data
            byte[] buffer = new byte[1024]; 

            // Define the packet to store the incoming data
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);

            // Wait to receive data
            socket.receive(packet);

            // Decode and print the message along with the sender's address
            String msj = new String(packet.getData()).trim(); 
            System.out.println(packet.getAddress() + " " + msj);
        }
        catch(SocketException e){
            e.printStackTrace();
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

---

## Sender.java

This program sends a UDP message to a specific address (in this case, `192.168.131.23`) on port 5000. The message sent is "Hola desde el Sender."

### Code:

```java
import java.io.IOException;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;
import java.net.DatagramPacket;

public class Sender {

    public static void main(String[] args){
        try (DatagramSocket socket = new DatagramSocket()) {

            // Message to send
            String msj = "Hola desde el Sender";

            // Define the packet to store the outgoing data
            DatagramPacket packet = new DatagramPacket(
                msj.getBytes(), // Message data
                msj.length()    // Size of the message
            );

            // Connect to the recipient's IP address (192.168.131.23) on port 5000
            socket.connect(InetAddress.getByName("192.168.131.23"), 5000);
            
            // Send the message
            socket.send(packet);
        }
        catch(SocketException e){
            e.printStackTrace();
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

---

## Explanation

1. **Reciver.java** listens for UDP messages on port `5000` and prints the sender's address and the message.
2. **Sender.java** sends a predefined message "Hola desde el Sender" to a specified IP address (`192.168.131.23`) and port `5000`.

---


</details>


<details>
  <summary>02_udp</summary>


# UDP Communication Example - PeerA & PeerB

This document contains two Java programs demonstrating UDP communication between two peers. One program sends a message (`PeerA`), while the other receives it (`PeerB`). Both programs use Java's `DatagramSocket` and `DatagramPacket` classes.

---

## PeerA (Sender)

This program sends a simple message ("Hola") using UDP to a specific address (`192.168.131.200`) on port `5000`.

### Code:

```java
package ui; 
import java.net.DatagramSocket; 
import java.net.DatagramPacket; 
import java.net.InetAddress; 

public class Main{
    public static void main(String[] args) {
        try (DatagramSocket socket = new DatagramSocket()){

            String msj = "Hola"; 

            DatagramPacket packet = 
                new DatagramPacket(
                    msj.getBytes(), msj.length()); 

            socket.connect( 
                InetAddress.getByName("192.168.131.200"), 5000);
            socket.send(packet); 
        } catch(Exception e){
            e.printStackTrace();
        }
    }
}
```

---

## PeerB (Receiver)

This program listens for a UDP message on port `5000`. When a message is received, it prints the sender's address and the message.

### Code:

```java
package ui; 

import java.net.DatagramSocket;
import java.net.DatagramPacket;

public class Main{
    public static void main(String[] args) {
        
        try (DatagramSocket socket = new DatagramSocket(5000)){
            byte[] buffer = new byte[1024]; 

            DatagramPacket packet = new DatagramPacket(
                buffer, buffer.length
                ); 

            socket.receive(packet); 

            String msj = new String(packet.getData()).trim(); 

            System.out.println(
                packet.getAddress() + " " + msj);
        } catch(Exception e) {
        }
    }
}
```

---

## Explanation

1. **PeerA (Sender)**: Sends the message `"Hola"` to a specified IP address (`192.168.131.200`) on port `5000`.
2. **PeerB (Receiver)**: Listens on port `5000` and prints out the received message along with the sender's address.

---


</details>


<details>
  <summary>03_TCP</summary>



# TCP Communication Example - Client & Server

This document contains two Java programs demonstrating TCP communication between a client and a server. The client sends a message, and the server receives and prints it. The communication is done using Java's `Socket`, `ServerSocket`, `BufferedReader`, and `BufferedWriter` classes.

---

## Client (Sender)

This program connects to a server at `127.0.0.1` on port `5000` and sends a message (`"Hola desde el cliente"`).

### Code:

```java
package ui;

import util.TCPConnection;

public class Main {
    
    public static void main(String[] args) {
        TCPConnection connection = TCPConnection.getInstance(); 
        // connection.initAsClient("127.0.0.1", 5000);
        connection.sendMessage("Hola desde el cliente", "127.0.0.1", 5000);
    }
}
```

### Explanation:
- This client program connects to the server running on `127.0.0.1` at port `5000`.
- The `sendMessage` method is used to send a message to the server.

---

## TCPConnection (Client-Side)

This class handles the connection and message sending for the client.

### Code:

```java
package util;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.net.Socket;

public class TCPConnection extends Thread {

    private static TCPConnection instance;

    private Socket socket;
    private Listener listener;

    private TCPConnection(){}

    public static TCPConnection getInstance(){
        if(instance == null){
            instance = new TCPConnection();
        }
        return instance;
    }

    public void setListener(Listener listener){
        this.listener = listener;
    }

    public void initAsServer(int port){
        try{
            socket = new ServerSocket(port).accept();
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }

    public void initAsClient(String ip, int port){
        try{
            socket = new Socket(ip, port);
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }

    @Override
    public void run(){
        try{
            // Receiving message
            InputStream is = socket.getInputStream();
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader reader = new BufferedReader(isr);

            String msj = reader.readLine();
            listener.onMessage(msj);
        }catch(IOException e){
        }
    }

    public void sendMessage(String msj, String ipDest, int portDest){
        Thread sender = new Thread(
            () -> {
                try {
                    // Send message
                    Socket remoteSocket = new Socket(ipDest, portDest);
                    OutputStream os = remoteSocket.getOutputStream();
                    OutputStreamWriter osw = new OutputStreamWriter(os);
                    BufferedWriter writer = new BufferedWriter(osw);
                    writer.write(msj);
                    writer.flush();
                    writer.close();
                    remoteSocket.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        );
        sender.start();
    }

    public interface Listener{
        public void onMessage(String msj);
    }
}
```

---

## Server (Receiver)

This program listens for incoming messages on port `5000` and prints the received message.

### Code:

```java
package ui;

import util.TCPConnection;

public class Main implements TCPConnection.Listener {
    public static void main(String[] args) {
        Main m = new Main();
        TCPConnection connection = TCPConnection.getInstance();
        connection.initAsServer(5000);
        connection.setListener(m);
        connection.start();
    }

    @Override
    public void onMessage(String msj){
        System.out.println(msj);
    }
}
```

### Explanation:
- This server program listens for incoming connections on port `5000`.
- Once a message is received, it is printed to the console.

---

## TCPConnection (Server-Side)

This class handles the connection and message reception for the server.

### Code:

```java
package util;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.net.Socket;

public class TCPConnection extends Thread {

    private static TCPConnection instance;

    private Socket socket;
    private Listener listener;

    private TCPConnection(){}

    public static TCPConnection getInstance(){
        if(instance == null){
            instance = new TCPConnection();
        }
        return instance;
    }

    public void setListener(Listener listener){
        this.listener = listener;
    }

    public void initAsServer(int port){
        try{
            socket = new ServerSocket(port).accept();
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }

    public void initAsClient(String ip, int port){
        try{
            socket = new Socket(ip, port);
        }
        catch(IOException e){
            e.printStackTrace();
        }
    }

    @Override
    public void run(){
        try{
            InputStream is = socket.getInputStream();
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader reader = new BufferedReader(isr);

            String msj = reader.readLine();
            System.out.println("TCPConnection::run::msj >>> " + msj + " " + socket.getLocalAddress());
            listener.onMessage(msj);
        } catch(IOException e){
        }
    }

    public void sendMessage(String msj, String ipDest, int portDest){
        Thread sender = new Thread(
            () -> {
                try {
                    Socket remoteSocket = new Socket(ipDest, portDest);
                    OutputStream os = remoteSocket.getOutputStream();
                    OutputStreamWriter osw = new OutputStreamWriter(os);
                    BufferedWriter writer = new BufferedWriter(osw);
                    writer.write(msj);
                    writer.flush();
                    writer.close();
                    remoteSocket.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        );
        sender.start();
    }

    public interface Listener{
        public void onMessage(String msj);
    }
}
```

---

## Explanation

1. **Client (Sender)**: Sends the message `"Hola desde el cliente"` to the server running at `127.0.0.1` on port `5000`.
2. **Server (Receiver)**: Listens for messages on port `5000` and prints the received message to the console.

---


</details>


<details>
  <summary>04_semaforos</summary>



# Semaphore Example - Client, Server, and Database

This document contains Java programs demonstrating the use of semaphores to control access to a shared resource. The system involves multiple clients attempting to request a shared resource, and the server handles these requests using semaphores to ensure proper synchronization.

---

## Client

The `Client` class simulates a client that makes a request to the server. Each client is a separate thread.

### Code:

```java
package model; 

public class Client implements Runnable {

    private int id; 
    private IServer server; 

    public Client(int id, IServer server){
        this.id = id; 
        this.server = server; 
    }

    @Override
    public void run(){
        System.out.println("El cliente " + id + " esta solicitando el recurso ...");
        server.solveRequest(id); 
    }
}
```

### Explanation:
- Each `Client` object represents a client that will make a request to the server.
- The `run` method simulates a request for a resource by calling `solveRequest` on the server.

---

## DataBase

The `DataBase` class simulates a resource that clients will request. It processes each request sequentially.

### Code:

```java
package model; 

public class DataBase{

    public void request(int clientId){
        System.out.println("Client: " + clientId + " doing request ...");

        try {
            Thread.sleep(1000); 
        } catch(Exception e){
            e.printStackTrace();
        }
        System.out.println("Fin de la consulta del cliente: "+ clientId);
    }
}
```

### Explanation:
- The `DataBase` class simulates performing a resource request.
- The `request` method pauses for 1 second to simulate the time taken to process the request.

---

## IServer

The `IServer` interface defines the contract for handling client requests.

### Code:

```java
package model; 

public interface IServer {
    void solveRequest(int clientId);
}
```

### Explanation:
- This interface defines the `solveRequest` method, which is implemented by the server classes to handle client requests.

---

## Server

The `Server` class handles client requests by delegating the task to the `DataBase` class.

### Code:

```java
package model; 

public class Server implements IServer{

    private DataBase dataBase; 

    public Server(){
        dataBase = new DataBase(); 
    }

    @Override
    public void solveRequest(int clientId){
        dataBase.request(clientId);
    }
}
```

### Explanation:
- The `Server` class implements the `IServer` interface.
- It uses the `DataBase` class to process requests by calling `request`.

---

## ServerSemaphore

The `ServerSemaphore` class is a variant of the server that uses a semaphore to control access to the shared `DataBase` resource. It ensures that only one client can access the resource at a time.

### Code:

```java
package model; 
import java.util.concurrent.Semaphore;

public class ServerSemaphore implements IServer{

    private DataBase dataBase; 
    private Semaphore semaphore; 

    public ServerSemaphore(){
        dataBase = new DataBase(); 
        semaphore = new Semaphore(1);
    }

    @Override
    public void solveRequest(int clientId){
        try{
            // First operation - acquire the semaphore
            semaphore.acquire();
            // Before accessing the resource, the semaphore blocks the request 
            dataBase.request(clientId);
        } catch(InterruptedException e){
            System.out.println("El cliente " + clientId + " fue interrumpido ....");
        }
        finally{
            // Release the resource after the operation
            semaphore.release();
        }
    }
}
```

### Explanation:
- The `ServerSemaphore` class uses a semaphore with 1 permit to ensure that only one client can access the `DataBase` at a time.
- The `acquire` method blocks access to the resource, and the `release` method frees it once the request is processed.

---

## Main

The `Main` class is the entry point of the application. It creates a server with semaphores and starts multiple client threads that will make requests to the server.

### Code:

```java
package ui;

import model.Client;
import model.IServer;
import model.Server;
import model.ServerSemaphore;

public class Main {

    public static void main(String[] args) {
        IServer server = new ServerSemaphore(); 
        for(int i = 0; i < 12; i++){
            Client c = new Client(i, server); 
            new Thread(c).start();
        }
    }
}
```

### Explanation:
- The `Main` class creates a `ServerSemaphore` object and starts 12 client threads.
- Each client tries to access the server and request a resource.

---

## Explanation

1. **Client**: Each client simulates a request for a resource from the server. Multiple clients are created as threads to simulate concurrent requests.
2. **DataBase**: Simulates the resource being requested by the clients. It processes one request at a time.
3. **Server**: A basic server that processes requests from clients without synchronization.
4. **ServerSemaphore**: A server that uses a semaphore to control access to the resource, ensuring that only one client can access it at a time.
5. **Main**: The entry point that creates clients and starts the server to handle their requests.

---



</details>



<details>
  <summary>05_threadPools</summary>



# Thread Pools and Semaphore Example

This document contains Java programs that demonstrate the use of thread pools and semaphores for handling client connections in a server. The server uses a fixed thread pool to manage multiple clients, and a semaphore is used to control access to shared resources.

---

## ClientHandler

The `ClientHandler` class handles the communication with a client in a separate thread. Each client is processed by a `Runnable` task.

### Code:

```java
package ui;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.Socket;

public class ClientHandler implements Runnable {

    private Socket socket; 

    public ClientHandler(Socket socket){
        this.socket = socket; 
    }

    @Override
    public void run(){
        try {
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            ); 

            String msj =  reader.readLine();
            System.out.println("El cliente: " + socket.getInetAddress() 
                + " por el puerto: " + socket.getPort() + " " + msj);
        } catch (Exception e) {
            // TODO: handle exception
        }
    }
}
```

### Explanation:
- The `ClientHandler` class implements `Runnable` and is used to handle each client connection in a separate thread.
- It reads the incoming message from the client and prints the client's address and port.

---

## Main (Server)

The `Main` class for the server initializes a fixed thread pool to handle client connections and uses a semaphore to control access to the shared resource.

### Code:

```java
package ui;

import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(1); 
        Semaphore semaphore = new Semaphore(1); 
        try {
            // Gestion de conexiones desde el servidor
            System.out.println("Esperando conexión de un cliente");
            ServerSocket serverSocket = new ServerSocket(5000);

            while (true) {
                // Accepting client connections
                Socket socket = serverSocket.accept();
                System.out.println("El cliente: " + socket.getInetAddress()
                        + " Se ha conectado ...");
                semaphore.acquire();
                pool.execute(new ClientHandler(socket));
                System.out.println("La conexión ha terminado ...");
                semaphore.release();
            }
        } catch (Exception e) {
            // TODO: handle exception
            pool.shutdown();
        }
    }
}
```

### Explanation:
- The server listens for incoming connections from clients.
- It uses a semaphore to ensure that only one client is processed at a time.
- A fixed thread pool is used to execute the `ClientHandler` tasks concurrently.

---

## Main (Client)

The `Main` class for the client establishes a connection to the server and sends a message.

### Code:

```java
package ui;

import java.io.BufferedWriter;
import java.io.OutputStreamWriter;
import java.net.Socket;

public class Main {
    public static void main(String[] args) {
        try {
            // Create a socket and send a message to the server
            Socket socket = new Socket("127.0.0.1", 5000);
            BufferedWriter writer = new BufferedWriter(
                new OutputStreamWriter(socket.getOutputStream())
            ); 

            writer.write("Hola desde el cliente: " + socket.getLocalAddress());

            Thread.sleep(1000);

            writer.flush();
            writer.close();
            socket.close();
        } catch (Exception e) {
            // TODO: handle exception
        }
    }
}
```

### Explanation:
- The client creates a socket to connect to the server at `127.0.0.1` on port `5000`.
- It sends a message, waits for 1 second, and then closes the connection.

---

## Explanation

1. **ClientHandler**: Handles each client's request in a separate thread using `Runnable`. It reads and prints the incoming message.
2. **Main (Server)**: Manages client connections using a fixed thread pool and synchronizes access to the resource using a semaphore.
3. **Main (Client)**: Sends a message to the server and establishes a connection.

---


</details>



<details>
  <summary>taskManager</summary>


# Task Manager Server with JDBC and Task Management

This document outlines the structure and implementation of a task manager server using JDBC for PostgreSQL. The server handles requests such as creating, updating, and fetching tasks and task stages through TCP communication. The application uses the DAO pattern for data persistence.

---

## ConnectionManager

The `ConnectionManager` class manages the database connection. It can be configured using environment variables or provided directly in the constructor.

### Code:

```java
package DBConfig;

import java.sql.Connection;
import java.sql.DriverManager;

public class ConnectionManager {

    private String url;
    private String user;
    private String password;

    private static ConnectionManager manager;

    public static ConnectionManager getInstance(){
        if(manager == null){
            manager = new ConnectionManager();
        }
        return manager;
    }

    public static ConnectionManager getInstance(String url, String user, String password){
        if(manager == null){
            manager = new ConnectionManager(url, user, password);
        }
        return manager;
    }

    private ConnectionManager(String url, String user, String password){
        this.url = url;
        this.user = user;
        this.password = password;
    }

    private ConnectionManager(){
        this.url = System.getenv("url");
        this.user = System.getenv("user");
        this.password = System.getenv("password");
    }

    public Connection getConnection(){
        try {
            Class.forName("org.postgresql.Driver");
            Connection con = DriverManager.getConnection(url, user, password);
            return con;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### Explanation:
- Manages the database connection using JDBC.
- The `getConnection` method establishes a connection to PostgreSQL.

---

## Dao (Generic Interface)

The `Dao` interface defines the CRUD operations for entities. It is implemented by `TaskDaoDB` and `StageDaoDB` to interact with the database.

### Code:

```java
package daos;

import java.util.List;

public interface Dao<T, P> {
    public List<T> findAll();
    public T findById(P id);
    public T update(T newEntity);
    public void delete(T entity);
    public void save(T entity);
}
```

### Explanation:
- Defines methods for basic CRUD operations: `findAll`, `findById`, `update`, `delete`, and `save`.

---

## StageDaoDB

The `StageDaoDB` class implements the `Dao` interface for `TaskStage` entities. It interacts with the `task_stage` table in the PostgreSQL database.

### Code:

```java
package daos;

import model.TaskStage;
import java.util.List;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;

public class StageDaoDB implements Dao<TaskStage, Integer> {

    public StageDaoDB() {}

    @Override
    public List<TaskStage> findAll(){
        List<TaskStage> stages = new ArrayList<>();
        String query = "SELECT * FROM task_stage";
        try (Connection conn = ConnectionManager.getInstance().getConnection()){
            Statement statement = conn.createStatement();
            ResultSet result = statement.executeQuery(query);

            while (result.next()) {
                TaskStage stage = new TaskStage();
                stage.setId(result.getInt("id"));
                stage.setName(result.getString("name"));
                stage.setDescription(result.getString("description"));
                stages.add(stage);
            }
        }
        catch(Exception e) {}

        return stages;
    }

    @Override
    public TaskStage findById(Integer id){
        TaskStage stage = null;
        String query = "SELECT * FROM task_stage WHERE id = ?";
        try (Connection conn = ConnectionManager.getInstance().getConnection()){
            PreparedStatement statement = conn.prepareStatement(query);
            statement.setInt(1, id);
            try (ResultSet result = statement.executeQuery()){
                if (result.next()) {
                    stage = new TaskStage();
                    stage.setId(result.getInt("id"));
                    stage.setName(result.getString("name"));
                    stage.setDescription(result.getString("description"));
                }
            }
        }
        catch (Exception e) {}

        return stage;
    }

    @Override
    public TaskStage update(TaskStage newEntity) {
        throw new UnsupportedOperationException("La actualización de stages no está permitida.");
    }

    @Override
    public void delete(TaskStage entity) {
        throw new UnsupportedOperationException("La eliminación de stages no está permitida.");
    }

    @Override
    public void save(TaskStage entity) {
        throw new UnsupportedOperationException("La creación de nuevos stages no está permitida.");
    }
}
```

### Explanation:
- Implements `Dao` interface for `TaskStage` entities.
- Fetches task stages from the `task_stage` table.

---

## TaskDaoDB

The `TaskDaoDB` class implements the `Dao` interface for `Task` entities. It interacts with the `task` table in PostgreSQL.

### Code:

```java
package daos;

import model.Task;
import model.TaskStage;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

public class TaskDaoDB implements Dao<Task, Integer>{

    @Override
    public List<Task> findAll() {
        try {
            List<Task> tasks = new ArrayList<>();
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "SELECT * FROM task";
            Statement statement = conn.createStatement();
            ResultSet result = statement.executeQuery(query);

            while (result.next()) {
                Task t = new Task();
                t.setId(result.getInt("id"));
                t.setTitle(result.getString("title"));
                t.setDescription(result.getString("description"));
                int stageID = result.getInt("stage_id");
                TaskStage stage = new TaskStage();
                stage.setId(stageID);
                tasks.add(t);
            }

            conn.close();
            return tasks;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Task findById(Integer id) {
        throw new UnsupportedOperationException("Unimplemented method 'findById'");
    }

    @Override
    public Task update(Task oldEntity) {
        throw new UnsupportedOperationException("Unimplemented method 'update'");
    }

    @Override
    public void delete(Task entity) {
        throw new UnsupportedOperationException("Unimplemented method 'delete'");
    }

    @Override
    public void save(Task entity) {
        String query = "INSERT INTO task (title, description, due_date, priority, stage_id) VALUES (?, ?, ?, ?, ?)";
        try (Connection conn = ConnectionManager.getInstance().getConnection()){
            PreparedStatement statement = conn.prepareStatement(query, Statement.RETURN_GENERATED_KEYS);
            statement.setString(1, entity.getTitle());
            statement.setString(2, entity.getDescription());
            statement.setString(3, entity.getDueDate());
            statement.setString(4, entity.getPriority());
            statement.setInt(5, entity.getStage().getId());

            statement.executeUpdate();

            try (ResultSet generatedKeys = statement.getGeneratedKeys()) {
                if (generatedKeys.next()) {
                    entity.setId(generatedKeys.getInt(1));
                }
            }
        } catch(Exception e) {}
    }
}
```

### Explanation:
- Implements `Dao` interface for `Task` entities.
- Provides functionality for saving tasks to the `task` table.

---

## ManagerServer

The `ManagerServer` class handles TCP connections and processes incoming requests. It uses `TaskServices` for managing tasks and stages.

### Code:

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import com.google.gson.Gson;

import dtos.Request;
import model.TaskStage;
import services.TaskServices;

public class ManagerServer {

    private Gson gson;
    private TaskServices services;
    private boolean running;

    public static void main(String[] args) throws Exception {
        new ManagerServer();
    }

    public ManagerServer() throws Exception {
        gson = new Gson();
        services = new TaskServices();
        ServerSocket socket = new ServerSocket(5000);
        running = true;
        while (running) {
            Socket sc = socket.accept();
            resolveClient(sc);
        }
        socket.close();
    }

    public void resolveClient(Socket sc) throws IOException {
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(sc.getInputStream()));
            BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(sc.getOutputStream()));
            while (true) {
                String line = br.readLine();
                String resp = "";
                Request rq = gson.fromJson(line, Request.class);
                switch (rq.getCommand()) {
                    case "GET_TASKS":
                        List<TaskStage> stages = services.getTasks();
                        resp = gson.toJson(stages);
                        break;

                    default:
                        Map<String, String> map = new HashMap<>();
                        map.put("msg", "Operation not supported");
                        resp = gson.toJson(map);
                        break;
                }
                System.out.println(resp);
                writer.write(resp + "
");
                writer.flush();
            }
        } catch (Exception e) {
            sc.close();
        }
    }
}
```

### Explanation:
- The `ManagerServer` listens on port `5000` and handles client requests in JSON format.
- It uses `TaskServices` to manage task operations like fetching tasks.

---

## Conclusion

This server uses **JDBC** for database persistence and **DAO** pattern for managing tasks. It communicates with clients over **TCP** using **JSON** for data serialization. The server provides basic CRUD operations for tasks and stages.

---


</details>



<details>
  <summary>Otros apuntes</summary>
  

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

</details>


<details>
  <summary>Chat Naval</summary>
  
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


</details>


<details>
  <summary>Davinchi</summary>
  

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



</details>



<details>
  <summary>Regalito</summary>

  
# Compunet — Implementación Integrada (JDBC + Sockets TCP + ThreadPool + Semáforos + GSON)

**Origen:** Este documento se creó tomando como referencia tu archivo `Compunet.md`. fileciteturn0file0

**Objetivo:** Proporcionar una implementación educativa y reproducible en Java que **integre** los 5 puntos solicitados:
1. JDBC (conexión entre Java y bases de datos relacionales)
2. Uso de sockets TCP
3. Uso de Thread Pools y sincronización
4. Uso de semáforos
5. Serialización JSON con GSON

> El código está organizado por paquetes y pensado para ser fácil de leer, ejecutar y adaptar.

---

## Estructura sugerida del proyecto (maven / gradle)
```
src/
  main/
    java/
      db/ConnectionManager.java
      model/Task.java
      model/TaskStage.java
      dao/TaskDaoDB.java
      dto/Request.java
      dto/Response.java
      services/TaskServices.java
      server/ClientHandler.java
      server/ManagerServer.java
      client/SimpleClient.java
resources/
pom.xml (o build.gradle)
```

### Dependencias principales (Maven)
```xml
<!-- añadir en pom.xml -->
<dependencies>
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
  </dependency>

  <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
  </dependency>
</dependencies>
```

---

## 1) JDBC: `db/ConnectionManager.java`

```java
package db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * Simple ConnectionManager for PostgreSQL using JDBC.
 * Use environment variables or constructor to configure.
 */
public class ConnectionManager {
    private final String url;
    private final String user;
    private final String pass;

    public ConnectionManager(String url, String user, String pass) {
        this.url = url;
        this.user = user;
        this.pass = pass;
    }

    public Connection getConnection() throws SQLException {
        // Driver registration is automatic in modern JDBC; keep Class.forName if older JDKs
        return DriverManager.getConnection(url, user, pass);
    }
}
```

**Explicación corta:** `ConnectionManager` centraliza la creación de `Connection`. En producción usarías un pool (HikariCP) para evitar sobrecarga por conexiones repetidas.

---

## 2) Modelos y DTOs (usados para GSON)

`model/Task.java`

```java
package model;

public class Task {
    private Integer id;
    private String title;
    private String description;
    private String dueDate;
    private String priority;
    private TaskStage stage;

    // constructor vacío, getters y setters
    public Task() {}
    // getters & setters ...
}
```

`model/TaskStage.java`

```java
package model;

public class TaskStage {
    private Integer id;
    private String name;
    private String description;
    // getters & setters ...
}
```

`dto/Request.java` (peticiones entrantes por TCP, en JSON)

```java
package dto;

public class Request {
    private String command;
    private String payload; // JSON string adicional opcional

    public Request() {}
    public String getCommand() { return command; }
    public void setCommand(String command) { this.command = command; }
    public String getPayload() { return payload; }
    public void setPayload(String payload) { this.payload = payload; }
}
```

`dto/Response.java`

```java
package dto;

public class Response {
    private boolean ok;
    private String message;
    private Object data; // serializable por GSON

    public Response() {}
    // getters/setters
}
```

---

## 3) DAO con JDBC: `dao/TaskDaoDB.java`

```java
package dao;

import model.Task;
import model.TaskStage;
import db.ConnectionManager;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class TaskDaoDB {
    private final ConnectionManager cm;

    public TaskDaoDB(ConnectionManager cm){
        this.cm = cm;
    }

    public List<Task> findAll() throws SQLException {
        String q = "SELECT t.id, t.title, t.description, t.due_date, t.priority, t.stage_id, s.name as stage_name " +
                   "FROM task t LEFT JOIN task_stage s ON t.stage_id = s.id";
        List<Task> list = new ArrayList<>();
        try (Connection c = cm.getConnection();
             PreparedStatement ps = c.prepareStatement(q);
             ResultSet rs = ps.executeQuery()) {
            while (rs.next()){
                Task t = new Task();
                t.setId(rs.getInt("id"));
                t.setTitle(rs.getString("title"));
                t.setDescription(rs.getString("description"));
                t.setDueDate(rs.getString("due_date"));
                t.setPriority(rs.getString("priority"));
                TaskStage stage = new TaskStage();
                stage.setId(rs.getInt("stage_id"));
                stage.setName(rs.getString("stage_name"));
                t.setStage(stage);
                list.add(t);
            }
        }
        return list;
    }

    public void save(Task task) throws SQLException {
        String q = "INSERT INTO task (title, description, due_date, priority, stage_id) VALUES (?, ?, ?, ?, ?)";
        try (Connection c = cm.getConnection();
             PreparedStatement ps = c.prepareStatement(q, Statement.RETURN_GENERATED_KEYS)) {
            ps.setString(1, task.getTitle());
            ps.setString(2, task.getDescription());
            ps.setString(3, task.getDueDate());
            ps.setString(4, task.getPriority());
            ps.setObject(5, task.getStage() != null ? task.getStage().getId() : null);
            ps.executeUpdate();
            try (ResultSet gk = ps.getGeneratedKeys()){
                if (gk.next()) task.setId(gk.getInt(1));
            }
        }
    }
}
```

**Notas:** usar `PreparedStatement` para evitar inyección. Manejar transacciones si haces varias operaciones.

---

## 4) Serialización JSON (GSON)

```java
import com.google.gson.Gson;
Gson gson = new Gson();

// Objeto a JSON
String json = gson.toJson(someObject);

// JSON a objeto
Task t = gson.fromJson(json, Task.class);
```

GSON hace coincidir campos por nombre; usa `transient` para excluir campos si necesario.

---

## 5) Sockets TCP + ThreadPool + Semáforos (servidor multi-hilo)

`server/ClientHandler.java`

```java
package server;

import java.io.*;
import java.net.Socket;
import com.google.gson.Gson;
import dto.Request;
import dto.Response;
import services.TaskServices;

/**
 * Cada ClientHandler atiende a un cliente en su propio hilo (ejecutado por ExecutorService).
 * Usa TaskServices para operar con la base de datos. Para limitar accesos concurrentes a DB
 * se utiliza un Semaphore en TaskServices (o a nivel de ManagerServer).
 */
public class ClientHandler implements Runnable {
    private final Socket socket;
    private final TaskServices services;
    private final Gson gson = new Gson();

    public ClientHandler(Socket socket, TaskServices services) {
        this.socket = socket;
        this.services = services;
    }

    @Override
    public void run() {
        try (
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))
        ) {
            String line;
            while ((line = br.readLine()) != null) {
                Request req = gson.fromJson(line, Request.class);
                Response resp = new Response();
                try {
                    if ("GET_TASKS".equals(req.getCommand())) {
                        resp.setOk(true);
                        resp.setData(services.getTasks()); // lista de Task serializable por GSON
                    } else if ("CREATE_TASK".equals(req.getCommand())) {
                        // payload contiene JSON con Task
                        Task newTask = gson.fromJson(req.getPayload(), model.Task.class);
                        services.createTask(newTask);
                        resp.setOk(true);
                        resp.setMessage("Task created");
                    } else {
                        resp.setOk(false);
                        resp.setMessage("Command not supported");
                    }
                } catch (Exception ex) {
                    resp.setOk(false);
                    resp.setMessage("Server error: " + ex.getMessage());
                }
                String out = gson.toJson(resp);
                bw.write(out + "\n");
                bw.flush();
            }
        } catch (IOException e) {
            // cliente desconectado o error I/O
        } finally {
            try { socket.close(); } catch (IOException ignored) {}
        }
    }
}
```

`server/ManagerServer.java`

```java
package server;

import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import services.TaskServices;

/**
 * ManagerServer: acepta clientes con ServerSocket, delega la atención a un ThreadPool.
 * Además, comparte la instancia de TaskServices (que contiene un Semaphore para limitar
 * concurrencia sobre la base de datos).
 */
public class ManagerServer {
    private final int port;
    private final ExecutorService pool;
    private final TaskServices services;

    public ManagerServer(int port, int poolSize, TaskServices services) {
        this.port = port;
        this.pool = Executors.newFixedThreadPool(poolSize);
        this.services = services;
    }

    public void start() throws Exception {
        try (ServerSocket server = new ServerSocket(port)) {
            System.out.println("Server listening on " + port);
            while (true) {
                Socket client = server.accept();
                System.out.println("Accepted from " + client.getInetAddress());
                pool.execute(new ClientHandler(client, services));
            }
        } finally {
            pool.shutdown();
        }
    }

    public static void main(String[] args) throws Exception {
        // Configuración de ejemplo:
        // DB url en formato jdbc:postgresql://host:5432/db
        services.TaskServices services = new services.TaskServices(
            new db.ConnectionManager("jdbc:postgresql://localhost:5432/compunet", "user", "pass")
        );
        new ManagerServer(5000, 8, services).start();
    }
}
```

**Puntos clave:** El `ExecutorService` permite manejar N clientes en paralelo sin crear hilos continuamente. `TaskServices` debe contener sincronización/semáforos para proteger la BD o recursos limitados.

---

## 6) Servicios y Semáforos: `services/TaskServices.java`

```java
package services;

import dao.TaskDaoDB;
import db.ConnectionManager;
import model.Task;

import java.util.List;
import java.util.concurrent.Semaphore;

/**
 * TaskServices gobierna el acceso a DAO usando un Semaphore para limitar
 * cuántas operaciones concurrentes de BD se permiten.
 */
public class TaskServices {
    private final TaskDaoDB dao;
    // Limitar p.ej. 5 conexiones simultáneas a la base de datos
    private final Semaphore dbSemaphore = new Semaphore(5);

    public TaskServices(ConnectionManager cm) {
        this.dao = new TaskDaoDB(cm);
    }

    // interfaz por simplicidad
    public List<Task> getTasks() throws Exception {
        dbSemaphore.acquire();
        try {
            return dao.findAll();
        } finally {
            dbSemaphore.release();
        }
    }

    public void createTask(Task t) throws Exception {
        dbSemaphore.acquire();
        try {
            dao.save(t);
        } finally {
            dbSemaphore.release();
        }
    }
}
```

**Explicación:** `dbSemaphore` controla cuántas operaciones a la base de datos se ejecutan en paralelo (útil cuando la BD o la red tiene límite de concurrencia).

---

## 7) Cliente simple que pide tareas: `client/SimpleClient.java`

```java
package client;

import java.io.*;
import java.net.Socket;
import com.google.gson.Gson;
import dto.Request;
import dto.Response;

public class SimpleClient {
    public static void main(String[] args) throws Exception {
        try (Socket s = new Socket("127.0.0.1", 5000);
             BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
             BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()))) {

            Gson gson = new Gson();
            Request r = new Request();
            r.setCommand("GET_TASKS");
            bw.write(gson.toJson(r) + "\n");
            bw.flush();

            String line = br.readLine();
            Response resp = gson.fromJson(line, Response.class);
            System.out.println("Server response: " + gson.toJson(resp));
        }
    }
}
```

---

## 8) SQL (esquema mínimo)
```sql
CREATE TABLE task_stage (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  description TEXT
);

CREATE TABLE task (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  description TEXT,
  due_date VARCHAR(50),
  priority VARCHAR(20),
  stage_id INTEGER REFERENCES task_stage(id)
);
```

---

## 9) Sincronización adicional y `synchronized`
- Usa `synchronized` para proteger estructuras en memoria (por ejemplo: un `Map` compartido por hilos que guarda sesiones).
- Usa `volatile` para banderas leídas por varios hilos.
- Evita bloqueos largos dentro de secciones críticas.

---

## 10) Consejos de despliegue y pruebas
1. Configura la BD y crea tablas antes de ejecutar.
2. Ajusta el tamaño del thread pool según CPU/IO (por ejemplo `poolSize = cores * 2` como regla base).
3. Monitorea latencias de BD: si la BD se satura, reduce el semáforo o mejora el pool de conexiones.
4. En producción usa connection-pool (HikariCP), y considera TLS para sockets.

---

## 11) Resumen conceptual (rápido)

- **JDBC**: API para ejecutar SQL desde Java; usa `PreparedStatement` y cierra recursos.
- **Sockets TCP**: `ServerSocket.accept()` crea `Socket`; cliente conecta con `new Socket(host, port)`.
- **ThreadPool**: `ExecutorService` para ejecutar tareas concurrentes y limitar creación de hilos.
- **Semáforos**: controlan cuántos hilos acceden a un recurso; `new Semaphore(n)`, `acquire()`, `release()`.
- **GSON**: serializa/deserializa JSON con `toJson()` y `fromJson()`.

---

## 12) ¿Qué entrego en este archivo?
- Código de ejemplo comentado que integra los 5 puntos.
- Esquema SQL y recomendaciones para producción.
- Instrucciones para compilar y ejecutar.

---

## Licencia educativa
Puedes reutilizar y adaptar este material para aprendizaje o proyecto personal. No incluyo dependencias propietarias.




</details>


<details>
  <summary>Regalito 2</summary>
  

# Compunet — Resumen: Los 5 puntos clave (lista descargable)

Este archivo contiene explicaciones concisas y ejemplos mínimos para cada uno de los **5 puntos** solicitados.

1) **JDBC (Conexión Java ↔ BD relacional)**
- Objetivo: ejecutar consultas SQL desde Java y persistir datos.
- Pasos:
  - Registrar driver (opcional en JDK reciente).
  - Crear `Connection` con `DriverManager.getConnection(url,user,pass)`.
  - Usar `PreparedStatement` y `ResultSet`.
  - Cerrar recursos (try-with-resources).
- Mini-ejemplo:
```java
try (Connection c = DriverManager.getConnection(url,user,pass);
     PreparedStatement ps = c.prepareStatement("SELECT * FROM task");
     ResultSet rs = ps.executeQuery()) {
    while (rs.next()) { System.out.println(rs.getString("title")); }
}
```

2) **Sockets TCP**
- Servidor: `ServerSocket server = new ServerSocket(port); Socket client = server.accept();`
- Cliente: `Socket s = new Socket(host, port);`
- Envío/Recepción: `BufferedReader`/`BufferedWriter` con `readLine()` / `write(...+"\n")`.
- Mini-ejemplo cliente:
```java
Socket s = new Socket("localhost", 5000);
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
bw.write("hola\n"); bw.flush();
```

3) **Thread Pools y sincronización**
- `ExecutorService pool = Executors.newFixedThreadPool(n);`
- `pool.execute(new RunnableTask())`
- `synchronized` protege métodos o bloques.
- Mini-ejemplo:
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.execute(() -> { synchronized(obj) { /* sección crítica */ } });
pool.shutdown();
```

4) **Semáforos**
- Controlan cuántos hilos pueden entrar simultáneamente.
- `Semaphore sem = new Semaphore(permits); sem.acquire(); ... sem.release();`
- Usos: limitar accesos a DB, implementar turnos, limitar recursos físicos.
- Mini-ejemplo:
```java
Semaphore sem = new Semaphore(1); // mutex
sem.acquire();
try { /* recurso */ } finally { sem.release(); }
```

5) **Serialización JSON con GSON**
- Convertir objetos Java ↔ JSON: `Gson gson = new Gson(); gson.toJson(obj); gson.fromJson(json, Class.class);`
- Útil para enviar objetos por sockets o almacenar configuraciones.
- Mini-ejemplo:
```java
Gson gson = new Gson();
String json = gson.toJson(myTask);
Task t = gson.fromJson(json, Task.class);
```

---

## Recomendación rápida para unir todo
- Servidor TCP con `ExecutorService`.
- `ClientHandler` parsea JSON (GSON) en `Request`.
- `TaskServices` ejecuta operaciones con `TaskDaoDB` (JDBC) controladas por `Semaphore`.
- Cliente envía comandos JSON y recibe `Response` JSON.



</details>


<details>
  <summary>Regalito 4</summary>
  
# Compunet — Implementación Cliente-Servidor (TCP) Integrada
**Basado en tu material original.** Fragmentos y ejemplos referenciados del archivo que me enviaste. fileciteturn2file0

--- 
## Objetivo
Crear una **implementación cliente-servidor TCP en Java** que integre:
1. JDBC (conexión a BD relacionales)
2. Sockets TCP
3. Thread pool (ExecutorService) y sincronización (`synchronized`)
4. Semáforos (`Semaphore`)
5. Serialización JSON usando **GSON**

Además se entrega:
- Código ejemplo listo para copiar/pegar y adaptar.
- Explicaciones paso a paso y recomendaciones de despliegue.
- Esquema SQL mínimo.

---

## Estructura de proyecto (sugerida)
```
src/
  main/
    java/
      db/ConnectionManager.java
      dao/TaskDaoDB.java
      model/Task.java
      dto/Request.java
      dto/Response.java
      services/TaskServices.java
      server/ClientHandler.java
      server/ManagerServer.java
      client/SimpleClient.java
resources/
pom.xml
```

Dependencias Maven (fragmento para `pom.xml`):
```xml
<dependencies>
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
  </dependency>
  <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
  </dependency>
</dependencies>
```

---

## SQL — esquema mínimo
```sql
CREATE TABLE task_stage (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  description TEXT
);

CREATE TABLE task (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  description TEXT,
  due_date VARCHAR(50),
  priority VARCHAR(20),
  stage_id INTEGER REFERENCES task_stage(id)
);
```

---

## 1) JDBC — `db/ConnectionManager.java`
```java
package db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * ConnectionManager simple. En producción usar pool (HikariCP).
 */
public class ConnectionManager {
    private final String url;
    private final String user;
    private final String pass;

    public ConnectionManager(String url, String user, String pass) {
        this.url = url;
        this.user = user;
        this.pass = pass;
    }

    public Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, user, pass);
    }
}
```

**Concepto:** centraliza la creación de `Connection`. Usar `try-with-resources` al ejecutar consultas.

---

## 2) Modelos y DTOs (para GSON)

`model/Task.java`
```java
package model;

public class Task {
    private Integer id;
    private String title;
    private String description;
    private String dueDate;
    private String priority;
    // getters, setters, constructor vacío
}
```

`dto/Request.java`
```java
package dto;
public class Request {
    private String command; // e.g., "GET_TASKS", "CREATE_TASK"
    private String payload; // JSON payload (p.ej. Task JSON)
    public Request(){}
    // getters/setters...
}
```

`dto/Response.java`
```java
package dto;
public class Response {
    private boolean ok;
    private String message;
    private Object data;
    public Response(){}
    // getters/setters...
}
```

---

## 3) DAO con JDBC — `dao/TaskDaoDB.java`
```java
package dao;

import db.ConnectionManager;
import model.Task;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class TaskDaoDB {
    private final ConnectionManager cm;

    public TaskDaoDB(ConnectionManager cm){
        this.cm = cm;
    }

    public List<Task> findAll() throws SQLException {
        String q = "SELECT t.id, t.title, t.description, t.due_date, t.priority, t.stage_id FROM task t";
        List<Task> list = new ArrayList<>();
        try (Connection c = cm.getConnection();
             PreparedStatement ps = c.prepareStatement(q);
             ResultSet rs = ps.executeQuery()) {
            while (rs.next()){
                Task t = new Task();
                t.setId(rs.getInt("id"));
                t.setTitle(rs.getString("title"));
                t.setDescription(rs.getString("description"));
                t.setDueDate(rs.getString("due_date"));
                t.setPriority(rs.getString("priority"));
                list.add(t);
            }
        }
        return list;
    }

    public void save(Task task) throws SQLException {
        String q = "INSERT INTO task (title, description, due_date, priority, stage_id) VALUES (?, ?, ?, ?, ?)";
        try (Connection c = cm.getConnection();
             PreparedStatement ps = c.prepareStatement(q, Statement.RETURN_GENERATED_KEYS)) {
            ps.setString(1, task.getTitle());
            ps.setString(2, task.getDescription());
            ps.setString(3, task.getDueDate());
            ps.setString(4, task.getPriority());
            ps.setObject(5, null);
            ps.executeUpdate();
            try (ResultSet gk = ps.getGeneratedKeys()){
                if (gk.next()) task.setId(gk.getInt(1));
            }
        }
    }
}
```

**Nota:** usa `PreparedStatement` para evitar inyección. Maneja transacciones si hay varias operaciones dependientes.

---

## 4) Servicios + Semáforos — `services/TaskServices.java`
```java
package services;

import dao.TaskDaoDB;
import db.ConnectionManager;
import model.Task;

import java.util.List;
import java.util.concurrent.Semaphore;

public class TaskServices {
    private final TaskDaoDB dao;
    private final Semaphore dbSemaphore;

    public TaskServices(ConnectionManager cm, int dbConcurrencyLimit) {
        this.dao = new TaskDaoDB(cm);
        this.dbSemaphore = new Semaphore(dbConcurrencyLimit);
    }

    public List<Task> getTasks() throws Exception {
        dbSemaphore.acquire();
        try {
            return dao.findAll();
        } finally {
            dbSemaphore.release();
        }
    }

    public void createTask(Task t) throws Exception {
        dbSemaphore.acquire();
        try {
            dao.save(t);
        } finally {
            dbSemaphore.release();
        }
    }
}
```

**Explicación:** `Semaphore` limita cuántas operaciones concurrentes hacia la BD se permiten. Útil si la BD no soporta muchas conexiones simultáneas.

---

## 5) Sockets TCP + ThreadPool + sincronización

### `server/ClientHandler.java`
```java
package server;

import dto.Request;
import dto.Response;
import com.google.gson.Gson;
import model.Task;
import services.TaskServices;

import java.io.*;
import java.net.Socket;
import java.util.List;

public class ClientHandler implements Runnable {
    private final Socket socket;
    private final TaskServices services;
    private final Gson gson = new Gson();

    public ClientHandler(Socket socket, TaskServices services) {
        this.socket = socket;
        this.services = services;
    }

    @Override
    public void run() {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))) {

            String line;
            while ((line = br.readLine()) != null) {
                Request req = gson.fromJson(line, Request.class);
                Response resp = new Response();
                try {
                    switch (req.getCommand()) {
                        case "GET_TASKS":
                            List<Task> tasks = services.getTasks();
                            resp.setOk(true);
                            resp.setData(tasks);
                            break;
                        case "CREATE_TASK":
                            Task t = gson.fromJson(req.getPayload(), Task.class);
                            services.createTask(t);
                            resp.setOk(true);
                            resp.setMessage("Task created");
                            break;
                        default:
                            resp.setOk(false);
                            resp.setMessage("Command not supported");
                    }
                } catch (Exception e) {
                    resp.setOk(false);
                    resp.setMessage("Server error: " + e.getMessage());
                }
                String out = gson.toJson(resp);
                bw.write(out + "\n");
                bw.flush();
            }
        } catch (IOException e) {
            // cliente desconectado o error
        } finally {
            try { socket.close(); } catch (IOException ignored) {}
        }
    }
}
```

### `server/ManagerServer.java`
```java
package server;

import db.ConnectionManager;
import services.TaskServices;

import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ManagerServer {
    public static void main(String[] args) throws Exception {
        String jdbcUrl = "jdbc:postgresql://localhost:5432/compunet";
        String user = "user";
        String pass = "pass";
        ConnectionManager cm = new ConnectionManager(jdbcUrl, user, pass);

        // limite concurrente a BD = 5 (ejemplo)
        TaskServices services = new TaskServices(cm, 5);

        int poolSize = Runtime.getRuntime().availableProcessors() * 2;
        ExecutorService pool = Executors.newFixedThreadPool(poolSize);

        try (ServerSocket server = new ServerSocket(5000)) {
            System.out.println("Server listening on 5000");
            while (true) {
                Socket client = server.accept();
                pool.execute(new ClientHandler(client, services));
            }
        } finally {
            pool.shutdown();
        }
    }
}
```

**Conceptos clave:**
- `ServerSocket.accept()` bloquea hasta nueva conexión.
- `ExecutorService` evita crear un hilo por cada conexión, reutiliza hilos.
- `Semaphore` en `TaskServices` controla concurrencia a la BD.

---

## 6) Cliente ejemplo — `client/SimpleClient.java`
```java
package client;

import com.google.gson.Gson;
import dto.Request;
import dto.Response;
import model.Task;

import java.io.*;
import java.net.Socket;
import java.util.List;

public class SimpleClient {
    public static void main(String[] args) throws Exception {
        try (Socket s = new Socket("127.0.0.1", 5000);
             BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
             BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()))) {

            Gson gson = new Gson();

            // GET_TASKS
            Request r = new Request();
            r.setCommand("GET_TASKS");
            bw.write(gson.toJson(r) + "\n");
            bw.flush();

            String line = br.readLine();
            Response resp = gson.fromJson(line, Response.class);
            System.out.println("GET_TASKS -> " + gson.toJson(resp));

            // CREATE_TASK (ejemplo)
            Task t = new Task();
            t.setTitle("Prueba desde cliente");
            t.setDescription("Descripción");
            Request cr = new Request();
            cr.setCommand("CREATE_TASK");
            cr.setPayload(gson.toJson(t));
            bw.write(gson.toJson(cr) + "\n");
            bw.flush();

            System.out.println("CREATE_TASK enviado");
        }
    }
}
```

---

## 7) Consideraciones sobre sincronización adicional
- Usa `synchronized` si compartes estructuras en memoria (p.ej. `Map` de sesiones).
- Evita bloquear hilos dentro de secciones críticas por mucho tiempo.
- Usa `volatile` para banderas simples leídas por varios hilos.

---

## 8) Pruebas y despliegue
1. Crear BD y tablas según el esquema SQL.
2. Ajustar `dbConcurrencyLimit` y `poolSize` según pruebas de carga.
3. En producción: usar HikariCP o similar para pool de conexiones en lugar de un `Semaphore` casero.
4. Considerar TLS sobre sockets si datos son sensibles; alternativamente usar HTTP+TLS y un framework web.

---

## 9) Referencias
Fragmentos y ejemplos tomados y adaptados del material que me enviaste. fileciteturn2file0

---

Fin del archivo.


</details>

<details>
  <summary>Regalito 5</summary>
  
# Compunet — Resumen: Los 5 puntos (descargable)
Basado en el material original que compartiste. fileciteturn2file0

1) **JDBC (Conexión Java ↔ BD relacional)**
- Objetivo: ejecutar SQL desde Java.
- Pasos clave: `DriverManager.getConnection(url,user,pass)`, `PreparedStatement`, `ResultSet`, cerrar recursos.
- Buenas prácticas: usar pool de conexiones en producción; `PreparedStatement` para evitar inyección.

2) **Sockets TCP**
- Servidor: `ServerSocket server = new ServerSocket(port); Socket client = server.accept();`
- Cliente: `Socket s = new Socket(host, port);`
- Comunicación: `BufferedReader`/`BufferedWriter` con `readLine()`/`write(... + "\n")`.
- Consejo: manejar I/O en hilos (thread pool) para multiples clientes.

3) **Thread Pools y sincronización**
- `ExecutorService pool = Executors.newFixedThreadPool(n);`
- `pool.execute(Runnable)` para delegar trabajo.
- `synchronized` para proteger recursos compartidos; `volatile` para banderas.

4) **Semáforos**
- Uso: limitar hilos que acceden a recurso (`Semaphore sem = new Semaphore(permits)`).
- Métodos: `acquire()` (bloquea hasta permiso) y `release()`.
- Uso típico: limitar acceso concurrente a BD o controlar turnos en juegos.

5) **Serialización JSON con GSON**
- `Gson gson = new Gson(); gson.toJson(obj); gson.fromJson(json, Clase.class);`
- Útil para enviar/recibir objetos por sockets o guardar configuraciones.

--- 
## Mini-guía de integración
- Servidor TCP con `ServerSocket` + `ExecutorService`.
- `ClientHandler` parsea JSON con GSON a `Request`, llama a `TaskServices`.
- `TaskServices` usa `Semaphore` para limitar operaciones en `TaskDaoDB` (JDBC).
- Cliente envía `Request` en JSON y recibe `Response` en JSON.

Referencia y ejemplos: material original. fileciteturn2file0


</details>

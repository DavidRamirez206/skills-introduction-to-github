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


<details>
  <summary>Alejandro</summary>

TaskManagerV2

----------------------------------------------------------------------------------------------
task-manager/managerServer/src/main/java/DBConfig/ConnectionManager.java

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

----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/daos/Dao.java

package daos;

import java.util.List;

public interface Dao<T, P> {
    public List<T> findAll();
    public T findById(P id);
    public T update(T newEntity);
    public void delete(T entity);
    public void save(T entity);
}
----------------------------------------------------------------------------------------------
task-manager/managerServer/src/main/java/daos/StageDao.java

package daos;

import java.util.ArrayList;
import java.util.List;

import model.TaskStage;

public class StageDao implements Dao<TaskStage, Integer>{

    private List<TaskStage> stages = new ArrayList<>();
    private int count = 0;

    public StageDao(){
        TaskStage todo = new TaskStage();
        todo.setName("TO DO");
        todo.setDescription("Tasks to be done");
        save(todo);

        TaskStage inProgress = new TaskStage();
        inProgress.setName("IN PROGRESS");
        inProgress.setDescription("Tasks in progress");
        save(inProgress);

        TaskStage done = new TaskStage();
        done.setName("DONE");
        done.setDescription("Completed tasks");
        save(done);
    }

    @Override
    public List<TaskStage> findAll() {
        return stages;
    }

    @Override
    public TaskStage findById(Integer id) {
        return stages.stream()
            .filter(s -> s.getId() == id)
            .findFirst().orElse(null);
    }

    @Override
    public TaskStage update(TaskStage newEntity) {
        TaskStage exist = findById(newEntity.getId());
        if(exist != null){
            exist.setDescription(newEntity.getDescription());
            exist.setName(newEntity.getName());
            exist.setTasks(newEntity.getTasks());
        }
        return exist;
    }

    @Override
    public void delete(TaskStage entity) {
        TaskStage ent = findById(entity.getId());
        if(ent != null){
            stages.remove(ent);
        }
    }

    @Override
    public void save(TaskStage entity) {
        entity.setId(++count);
        stages.add(entity);
    }
    
}

----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/daos/TaskDaoDB.java

package daos;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

import DBConfig.ConnectionManager;
import model.Task;
import model.TaskStage;

public class TaskDaoDB implements Dao<Task, Integer>{

    @Override
    public List<Task> findAll() {
        try {
            List<Task> tasks = new ArrayList<>();
            Connection conn = ConnectionManager
            .getInstance(
).getConnection();
            String query = "Select * from task";

            Statement statement = conn.createStatement();

            ResultSet result = statement.executeQuery(query);

            while (result.next()) {
                Task t = new Task();
                //TODO completar mapeo de las entidades
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
        // TODO Auto-generated method stub
        throw new UnsupportedOperationException("Unimplemented method 'finById'");
    }

    @Override
    public Task update(Task oldEntity) {
        // TODO Auto-generated method stub
        throw new UnsupportedOperationException("Unimplemented method 'update'");
    }

    @Override
    public void delete(Task entity) {
        // TODO Auto-generated method stub
        throw new UnsupportedOperationException("Unimplemented method 'delete'");
    }

    @Override
    public void save(Task entity) {
        // TODO Auto-generated method stub
        throw new UnsupportedOperationException("Unimplemented method 'save'");
    }
    
}
----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/daos/TaskMemDao.java

package daos;

import java.util.ArrayList;
import java.util.List;

import model.Task;

public class TaskMemDao implements Dao<Task, Integer>{
    private List<Task> tasks = new ArrayList<>();

    private int count = 0;

    @Override
    public List<Task> findAll() {
        return tasks;
    }

    @Override
    public Task findById(Integer id) {
        return tasks.stream()
            .filter(t -> t.getId() == id)
            .findFirst().orElse(null);
    }

    @Override
    public Task update(Task newEntity) {
        Task exist = findById(newEntity.getId());
        if(exist != null){
            exist.setDescription(newEntity.getDescription());
            exist.setTitle(newEntity.getTitle());
            exist.setStage(newEntity.getStage());
            exist.setDueDate(newEntity.getDueDate());
            exist.setPriority(newEntity.getPriority());
        }
        return exist;
    }

    @Override
    public void delete(Task entity) {
        Task ent = findById(entity.getId());
        if(ent != null){
            tasks.remove(ent);
        }
    }

    public void save(Task entity){
        entity.setId(++count);
        tasks.add(entity);
    }
    
}


----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/dtos/Request.java

package dtos;

import com.google.gson.JsonObject;
import com.google.gson.annotations.Expose;

public class Request {
    @Expose private String command;
    @Expose private JsonObject data;

    public Request(String command, JsonObject data) {
        this.command = command;
        this.data = data;
    }

    public String getCommand() {
        return command;
    }
    public JsonObject getData() {
        return data;
    }

    public void setCommand(String command) {
        this.command = command;
    }

    public void setData(JsonObject data) {
        this.data = data;
    }
}



----------------------------------------------------------------------------------------------
task-manager/managerServer/src/main/java/model/Task.java

package model;

import com.google.gson.annotations.Expose;

public class Task {
    @Expose private int id;
    @Expose private String title;
    @Expose private String description;
    @Expose private String dueDate;
    @Expose private String priority;

    @Expose(serialize = false, deserialize = true)
    private TaskStage stage;

    public Task(){

    }

    public String getDueDate() {
        return dueDate;
    }

    public String getPriority() {
        return priority;
    }

    public void setDueDate(String dueDate) {
        this.dueDate = dueDate;
    }

    public void setPriority(String priority) {
        this.priority = priority;
    }

    public void setId(int id) {
        this.id = id;
    }
    public int getId() {
        return id;
    }
    public void setTitle(String name) {
        this.title = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public TaskStage getStage() {
        return stage;
    }

    public void setStage(TaskStage stage) {
        this.stage = stage;
    }

    public String getTitle() {
        return title;
    }

    @Override
    public String toString() {
        return id +" - " +title;
    }

}

----------------------------------------------------------------------------------------------
task-manager/managerServer/src/main/java/model/TaskStage.java

package model;

import java.util.ArrayList;
import java.util.List;

import com.google.gson.annotations.Expose;

public class TaskStage {
    @Expose private int id;
    @Expose private String name;
    @Expose private String description;
    @Expose private List<Task> tasks;

    public TaskStage(int id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    public TaskStage(){

    }

    public List<Task> getTasks() {
        if (tasks == null) {
            tasks = new ArrayList<>();
        }
        return tasks;
    }

    public void setTasks(List<Task> tasks) {
        this.tasks = tasks;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }
    public String getName() {
        return name;
    }
    public String getDescription() {
        return description;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    
}
----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/model/Task.java

package model;

import com.google.gson.annotations.Expose;

public class Task {
    @Expose private int id;
    @Expose private String title;
    @Expose private String description;
    @Expose private String dueDate;
    @Expose private String priority;

    @Expose(serialize = false, deserialize = true)
    private TaskStage stage;

    public Task(){

    }

    public String getDueDate() {
        return dueDate;
    }

    public String getPriority() {
        return priority;
    }

    public void setDueDate(String dueDate) {
        this.dueDate = dueDate;
    }

    public void setPriority(String priority) {
        this.priority = priority;
    }

    public void setId(int id) {
        this.id = id;
    }
    public int getId() {
        return id;
    }
    public void setTitle(String name) {
        this.title = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public TaskStage getStage() {
        return stage;
    }

    public void setStage(TaskStage stage) {
        this.stage = stage;
    }

    public String getTitle() {
        return title;
    }

    @Override
    public String toString() {
        return id +" - " +title;
    }

}

----------------------------------------------------------------------------------------------
task-manager/managerServer/src/main/java/model/TaskStage.java

package model;

import java.util.ArrayList;
import java.util.List;

import com.google.gson.annotations.Expose;

public class TaskStage {
    @Expose private int id;
    @Expose private String name;
    @Expose private String description;
    @Expose private List<Task> tasks;

    public TaskStage(int id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    public TaskStage(){

    }

    public List<Task> getTasks() {
        if (tasks == null) {
            tasks = new ArrayList<>();
        }
        return tasks;
    }

    public void setTasks(List<Task> tasks) {
        this.tasks = tasks;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }
    public String getName() {
        return name;
    }
    public String getDescription() {
        return description;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    
}
----------------------------------------------------------------------------------------------

task-manager/managerServer/src/main/java/services/TaskServices.java

package services;

import java.util.List;

import daos.Dao;
import daos.StageDao;
import daos.TaskMemDao;
import model.Task;
import model.TaskStage;

public class TaskServices {

    private Dao<Task,Integer> repository;
    private Dao<TaskStage,Integer> repositoryStage;

    public TaskServices(){
        repository = new TaskMemDao();
        repositoryStage = new StageDao();
    }
    
    public Task saveTask(Task t){
        TaskStage s = repositoryStage.findById(t.getStage().getId());
        s.getTasks().add(t);
        t.setStage(s);
        repository.save(t);
        return t;
    }

    public Task changeStage(int taskId, int stageId){
        Task t = repository.findById(taskId);
        TaskStage oldStage = t.getStage();
        oldStage.getTasks().remove(t);
        TaskStage s = repositoryStage.findById(stageId);
        s.getTasks().add(t);
        t.setStage(s);
        repository.update(t);

        return t;
    }

    public List<TaskStage> getTask(){
        return repositoryStage.findAll();
    }
}


----------------------------------------------------------------------------------------------


task-manager/managerServer/src/main/java/ManagerServer.java

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.List;
import java.util.Map;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;

import dtos.Request;
import model.Task;
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
        gson = new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create();
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
                Request rq = gson.fromJson(line, Request.class);
                JsonObject obj = rq.getData();
                String resp = null;
                try {
                    switch (rq.getCommand()) {
                        case "CREATE_TASK":
                            Task t = gson.fromJson(obj, Task.class);
                            t = services.saveTask(t);
                            resp = gson.toJson(t);
                            break;
                        case "UPDATE_TASK":
                            String taskId = obj.get("taskId").getAsString();
                            int stage = obj.get("stage").getAsInt();
                            t = services.changeStage(Integer.parseInt(taskId), stage);
                            resp = gson.toJson(t);
                            break;
                        case "GET_TASKS":
                            List<TaskStage> stages = services.getTask();
                            resp = gson.toJson(stages);
                            break;
                        case "HELLO":
                            Map<String,String> hello = Map.of("message", "Hello from Java server");
                            resp = gson.toJson(hello);
                            break;
                        default:
                            Map<String,String> error = Map.of("error", "No support operation");
                            resp = gson.toJson(error);
                            break;
                    }
                } catch (Exception e) {
                    Map<String,String> error = Map.of("error", e.getMessage());
                    resp = gson.toJson(error);
                    e.printStackTrace();
                }
                writer.write(resp+"\n");
                writer.flush();        
            }
        } catch (Exception e) {
            e.printStackTrace();
            sc.close();
        }
    }

}
----------------------------------------------------------------------------------------------
Adicional

task dao db:

package daos;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

import DBConfig.ConnectionManager;
import model.Task;
import model.TaskStage;

public class TaskDaoDB implements Dao<Task, Integer> {

    @Override
    public List<Task> findAll() {
        try {
            List<Task> tasks = new ArrayList<>();
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "SELECT t.id, t.title, t.description, t.stage_id, s.name AS stage_name " +
                           "FROM task t JOIN task_stage s ON t.stage_id = s.id";

            Statement statement = conn.createStatement();
            ResultSet result = statement.executeQuery(query);

            while (result.next()) {
                Task t = new Task();
                t.setId(result.getInt("id"));
                t.setTitle(result.getString("title"));
                t.setDescription(result.getString("description"));

                TaskStage stage = new TaskStage();
                stage.setId(result.getInt("stage_id"));
                stage.setName(result.getString("stage_name"));
                t.setStage(stage);

                tasks.add(t);
            }

            conn.close();
            return tasks;
        } catch (Exception e) {
            throw new RuntimeException("Error fetching all tasks", e);
        }
    }

    @Override
    public Task findById(Integer id) {
        try {
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "SELECT t.id, t.title, t.description, t.stage_id, s.name AS stage_name " +
                           "FROM task t JOIN task_stage s ON t.stage_id = s.id WHERE t.id = ?";

            PreparedStatement ps = conn.prepareStatement(query);
            ps.setInt(1, id);
            ResultSet result = ps.executeQuery();

            Task t = null;
            if (result.next()) {
                t = new Task();
                t.setId(result.getInt("id"));
                t.setTitle(result.getString("title"));
                t.setDescription(result.getString("description"));

                TaskStage stage = new TaskStage();
                stage.setId(result.getInt("stage_id"));
                stage.setName(result.getString("stage_name"));
                t.setStage(stage);
            }

            conn.close();
            return t;
        } catch (Exception e) {
            throw new RuntimeException("Error fetching task by id", e);
        }
    }

    @Override
    public void save(Task entity) {
        try {
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "INSERT INTO task (title, description, stage_id) VALUES (?, ?, ?)";

            PreparedStatement ps = conn.prepareStatement(query);
            ps.setString(1, entity.getTitle());
            ps.setString(2, entity.getDescription());
            ps.setInt(3, entity.getStage().getId());

            ps.executeUpdate();
            conn.close();
        } catch (Exception e) {
            throw new RuntimeException("Error saving task", e);
        }
    }

    @Override
    public Task update(Task entity) {
        try {
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "UPDATE task SET title = ?, description = ?, stage_id = ? WHERE id = ?";

            PreparedStatement ps = conn.prepareStatement(query);
            ps.setString(1, entity.getTitle());
            ps.setString(2, entity.getDescription());
            ps.setInt(3, entity.getStage().getId());
            ps.setInt(4, entity.getId());

            ps.executeUpdate();
            conn.close();

            return entity;
        } catch (Exception e) {
            throw new RuntimeException("Error updating task", e);
        }
    }

    @Override
    public void delete(Task entity) {
        try {
            Connection conn = ConnectionManager.getInstance().getConnection();
            String query = "DELETE FROM task WHERE id = ?";

            PreparedStatement ps = conn.prepareStatement(query);
            ps.setInt(1, entity.getId());

            ps.executeUpdate();
            conn.close();
        } catch (Exception e) {
            throw new RuntimeException("Error deleting task", e);
        }
    }
}

----------------------------------------------------------------------------------------------
task-manager/web-client/proxy/index.js

const express = require('express');
const net = require('net');
const cors = require('cors');


const socket = new net.Socket();
let connected = false;

socket.connect(5000, "127.0.0.1", () => {
    console.log("Conectado al servidor de Java");
    const message = {
        command: "HELLO",
        data: {}
    }
    socket.write(JSON.stringify(message) + "\n")
    socket.once("data", (data) => {
        console.log("Respuesta del servidor:", data.toString().trim());
    });
    connected = true;
})

const app = express();
app.use(cors());
const port = 3000;
app.use(express.json());

app.post('/tasks', (req, res) => {
    const body = req.body
    const backReq = {
        command: "CREATE_TASK",
        data: body
    }
    const bodyStr = JSON.stringify(backReq)
    if(connected){
        socket.write(bodyStr)
        socket.write("\n")
        socket.once("data", (data) => {
            const message = data.toString().trim();
            console.log("Respuesta del servidor:", message);
            try{
                res.json(JSON.parse(message));
            }catch(e){
                res.status(500).json({ error: "Error al procesar la respuesta del servidor" });
            }
        });
    }else{
        res.status(500).json({ error: "Socket no conectado" });
    }
});

app.put('/tasks/:id', (req, res) => {
    const body = req.body
    const taskId = req.params.id
    body.taskId = taskId
    const backReq = {
        command: "UPDATE_TASK",
        data: body
    }
    const bodyStr = JSON.stringify(backReq)
    if(connected){
        socket.write(bodyStr)
        socket.write("\n")
        socket.once("data", (data) => {
            const message = data.toString().trim();
            console.log("Respuesta del servidor:", message);
            try{
                res.json(JSON.parse(message));
            }catch(e){
                res.status(500).json({ error: "Error al procesar la respuesta del servidor" });
            }
        });
    }else{
        res.status(500).json({ error: "Socket no conectado" });
    }
});

app.get('/tasks', (req, res) => {
    if (connected) {
        const backReq = {
            command: "GET_TASKS",
            data: {}
        }
        const bodyStr = JSON.stringify(backReq)
        // Enviar el comando
        socket.write(bodyStr);
        socket.write("\n");

        socket.once("data", (data) => {
            const message = data.toString().trim();
            console.log("Respuesta del servidor:", message);
            try{
                res.json(JSON.parse(message));
            }catch(e){
                res.status(500).json({ error: "Error al procesar la respuesta del servidor" });
            }  
        });
    } else {
        res.status(500).json({ error: "Socket no conectado" });
    }
});

app.listen(port, () => {
    console.log(`API server running at http://localhost:${port}`);
});
----------------------------------------------------------------------------------------------
task-manager/web-client/src/assets/tasks.js

const tasks =[
    {
        "id": 1,
        "name": "To Do",
        "description": "Tasks that need to be done",
        "tasks": [
            {
                "id": 1,
                "title": "Buy groceries",
                "description": "Milk, Bread, Eggs, Butter",
                "dueDate": "2023-10-01",
                "priority": "High"
            },
            {
                "id": 2,
                "title": "Read a book",
                "description": "Finish reading 'The Great Gatsby'",
                "dueDate": "2023-10-05",
                "priority": "Medium"
            }
        ]
    },
    {
        "id": 2,
        "name": "In Progress",
        "description": "Tasks that are currently being worked on",
        "tasks": [
            {
                "id": 3,
                "title": "Develop feature X",
                "description": "Implement the new feature for the app",
                "dueDate": "2023-10-10",
                "priority": "High"
            }
        ]
    },
    {
        "id": 3,
        "name": "Completed",
        "description": "Tasks that have been completed",
        "tasks": [
            {
                "id": 4,
                "title": "Morning workout",
                "description": "30 minutes of cardio and strength training",
                "dueDate": "2023-09-30",
                "priority": "Low"
            }
        ]
    }
]

export default tasks;
----------------------------------------------------------------------------------------------
task-manager/web-client/src/components/ListTask.js

import createTaskElement from "./TaskCard.js";
const updateTask = (taskId, newStage, callback) => {
    fetch(`http://localhost:3000/tasks/${taskId}`, {
        method: "PUT",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({ stage: newStage })
    })
    .then(response => {
        if (!response.ok) {
            throw new Error("Server response was not ok");
        }
        return response.json()
    })
    .then(data => {
        if (callback ) callback(data);
    })
    .catch(error => {
        console.error("Error updating task:", error);
        alert("Error updating task: " + error.message);
        if (callback ) callback(null);
    });
}

const ListTask = (list) => {
    const container = document.createElement("div");
    container.className = "task-list-container";

    const title = document.createElement("h2")
    title.innerText = list.name
    container.appendChild(
        title
    )
    const description = document.createElement("p")
    description.innerText = list.description
    container.appendChild(
        description
    );
    const taskContainer = document.createElement("div");
    taskContainer.className = "task-items";
    container.appendChild(taskContainer);
    list.tasks?.forEach(task => {
        const taskElement = createTaskElement(task);
        taskContainer.appendChild(taskElement);
    });


    taskContainer.addEventListener("dragover", (e) => {
        e.preventDefault(); 
        taskContainer.classList.add("drag-over");
    });

    taskContainer.addEventListener("dragleave", () => {
        taskContainer.classList.remove("drag-over");
    });

    taskContainer.addEventListener("drop", (e) => {
        e.preventDefault();
        taskContainer.classList.remove("drag-over");

        const taskId = e.dataTransfer.getData("text/plain");
        console.log(`Dropped task: ${taskId} to list: ${list.id}`);

        // Aquí podrías mover el elemento visualmente
        const dragged = document.querySelector(".dragging");
        const referenceNode =  document.querySelector(".drag-over-sibling");
        updateTask(taskId, list.id, (updatedTask) => {
            if (!updatedTask) {
                if(referenceNode) referenceNode.classList.remove("drag-over-sibling");
                return;
            }
            if (referenceNode && dragged) {
                referenceNode.after(dragged);
                referenceNode.classList.remove("drag-over-sibling");
            } else if (dragged) {
                taskContainer.appendChild(dragged);
            }
        });

        // O actualizar el modelo de datos (list.tasks)
        // según tu lógica de negocio
    });


    return container;
}



export default ListTask;


----------------------------------------------------------------------------------------------
task-manager/web-client/src/components/TaskCard.js



function createTaskElement(task) {
    const li = document.createElement("div");
    li.className = "task-item";
    li.draggable = true; 

    const title = document.createElement("span");
    title.textContent = `${task.title}`; 

    const description = document.createElement("span");
    description.textContent = ` - ${task.description}`;

    const createdAt = document.createElement("span");
    createdAt.textContent = ` (Due: ${task.dueDate})`;

    const delBtn = document.createElement("button");
    delBtn.textContent = "Delete";
    delBtn.onclick = () => {
        li.remove();
    };

    li.appendChild(title);
    li.appendChild(description);
    li.appendChild(createdAt);

    li.appendChild(delBtn);

    li.addEventListener("dragstart", (e) => {
        e.dataTransfer.setData("text/plain", task.id); // o JSON.stringify(task)
        li.classList.add("dragging");
    });

    li.addEventListener("dragend", () => {
        li.classList.remove("dragging");
    });

    li.addEventListener("dragover", (e) => {
        e.preventDefault(); 
        li.classList.add("drag-over-sibling");
    });

    
    li.addEventListener("dragleave", () => {
        li.classList.remove("drag-over-sibling");
    });
    

    return li;
}

export default createTaskElement;

----------------------------------------------------------------------------------------------
task-manager/web-client/src/delegate/taskDelegate.js


const getTask = ()=>{
    
}


----------------------------------------------------------------------------------------------
task-manager/web-client/src/pages/BoardTask.js


import createTaskElement from "../components/TaskCard.js";
import ListTask from "../components/ListTask.js";

const onAddTask = (e, container, title, description, dueDate) => {
    const element = {
        "id": 0,
        "title": title,
        "description": description,
        "dueDate": dueDate,
        "priority": "High",
        "stage": {
            "id": 1
        }
    }
    fetch("http://localhost:3000/tasks", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(element)
    })
    .then(response => {
        if (!response.ok) {
            throw new Error("Server response was not ok");
        }
        return response.json();
    })
    .then(data => {
        const newTask = createTaskElement(data)
        container.appendChild(newTask)
        console.log(data)
    })
    .catch(error => {
        console.error("Error creating task:", error);
        alert("Error creating task: " + error.message);
    });
}

const BoardTask = (tasks) => {
    const comp = document.createElement("div");
    comp.className = "board-task";
    const h1 = document.createElement("h1");
    h1.textContent = "Task Manager";

    const input = document.createElement("input");
    input.type = "text";
    input.id = "task-name"
    input.placeholder = "Nueva tarea...";

    const description = document.createElement("input");
    description.type = "text";
    description.id = "task-description"
    description.placeholder = "Descripcion...";

    const dueDate = document.createElement("input");
    dueDate.type = "date";
    dueDate.id = "task-due-date"
    dueDate.placeholder = "Due Date...";
    
    const lists = document.createElement("div");
    lists.className = "group-lists";

    const button = document.createElement("button");
    button.textContent = "Guardar";
    
    tasks.forEach(task => {
        const listElement = ListTask(task);
        lists.appendChild(listElement);
    });

    button.onclick = (e) => onAddTask(e, lists.firstElementChild.getElementsByClassName("task-items")[0], input.value, description.value, dueDate.value)

    comp.appendChild(input)
    comp.appendChild(description)
    comp.appendChild(dueDate)
    comp.appendChild(button)
    comp.appendChild(lists)

    return comp;
}



export default BoardTask;

----------------------------------------------------------------------------------------------

task-manager/web-client/src/index.js

import BoardTask from "./pages/BoardTask.js";

const getTasks = () => {
    return fetch("http://localhost:3000/tasks")
    .then(response => {
        if (!response.ok) {
            throw new Error("Server response was not ok");
        }
        return response.json();
    })
    .then(data => {
        return data;
    })
    .catch(error => {
        console.error("Error fetching tasks:", error);
        alert("Error fetching tasks: " + error.message);
        return [];
    });
}

function renderApp() {
    const app = document.getElementById("app");
    const h1 = document.createElement("h1");
    h1.textContent = "Task Manager";
    app.appendChild(h1);

    getTasks().then(tasks => {
        const board = BoardTask(tasks);
        app.appendChild(board);
    });
}

renderApp();

----------------------------------------------------------------------------------------------

task-manager/web-client/app.css

.task-item{
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    background: #fdfdfd;
    border: 1px solid #ddd;
    padding: 10px;
    margin-bottom: 6px;
    border-radius: 8px;
    cursor: grab;
    transition: 'transform 0.2s, box-shadow 0.2s';
}

.group-lists{
    display: flex;
    gap: 16px;
    padding: 16px;
    flex-direction: row;
    width: 80%;
    align-items: flex-start;
    
}

.task-list-container{
    background-color: #f4f4f4;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    width: 33%;
    min-height: 300px;
}

.board-task{
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family: Arial, sans-serif;
}

input{
    width: 500px;
    padding: 8px;
    margin: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button{
    padding: 8px;
    margin: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
}


.task-item.dragging {
  opacity: 0.5;
  transform: scale(1.05);
  box-shadow: 0 0 8px rgba(0,0,0,0.3);
}

.task-items.drag-over {
  background: #f0f8ff;
  border: 2px dashed #2196f3;
  transition: 'background 0.3s';
}

.task-items{
    min-height: 200px;
}
.drag-over-sibling{
    border-bottom: 4px dashed #21f39c;
}

----------------------------------------------------------------------------------------------
task-manager/web-client/index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="app.css">
    <title>Document</title>
</head>
<body>
    <div id="app"></div>
    <script src="src/index.js" type="module"></script>
</body>
</html>
----------------------------------------------------------------------------------------------

task-manager/web-client/package-lock.json

{
  "name": "web-client",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "web-client",
      "version": "1.0.0",
      "license": "ISC",
      "dependencies": {
        "cors": "^2.8.5",
        "express": "^5.1.0"
      }
    },
    "node_modules/accepts": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/accepts/-/accepts-2.0.0.tgz",
      "integrity": "sha512-5cvg6CtKwfgdmVqY1WIiXKc3Q1bkRqGLi+2W/6ao+6Y7gu/RCwRuAhGEzh5B4KlszSuTLgZYuqFqo5bImjNKng==",
      "license": "MIT",
      "dependencies": {
        "mime-types": "^3.0.0",
        "negotiator": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/body-parser": {
      "version": "2.2.0",
      "resolved": "https://registry.npmjs.org/body-parser/-/body-parser-2.2.0.tgz",
      "integrity": "sha512-02qvAaxv8tp7fBa/mw1ga98OGm+eCbqzJOKoRt70sLmfEEi+jyBYVTDGfCL/k06/4EMk/z01gCe7HoCH/f2LTg==",
      "license": "MIT",
      "dependencies": {
        "bytes": "^3.1.2",
        "content-type": "^1.0.5",
        "debug": "^4.4.0",
        "http-errors": "^2.0.0",
        "iconv-lite": "^0.6.3",
        "on-finished": "^2.4.1",
        "qs": "^6.14.0",
        "raw-body": "^3.0.0",
        "type-is": "^2.0.0"
      },
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/bytes": {
      "version": "3.1.2",
      "resolved": "https://registry.npmjs.org/bytes/-/bytes-3.1.2.tgz",
      "integrity": "sha512-/Nf7TyzTx6S3yRJObOAV7956r8cr2+Oj8AC5dt8wSP3BQAoeX58NoHyCU8P8zGkNXStjTSi6fzO6F0pBdcYbEg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/call-bind-apply-helpers": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/call-bind-apply-helpers/-/call-bind-apply-helpers-1.0.2.tgz",
      "integrity": "sha512-Sp1ablJ0ivDkSzjcaJdxEunN5/XvksFJ2sMBFfq6x0ryhQV/2b/KwFe21cMpmHtPOSij8K99/wSfoEuTObmuMQ==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/call-bound": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/call-bound/-/call-bound-1.0.4.tgz",
      "integrity": "sha512-+ys997U96po4Kx/ABpBCqhA9EuxJaQWDQg7295H4hBphv3IZg0boBKuwYpt4YXp6MZ5AmZQnU/tyMTlRpaSejg==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "get-intrinsic": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/content-disposition": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/content-disposition/-/content-disposition-1.0.0.tgz",
      "integrity": "sha512-Au9nRL8VNUut/XSzbQA38+M78dzP4D+eqg3gfJHMIHHYa3bg067xj1KxMUWj+VULbiZMowKngFFbKczUrNJ1mg==",
      "license": "MIT",
      "dependencies": {
        "safe-buffer": "5.2.1"
      },
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/content-type": {
      "version": "1.0.5",
      "resolved": "https://registry.npmjs.org/content-type/-/content-type-1.0.5.tgz",
      "integrity": "sha512-nTjqfcBFEipKdXCv4YDQWCfmcLZKm81ldF0pAopTvyrFGVbcR6P/VAAd5G7N+0tTr8QqiU0tFadD6FK4NtJwOA==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/cookie": {
      "version": "0.7.2",
      "resolved": "https://registry.npmjs.org/cookie/-/cookie-0.7.2.tgz",
      "integrity": "sha512-yki5XnKuf750l50uGTllt6kKILY4nQ1eNIQatoXEByZ5dWgnKqbnqmTrBE5B4N7lrMJKQ2ytWMiTO2o0v6Ew/w==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/cookie-signature": {
      "version": "1.2.2",
      "resolved": "https://registry.npmjs.org/cookie-signature/-/cookie-signature-1.2.2.tgz",
      "integrity": "sha512-D76uU73ulSXrD1UXF4KE2TMxVVwhsnCgfAyTg9k8P6KGZjlXKrOLe4dJQKI3Bxi5wjesZoFXJWElNWBjPZMbhg==",
      "license": "MIT",
      "engines": {
        "node": ">=6.6.0"
      }
    },
    "node_modules/cors": {
      "version": "2.8.5",
      "resolved": "https://registry.npmjs.org/cors/-/cors-2.8.5.tgz",
      "integrity": "sha512-KIHbLJqu73RGr/hnbrO9uBeixNGuvSQjul/jdFvS/KFSIH1hWVd1ng7zOHx+YrEfInLG7q4n6GHQ9cDtxv/P6g==",
      "license": "MIT",
      "dependencies": {
        "object-assign": "^4",
        "vary": "^1"
      },
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/debug": {
      "version": "4.4.3",
      "resolved": "https://registry.npmjs.org/debug/-/debug-4.4.3.tgz",
      "integrity": "sha512-RGwwWnwQvkVfavKVt22FGLw+xYSdzARwm0ru6DhTVA3umU5hZc28V3kO4stgYryrTlLpuvgI9GiijltAjNbcqA==",
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.3"
      },
      "engines": {
        "node": ">=6.0"
      },
      "peerDependenciesMeta": {
        "supports-color": {
          "optional": true
        }
      }
    },
    "node_modules/depd": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/depd/-/depd-2.0.0.tgz",
      "integrity": "sha512-g7nH6P6dyDioJogAAGprGpCtVImJhpPk/roCzdb3fIh61/s/nPsfR6onyMwkCAR/OlC3yBC0lESvUoQEAssIrw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/dunder-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/dunder-proto/-/dunder-proto-1.0.1.tgz",
      "integrity": "sha512-KIN/nDJBQRcXw0MLVhZE9iQHmG68qAVIBg9CqmUYjmQIhgij9U5MFvrqkUL5FbtyyzZuOeOt0zdeRe4UY7ct+A==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.1",
        "es-errors": "^1.3.0",
        "gopd": "^1.2.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/ee-first": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/ee-first/-/ee-first-1.1.1.tgz",
      "integrity": "sha512-WMwm9LhRUo+WUaRN+vRuETqG89IgZphVSNkdFgeb6sS/E4OrDIN7t48CAewSHXc6C8lefD8KKfr5vY61brQlow==",
      "license": "MIT"
    },
    "node_modules/encodeurl": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/encodeurl/-/encodeurl-2.0.0.tgz",
      "integrity": "sha512-Q0n9HRi4m6JuGIV1eFlmvJB7ZEVxu93IrMyiMsGC0lrMJMWzRgx6WGquyfQgZVb31vhGgXnfmPNNXmxnOkRBrg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/es-define-property": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/es-define-property/-/es-define-property-1.0.1.tgz",
      "integrity": "sha512-e3nRfgfUZ4rNGL232gUgX06QNyyez04KdjFrF+LTRoOXmrOgFKDg4BCdsjW8EnT69eqdYGmRpJwiPVYNrCaW3g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-errors": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/es-errors/-/es-errors-1.3.0.tgz",
      "integrity": "sha512-Zf5H2Kxt2xjTvbJvP2ZWLEICxA6j+hAmMzIlypy4xcBg1vKVnx89Wy0GbS+kf5cwCVFFzdCFh2XSCFNULS6csw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-object-atoms": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/es-object-atoms/-/es-object-atoms-1.1.1.tgz",
      "integrity": "sha512-FGgH2h8zKNim9ljj7dankFPcICIK9Cp5bm+c2gQSYePhpaG5+esrLODihIorn+Pe6FGJzWhXQotPv73jTaldXA==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/escape-html": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/escape-html/-/escape-html-1.0.3.tgz",
      "integrity": "sha512-NiSupZ4OeuGwr68lGIeym/ksIZMJodUGOSCZ/FSnTxcrekbvqrgdUxlJOMpijaKZVjAJrWrGs/6Jy8OMuyj9ow==",
      "license": "MIT"
    },
    "node_modules/etag": {
      "version": "1.8.1",
      "resolved": "https://registry.npmjs.org/etag/-/etag-1.8.1.tgz",
      "integrity": "sha512-aIL5Fx7mawVa300al2BnEE4iNvo1qETxLrPI/o05L7z6go7fCw1J6EQmbK4FmJ2AS7kgVF/KEZWufBfdClMcPg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/express": {
      "version": "5.1.0",
      "resolved": "https://registry.npmjs.org/express/-/express-5.1.0.tgz",
      "integrity": "sha512-DT9ck5YIRU+8GYzzU5kT3eHGA5iL+1Zd0EutOmTE9Dtk+Tvuzd23VBU+ec7HPNSTxXYO55gPV/hq4pSBJDjFpA==",
      "license": "MIT",
      "dependencies": {
        "accepts": "^2.0.0",
        "body-parser": "^2.2.0",
        "content-disposition": "^1.0.0",
        "content-type": "^1.0.5",
        "cookie": "^0.7.1",
        "cookie-signature": "^1.2.1",
        "debug": "^4.4.0",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "etag": "^1.8.1",
        "finalhandler": "^2.1.0",
        "fresh": "^2.0.0",
        "http-errors": "^2.0.0",
        "merge-descriptors": "^2.0.0",
        "mime-types": "^3.0.0",
        "on-finished": "^2.4.1",
        "once": "^1.4.0",
        "parseurl": "^1.3.3",
        "proxy-addr": "^2.0.7",
        "qs": "^6.14.0",
        "range-parser": "^1.2.1",
        "router": "^2.2.0",
        "send": "^1.1.0",
        "serve-static": "^2.2.0",
        "statuses": "^2.0.1",
        "type-is": "^2.0.1",
        "vary": "^1.1.2"
      },
      "engines": {
        "node": ">= 18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/finalhandler": {
      "version": "2.1.0",
      "resolved": "https://registry.npmjs.org/finalhandler/-/finalhandler-2.1.0.tgz",
      "integrity": "sha512-/t88Ty3d5JWQbWYgaOGCCYfXRwV1+be02WqYYlL6h0lEiUAMPM8o8qKGO01YIkOHzka2up08wvgYD0mDiI+q3Q==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.4.0",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "on-finished": "^2.4.1",
        "parseurl": "^1.3.3",
        "statuses": "^2.0.1"
      },
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/forwarded": {
      "version": "0.2.0",
      "resolved": "https://registry.npmjs.org/forwarded/-/forwarded-0.2.0.tgz",
      "integrity": "sha512-buRG0fpBtRHSTCOASe6hD258tEubFoRLb4ZNA6NxMVHNw2gOcwHo9wyablzMzOA5z9xA9L1KNjk/Nt6MT9aYow==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/fresh": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/fresh/-/fresh-2.0.0.tgz",
      "integrity": "sha512-Rx/WycZ60HOaqLKAi6cHRKKI7zxWbJ31MhntmtwMoaTeF7XFH9hhBp8vITaMidfljRQ6eYWCKkaTK+ykVJHP2A==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/function-bind": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/function-bind/-/function-bind-1.1.2.tgz",
      "integrity": "sha512-7XHNxH7qX9xG5mIwxkhumTox/MIRNcOgDrxWsMt2pAr23WHp6MrRlN7FBSFpCpr+oVO0F744iUgR82nJMfG2SA==",
      "license": "MIT",
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/get-intrinsic": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/get-intrinsic/-/get-intrinsic-1.3.0.tgz",
      "integrity": "sha512-9fSjSaos/fRIVIp+xSJlE6lfwhES7LNtKaCBIamHsjr2na1BiABJPo0mOjjz8GJDURarmCPGqaiVg5mfjb98CQ==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.1",
        "function-bind": "^1.1.2",
        "get-proto": "^1.0.1",
        "gopd": "^1.2.0",
        "has-symbols": "^1.1.0",
        "hasown": "^2.0.2",
        "math-intrinsics": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/get-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/get-proto/-/get-proto-1.0.1.tgz",
      "integrity": "sha512-sTSfBjoXBp89JvIKIefqw7U2CCebsc74kiY6awiGogKtoSGbgjYE/G/+l9sF3MWFPNc9IcoOC4ODfKHfxFmp0g==",
      "license": "MIT",
      "dependencies": {
        "dunder-proto": "^1.0.1",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/gopd": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/gopd/-/gopd-1.2.0.tgz",
      "integrity": "sha512-ZUKRh6/kUFoAiTAtTYPZJ3hw9wNxx+BIBOijnlG9PnrJsCcSjs1wyyD6vJpaYtgnzDrKYRSqf3OO6Rfa93xsRg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-symbols": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/has-symbols/-/has-symbols-1.1.0.tgz",
      "integrity": "sha512-1cDNdwJ2Jaohmb3sg4OmKaMBwuC48sYni5HUw2DvsC8LjGTLK9h+eb1X6RyuOHe4hT0ULCW68iomhjUoKUqlPQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/hasown": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/hasown/-/hasown-2.0.2.tgz",
      "integrity": "sha512-0hJU9SCPvmMzIBdZFqNPXWa6dqh7WdH0cII9y+CyS8rG3nL48Bclra9HmKhVVUHyPWNH5Y7xDwAB7bfgSjkUMQ==",
      "license": "MIT",
      "dependencies": {
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/http-errors": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/http-errors/-/http-errors-2.0.0.tgz",
      "integrity": "sha512-FtwrG/euBzaEjYeRqOgly7G0qviiXoJWnvEH2Z1plBdXgbyjv34pHTSb9zoeHMyDy33+DWy5Wt9Wo+TURtOYSQ==",
      "license": "MIT",
      "dependencies": {
        "depd": "2.0.0",
        "inherits": "2.0.4",
        "setprototypeof": "1.2.0",
        "statuses": "2.0.1",
        "toidentifier": "1.0.1"
      },
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/http-errors/node_modules/statuses": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/statuses/-/statuses-2.0.1.tgz",
      "integrity": "sha512-RwNA9Z/7PrK06rYLIzFMlaF+l73iwpzsqRIFgbMLbTcLD6cOao82TaWefPXQvB2fOC4AjuYSEndS7N/mTCbkdQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/iconv-lite": {
      "version": "0.6.3",
      "resolved": "https://registry.npmjs.org/iconv-lite/-/iconv-lite-0.6.3.tgz",
      "integrity": "sha512-4fCk79wshMdzMp2rH06qWrJE4iolqLhCUH+OiuIgU++RB0+94NlDL81atO7GX55uUKueo0txHNtvEyI6D7WdMw==",
      "license": "MIT",
      "dependencies": {
        "safer-buffer": ">= 2.1.2 < 3.0.0"
      },
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/inherits": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/inherits/-/inherits-2.0.4.tgz",
      "integrity": "sha512-k/vGaX4/Yla3WzyMCvTQOXYeIHvqOKtnqBduzTHpzpQZzAskKMhZ2K+EnBiSM9zGSoIFeMpXKxa4dYeZIQqewQ==",
      "license": "ISC"
    },
    "node_modules/ipaddr.js": {
      "version": "1.9.1",
      "resolved": "https://registry.npmjs.org/ipaddr.js/-/ipaddr.js-1.9.1.tgz",
      "integrity": "sha512-0KI/607xoxSToH7GjN1FfSbLoU0+btTicjsQSWQlh/hZykN8KpmMf7uYwPW3R+akZ6R/w18ZlXSHBYXiYUPO3g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/is-promise": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/is-promise/-/is-promise-4.0.0.tgz",
      "integrity": "sha512-hvpoI6korhJMnej285dSg6nu1+e6uxs7zG3BYAm5byqDsgJNWwxzM6z6iZiAgQR4TJ30JmBTOwqZUw3WlyH3AQ==",
      "license": "MIT"
    },
    "node_modules/math-intrinsics": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/math-intrinsics/-/math-intrinsics-1.1.0.tgz",
      "integrity": "sha512-/IXtbwEk5HTPyEwyKX6hGkYXxM9nbj64B+ilVJnC/R6B0pH5G4V3b0pVbL7DBj4tkhBAppbQUlf6F6Xl9LHu1g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/media-typer": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/media-typer/-/media-typer-1.1.0.tgz",
      "integrity": "sha512-aisnrDP4GNe06UcKFnV5bfMNPBUw4jsLGaWwWfnH3v02GnBuXX2MCVn5RbrWo0j3pczUilYblq7fQ7Nw2t5XKw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/merge-descriptors": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/merge-descriptors/-/merge-descriptors-2.0.0.tgz",
      "integrity": "sha512-Snk314V5ayFLhp3fkUREub6WtjBfPdCPY1Ln8/8munuLuiYhsABgBVWsozAG+MWMbVEvcdcpbi9R7ww22l9Q3g==",
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/mime-db": {
      "version": "1.54.0",
      "resolved": "https://registry.npmjs.org/mime-db/-/mime-db-1.54.0.tgz",
      "integrity": "sha512-aU5EJuIN2WDemCcAp2vFBfp/m4EAhWJnUNSSw0ixs7/kXbd6Pg64EmwJkNdFhB8aWt1sH2CTXrLxo/iAGV3oPQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/mime-types": {
      "version": "3.0.1",
      "resolved": "https://registry.npmjs.org/mime-types/-/mime-types-3.0.1.tgz",
      "integrity": "sha512-xRc4oEhT6eaBpU1XF7AjpOFD+xQmXNB5OVKwp4tqCuBpHLS/ZbBDrc07mYTDqVMg6PfxUjjNp85O6Cd2Z/5HWA==",
      "license": "MIT",
      "dependencies": {
        "mime-db": "^1.54.0"
      },
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/ms": {
      "version": "2.1.3",
      "resolved": "https://registry.npmjs.org/ms/-/ms-2.1.3.tgz",
      "integrity": "sha512-6FlzubTLZG3J2a/NVCAleEhjzq5oxgHyaCU9yYXvcLsvoVaHJq/s5xXI6/XXP6tz7R9xAOtHnSO/tXtF3WRTlA==",
      "license": "MIT"
    },
    "node_modules/negotiator": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/negotiator/-/negotiator-1.0.0.tgz",
      "integrity": "sha512-8Ofs/AUQh8MaEcrlq5xOX0CQ9ypTF5dl78mjlMNfOK08fzpgTHQRQPBxcPlEtIw0yRpws+Zo/3r+5WRby7u3Gg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/object-assign": {
      "version": "4.1.1",
      "resolved": "https://registry.npmjs.org/object-assign/-/object-assign-4.1.1.tgz",
      "integrity": "sha512-rJgTQnkUnH1sFw8yT6VSU3zD3sWmu6sZhIseY8VX+GRu3P6F7Fu+JNDoXfklElbLJSnc3FUQHVe4cU5hj+BcUg==",
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/object-inspect": {
      "version": "1.13.4",
      "resolved": "https://registry.npmjs.org/object-inspect/-/object-inspect-1.13.4.tgz",
      "integrity": "sha512-W67iLl4J2EXEGTbfeHCffrjDfitvLANg0UlX3wFUUSTx92KXRFegMHUVgSqE+wvhAbi4WqjGg9czysTV2Epbew==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/on-finished": {
      "version": "2.4.1",
      "resolved": "https://registry.npmjs.org/on-finished/-/on-finished-2.4.1.tgz",
      "integrity": "sha512-oVlzkg3ENAhCk2zdv7IJwd/QUD4z2RxRwpkcGY8psCVcCYZNq4wYnVWALHM+brtuJjePWiYF/ClmuDr8Ch5+kg==",
      "license": "MIT",
      "dependencies": {
        "ee-first": "1.1.1"
      },
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/once": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/once/-/once-1.4.0.tgz",
      "integrity": "sha512-lNaJgI+2Q5URQBkccEKHTQOPaXdUxnZZElQTZY0MFUAuaEqe1E+Nyvgdz/aIyNi6Z9MzO5dv1H8n58/GELp3+w==",
      "license": "ISC",
      "dependencies": {
        "wrappy": "1"
      }
    },
    "node_modules/parseurl": {
      "version": "1.3.3",
      "resolved": "https://registry.npmjs.org/parseurl/-/parseurl-1.3.3.tgz",
      "integrity": "sha512-CiyeOxFT/JZyN5m0z9PfXw4SCBJ6Sygz1Dpl0wqjlhDEGGBP1GnsUVEL0p63hoG1fcj3fHynXi9NYO4nWOL+qQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/path-to-regexp": {
      "version": "8.3.0",
      "resolved": "https://registry.npmjs.org/path-to-regexp/-/path-to-regexp-8.3.0.tgz",
      "integrity": "sha512-7jdwVIRtsP8MYpdXSwOS0YdD0Du+qOoF/AEPIt88PcCFrZCzx41oxku1jD88hZBwbNUIEfpqvuhjFaMAqMTWnA==",
      "license": "MIT",
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/proxy-addr": {
      "version": "2.0.7",
      "resolved": "https://registry.npmjs.org/proxy-addr/-/proxy-addr-2.0.7.tgz",
      "integrity": "sha512-llQsMLSUDUPT44jdrU/O37qlnifitDP+ZwrmmZcoSKyLKvtZxpyV0n2/bD/N4tBAAZ/gJEdZU7KMraoK1+XYAg==",
      "license": "MIT",
      "dependencies": {
        "forwarded": "0.2.0",
        "ipaddr.js": "1.9.1"
      },
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/qs": {
      "version": "6.14.0",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.14.0.tgz",
      "integrity": "sha512-YWWTjgABSKcvs/nWBi9PycY/JiPJqOD4JA6o9Sej2AtvSGarXxKC3OQSk4pAarbdQlKAh5D4FCQkJNkW+GAn3w==",
      "license": "BSD-3-Clause",
      "dependencies": {
        "side-channel": "^1.1.0"
      },
      "engines": {
        "node": ">=0.6"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/range-parser": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/range-parser/-/range-parser-1.2.1.tgz",
      "integrity": "sha512-Hrgsx+orqoygnmhFbKaHE6c296J+HTAQXoxEF6gNupROmmGJRoyzfG3ccAveqCBrwr/2yxQ5BVd/GTl5agOwSg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/raw-body": {
      "version": "3.0.1",
      "resolved": "https://registry.npmjs.org/raw-body/-/raw-body-3.0.1.tgz",
      "integrity": "sha512-9G8cA+tuMS75+6G/TzW8OtLzmBDMo8p1JRxN5AZ+LAp8uxGA8V8GZm4GQ4/N5QNQEnLmg6SS7wyuSmbKepiKqA==",
      "license": "MIT",
      "dependencies": {
        "bytes": "3.1.2",
        "http-errors": "2.0.0",
        "iconv-lite": "0.7.0",
        "unpipe": "1.0.0"
      },
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/raw-body/node_modules/iconv-lite": {
      "version": "0.7.0",
      "resolved": "https://registry.npmjs.org/iconv-lite/-/iconv-lite-0.7.0.tgz",
      "integrity": "sha512-cf6L2Ds3h57VVmkZe+Pn+5APsT7FpqJtEhhieDCvrE2MK5Qk9MyffgQyuxQTm6BChfeZNtcOLHp9IcWRVcIcBQ==",
      "license": "MIT",
      "dependencies": {
        "safer-buffer": ">= 2.1.2 < 3.0.0"
      },
      "engines": {
        "node": ">=0.10.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/router": {
      "version": "2.2.0",
      "resolved": "https://registry.npmjs.org/router/-/router-2.2.0.tgz",
      "integrity": "sha512-nLTrUKm2UyiL7rlhapu/Zl45FwNgkZGaCpZbIHajDYgwlJCOzLSk+cIPAnsEqV955GjILJnKbdQC1nVPz+gAYQ==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.4.0",
        "depd": "^2.0.0",
        "is-promise": "^4.0.0",
        "parseurl": "^1.3.3",
        "path-to-regexp": "^8.0.0"
      },
      "engines": {
        "node": ">= 18"
      }
    },
    "node_modules/safe-buffer": {
      "version": "5.2.1",
      "resolved": "https://registry.npmjs.org/safe-buffer/-/safe-buffer-5.2.1.tgz",
      "integrity": "sha512-rp3So07KcdmmKbGvgaNxQSJr7bGVSVk5S9Eq1F+ppbRo70+YeaDxkw5Dd8NPN+GD6bjnYm2VuPuCXmpuYvmCXQ==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT"
    },
    "node_modules/safer-buffer": {
      "version": "2.1.2",
      "resolved": "https://registry.npmjs.org/safer-buffer/-/safer-buffer-2.1.2.tgz",
      "integrity": "sha512-YZo3K82SD7Riyi0E1EQPojLz7kpepnSQI9IyPbHHg1XXXevb5dJI7tpyN2ADxGcQbHG7vcyRHk0cbwqcQriUtg==",
      "license": "MIT"
    },
    "node_modules/send": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/send/-/send-1.2.0.tgz",
      "integrity": "sha512-uaW0WwXKpL9blXE2o0bRhoL2EGXIrZxQ2ZQ4mgcfoBxdFmQold+qWsD2jLrfZ0trjKL6vOw0j//eAwcALFjKSw==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.3.5",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "etag": "^1.8.1",
        "fresh": "^2.0.0",
        "http-errors": "^2.0.0",
        "mime-types": "^3.0.1",
        "ms": "^2.1.3",
        "on-finished": "^2.4.1",
        "range-parser": "^1.2.1",
        "statuses": "^2.0.1"
      },
      "engines": {
        "node": ">= 18"
      }
    },
    "node_modules/serve-static": {
      "version": "2.2.0",
      "resolved": "https://registry.npmjs.org/serve-static/-/serve-static-2.2.0.tgz",
      "integrity": "sha512-61g9pCh0Vnh7IutZjtLGGpTA355+OPn2TyDv/6ivP2h/AdAVX9azsoxmg2/M6nZeQZNYBEwIcsne1mJd9oQItQ==",
      "license": "MIT",
      "dependencies": {
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "parseurl": "^1.3.3",
        "send": "^1.2.0"
      },
      "engines": {
        "node": ">= 18"
      }
    },
    "node_modules/setprototypeof": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/setprototypeof/-/setprototypeof-1.2.0.tgz",
      "integrity": "sha512-E5LDX7Wrp85Kil5bhZv46j8jOeboKq5JMmYM3gVGdGH8xFpPWXUMsNrlODCrkoxMEeNi/XZIwuRvY4XNwYMJpw==",
      "license": "ISC"
    },
    "node_modules/side-channel": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/side-channel/-/side-channel-1.1.0.tgz",
      "integrity": "sha512-ZX99e6tRweoUXqR+VBrslhda51Nh5MTQwou5tnUDgbtyM0dBgmhEDtWGP/xbKn6hqfPRHujUNwz5fy/wbbhnpw==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.3",
        "side-channel-list": "^1.0.0",
        "side-channel-map": "^1.0.1",
        "side-channel-weakmap": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-list": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/side-channel-list/-/side-channel-list-1.0.0.tgz",
      "integrity": "sha512-FCLHtRD/gnpCiCHEiJLOwdmFP+wzCmDEkc9y7NsYxeF4u7Btsn1ZuwgwJGxImImHicJArLP4R0yX4c2KCrMrTA==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-map": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/side-channel-map/-/side-channel-map-1.0.1.tgz",
      "integrity": "sha512-VCjCNfgMsby3tTdo02nbjtM/ewra6jPHmpThenkTYh8pG9ucZ/1P8So4u4FGBek/BjpOVsDCMoLA/iuBKIFXRA==",
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-weakmap": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/side-channel-weakmap/-/side-channel-weakmap-1.0.2.tgz",
      "integrity": "sha512-WPS/HvHQTYnHisLo9McqBHOJk2FkHO/tlpvldyrnem4aeQp4hai3gythswg6p01oSoTl58rcpiFAjF2br2Ak2A==",
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3",
        "side-channel-map": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/statuses": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/statuses/-/statuses-2.0.2.tgz",
      "integrity": "sha512-DvEy55V3DB7uknRo+4iOGT5fP1slR8wQohVdknigZPMpMstaKJQWhwiYBACJE3Ul2pTnATihhBYnRhZQHGBiRw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/toidentifier": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/toidentifier/-/toidentifier-1.0.1.tgz",
      "integrity": "sha512-o5sSPKEkg/DIQNmH43V0/uerLrpzVedkUh8tGNvaeXpfpuwjKenlSox/2O/BTlZUtEe+JG7s5YhEz608PlAHRA==",
      "license": "MIT",
      "engines": {
        "node": ">=0.6"
      }
    },
    "node_modules/type-is": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/type-is/-/type-is-2.0.1.tgz",
      "integrity": "sha512-OZs6gsjF4vMp32qrCbiVSkrFmXtG/AZhY3t0iAMrMBiAZyV9oALtXO8hsrHbMXF9x6L3grlFuwW2oAz7cav+Gw==",
      "license": "MIT",
      "dependencies": {
        "content-type": "^1.0.5",
        "media-typer": "^1.1.0",
        "mime-types": "^3.0.0"
      },
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/unpipe": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/unpipe/-/unpipe-1.0.0.tgz",
      "integrity": "sha512-pjy2bYhSsufwWlKwPc+l3cN7+wuJlK6uz0YdJEOlQDbl6jo/YlPi4mb8agUkVC8BF7V8NuzeyPNqRksA3hztKQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/vary": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/vary/-/vary-1.1.2.tgz",
      "integrity": "sha512-BNGbWLfd0eUPabhkXUVm0j8uuvREyTh5ovRa/dyow/BqAbZJyC+5fU+IzQOzmAKzYqYRAISoRhdQr3eIZ/PXqg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/wrappy": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/wrappy/-/wrappy-1.0.2.tgz",
      "integrity": "sha512-l4Sp/DRseor9wL6EvV2+TuQn63dMkPjZ/sp9XkghTEbV9KlPS1xUsZ3u7/IQO4wxtcFB4bgpQPRcR3QCvezPcQ==",
      "license": "ISC"
    }
  }
}
----------------------------------------------------------------------------------------------

task-manager/web-client/package.json

{
  "name": "web-client",
  "version": "1.0.0",
  "description": "Task-manager",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "AMB",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^5.1.0"
  }
}
----------------------------------------------------------------------------------------------

task-manager/ReadMe.md


# Taller: Servidor de Gestión de Tareas con ServerSocket

## Objetivo
Desarrollar un **servidor TCP** en Java utilizando `ServerSocket` que gestione tareas mediante un protocolo de mensajes en formato **JSON**.  
El servidor debe aplicar el **patrón DAO** para el manejo de los datos (en memoria o con JDBC).

---

## Descripción general

El propósito de este taller es que el estudiante implemente un servidor básico que reciba solicitudes de un cliente (por ejemplo, un cliente en Node.js o en consola) a través de **sockets TCP**.  

Cada solicitud corresponde a una **acción sobre tareas**, como crear, actualizar o listar.

Los datos se almacenarán mediante un **DAO**, el cual puede tener una implementación:

- **En memoria** (mínimo requerido)
- **Con JDBC y PostgreSQL** (puntuación extra)

---

## Protocolo de comunicación

El servidor recibirá mensajes en formato JSON **por línea** (`\n` al final).  
Cada mensaje contiene un comando y una estructura de datos asociada.

## Estructura de los mensajes y entidades

El servidor y el cliente se comunican **mediante mensajes JSON**.  
Cada mensaje tiene dos campos principales:
- `command`: indica la acción que el servidor debe ejecutar.
- `data`: contiene los datos necesarios para la operación.


### Ejemplos de mensajes

####  **Crear una tarea**
```json
{
  "command": "CREATE_TASK",
  "data": {
    "id": 0,
    "title": "Buy groceries",
    "description": "Milk, Bread, Eggs, Butter",
    "dueDate": "2023-10-01",
    "priority": "High"
  }
}
```
#### **Actualizar el estado o etapa de una tarea**

```json
{
  "command": "UPDATE_TASK",
  "data": {
    "stage": 2,
    "taskId": "1"
  }
}
```

#### **Obtener todas las tareas y etapas**

```json
{
  "command": "GET_TASKS",
  "data": {}
}

```

El servidor debe responder con la tarea creada en formato Json. Use gson para serializar los objetos.

---

###  Entidades principales

#### TaskStage (Etapa o lista de tareas)
Representa una agrupación de tareas, como "To Do", "In Progress" o "Done".

```json
{
  "id": 1,
  "name": "To Do",
  "description": "Tasks that need to be done",
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "description": "Milk, Bread, Eggs, Butter",
      "dueDate": "2023-10-01",
      "priority": "High"
    },
    {
      "id": 2,
      "title": "Read a book",
      "description": "Finish reading 'The Great Gatsby'",
      "dueDate": "2023-10-05",
      "priority": "Medium"
    }
  ]
}
```

**Task (Tarea individual)**

Representa una tarea que pertenece a una etapa específica.
```json
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk, Bread, Eggs, Butter",
  "dueDate": "2023-10-01",
  "priority": "High"
}

```

## Ejecución de la aplicación

Para validar el funcionamiento de servidor tcp, se ha dejado la implementación de un cliente web para revisar las interacciones con el servicio. Para ejecutar estos elementos siga los siguientes pasos:

### Verificar herramientas

Para este la ejecución del proyecto se requiere **nodejs**, un **servidor web** (puede ser el Live Preview de vscode) y **java** instalado en la máquina.

### Instalación de dependencias.

```bash
cd web-client
npm install
```

### Ejecución del proxy web

dentro de la carpeta **web-client**

```bash
node proxy/index.js
```

### Ejecución del Servidor TCP.

Puede ejecutarlo desde el editor ó usando la CLI

```bash
java -jar managerServer/build/libs/managerServer.jar
```

### Visualización de la página.

Abrir el archivo **index.html** con el Servidor web instalado. Si usa **Live Preview** abra el archivo en vscode y en la parte superior derecha puede pre visualizarlo. Copie la URL a un  navegador para mayor comodidad. 

> **Importante:**  
> 1. El servidor TCP debe exponer el puerto **5000** por la interface de red **localhost**
> 2. Este taller está diseñado para fortalecer habilidades en programación de redes, manejo de JSON y aplicación de patrones de diseño como DAO.  
> 3. Se recomienda seguir buenas prácticas de organización de código y documentación para facilitar la comprensión y mantenimiento del proyecto.


## Estado inicial.

La aplicación debe contar con tres TaskStage por defecto:

```java

  TaskStage s = new TaskStage();
  s.setId(1);
  s.setName("TO DO");
  s.setDescription("Desc");

  TaskStage s1 = new TaskStage();
  s1.setId(2);
  s1.setName("Doing");
  s1.setDescription("Desc");

  TaskStage s2 = new TaskStage();
  s2.setId(3);
  s2.setName("Done");
  s2.setDescription("Desc");
  
```
----------------------------------------------------------------------------------------------


</details>


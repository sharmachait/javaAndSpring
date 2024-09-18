```java
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.util.concurrent.CompletableFuture;

public class NetworkSender {
    public static CompletableFuture<Void> send(Socket socket, int data) {
        return CompletableFuture.runAsync(() -> {
            try (OutputStream os = socket.getOutputStream()) {
                // Convert data to bytes and send
                byte[] dataBytes = Integer.toString(data).getBytes();
                os.write(dataBytes);
                os.flush();
            } catch (IOException e) {
                e.printStackTrace(); // Handle exceptions appropriately
            }
        });
    }

    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8080); // Connect to server
            int data = 1234; // Example data to send

            // Call the send method and await its completion
            CompletableFuture<Void> future = send(socket, data);
            future.join(); // Waits for the task to complete

            // Close the socket after sending the data
            socket.close();
        } catch (IOException e) {
            e.printStackTrace(); // Handle exceptions appropriately
        }
    }
}
```

when you use `CompletableFuture.runAsync(() -> {})`, the code inside the lambda expression is executed on a separate thread. By default, it uses the **ForkJoinPool.commonPool()**, which is a thread pool provided by the Java runtime.

If you want to specify a custom thread pool, you can provide an `Executor` as a second argument to `runAsync`, like this:

```java
Executor executor = Executors.newFixedThreadPool(4);
CompletableFuture.runAsync(() -> {
    // your code here
}, executor);
```


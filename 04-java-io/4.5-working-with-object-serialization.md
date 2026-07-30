# Unit 4: Java IO

## Working with Object Serialization

**Object Serialization** is the process of converting the internal state of a Java object into a sequence of bytes. This byte stream can be saved to a disk file, stored in a database, or transmitted across a network. **Deserialization** performs the reverse operation, reconstructing the byte stream back into a live Java object in memory.

---

### Core Concepts

#### 1. The Serialization Architecture

```
                                  Serialization Process
  ┌─────────────────────────┐   (ObjectOutputStream.writeObject)   ┌───────────────────────────┐
  │   Live Object Instance  │ ───────────────────────────────────> │ Serialized Byte Stream    │
  │ (UserAccount in Memory) │ <─────────────────────────────────── │ (session_data.ser File)   │
  └─────────────────────────┘  Deserialization Process             └───────────────────────────┘
                                (ObjectInputStream.readObject)
```

#### 2. The `java.io.Serializable` Interface
- A class must implement the `java.io.Serializable` marker interface to enable serialization.
- `Serializable` is a **marker interface**—it contains no methods or fields. It informs the JVM that instances of the class are safe to serialize.

#### 3. Core Serialization Streams
- **`ObjectOutputStream`**: Wraps an underlying `OutputStream` (like `FileOutputStream`) and writes object graphs using `.writeObject(Object obj)`.
- **`ObjectInputStream`**: Wraps an `InputStream` (like `FileInputStream`) and reads object graphs using `.readObject()`, requiring an explicit type cast to the target class.

#### 4. The `transient` Modifier
- Fields declared with the `transient` keyword are **excluded from serialization**.
- Use `transient` for sensitive credentials (passwords, PINs), runtime-specific handles (file descriptors, database connections, thread references), or calculated fields that can be recomputed.

#### 5. `serialVersionUID`
- `serialVersionUID` is a unique `long` identifier used to verify that the sender and receiver of a serialized object maintain binary schema compatibility:
  ```java
  private static final long serialVersionUID = 1L;
  ```
- If no `serialVersionUID` is declared, the JVM computes one automatically based on class structure. Modifying class fields later changes this computed hash, causing deserialization to fail with an `InvalidClassException`.

---

### Common Pitfalls

1. **`NotSerializableException` on Non-Serializable Object Fields**:
   - If a class implements `Serializable`, but one of its non-transient member fields references an object that does **not** implement `Serializable`, Java throws a `NotSerializableException` at runtime.
2. **Expecting `static` Fields to Serialize**:
   - `static` fields belong to the class, not individual object instances. Therefore, `static` variables are not saved into serialized byte streams.
3. **Omitting `serialVersionUID`**:
   - Altering a class (such as adding a field) after saving objects to disk makes old serialized files unreadable unless a constant `serialVersionUID` is declared.

---

## 1. Demo: Basic Object Serialization and Transient Fields

### `SerializationBasicsDemo.java`

```java
import java.io.Serializable;
import java.io.ObjectOutputStream;
import java.io.ObjectInputStream;
import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.File;
import java.io.IOException;

/**
 * Demonstrates ObjectOutputStream, ObjectInputStream, and the transient keyword.
 */
class UserProfile implements Serializable {
    private static final long serialVersionUID = 101L;

    private String username;
    private String email;
    private transient String password; // Excluded from serialization

    public UserProfile(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }

    @Override
    public String toString() {
        return String.format("UserProfile [Username: %s | Email: %s | Password: %s]",
                username, email, (password == null ? "NULL (Excluded by transient)" : password));
    }
}

public class SerializationBasicsDemo {

    public static void main(String[] args) {
        File storageFile = new File("scratch/user_profile.ser");
        storageFile.getParentFile().mkdirs();

        // 1. Instantiate Object
        UserProfile originalUser = new UserProfile("alex_dev", "alex@example.com", "SecretPass123");
        System.out.println("Original Object Before Serialization:");
        System.out.println("  -> " + originalUser);

        // 2. Serialize Object to File (ObjectOutputStream)
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(storageFile))) {
            oos.writeObject(originalUser);
            System.out.println("\nObject serialized successfully to: " + storageFile.getName());
        } catch (IOException e) {
            System.err.println("Serialization Error: " + e.getMessage());
        }

        // 3. Deserialize Object from File (ObjectInputStream)
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(storageFile))) {
            UserProfile restoredUser = (UserProfile) ois.readObject();
            System.out.println("\nRestored Object After Deserialization:");
            System.out.println("  -> " + restoredUser);
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("Deserialization Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **`Serializable` Marker Interface**:
   - `UserProfile implements Serializable` declares that instances of `UserProfile` can be serialized by `ObjectOutputStream`.

2. **The `transient` Keyword**:
   - `private transient String password;` prevents sensitive password data from being written to disk. Upon deserialization, `password` resets to default `null`.

3. **Type Casting (`(UserProfile) ois.readObject()`)**:
   - `readObject()` returns type `Object`, requiring an explicit cast to `UserProfile`.

---

## 2. Practical Program: Enterprise User Session Store & Recovery Manager

### `EnterpriseSessionManager.java`

```java
import java.io.Serializable;
import java.io.ObjectOutputStream;
import java.io.ObjectInputStream;
import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.File;
import java.io.IOException;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * Enterprise user session state manager serializing complex object maps to binary storage for server session recovery.
 */
class UserSessionState implements Serializable {
    private static final long serialVersionUID = 20260730L;

    private String sessionId;
    private String userId;
    private Date loginTime;
    private Map<String, String> sessionAttributes;
    private transient long databaseConnectionHandle; // Unserializable hardware resource

    public UserSessionState(String sessionId, String userId) {
        this.sessionId = sessionId;
        this.userId = userId;
        this.loginTime = new Date();
        this.sessionAttributes = new HashMap<>();
        this.databaseConnectionHandle = 987654321L; // Simulated runtime handle
    }

    public void setAttribute(String key, String value) {
        sessionAttributes.put(key, value);
    }

    public void printSessionSummary() {
        System.out.println("Session ID     : " + sessionId);
        System.out.println("User ID        : " + userId);
        System.out.println("Login Time     : " + loginTime);
        System.out.println("Attributes     : " + sessionAttributes);
        System.out.println("DB Connection  : " + (databaseConnectionHandle == 0 ? "0 (Reset on Deserialization)" : databaseConnectionHandle));
    }
}

public class EnterpriseSessionManager {

    public static void saveSession(UserSessionState session, File destinationFile) {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(destinationFile))) {
            oos.writeObject(session);
            System.out.println("Session state saved successfully to: " + destinationFile.getAbsolutePath());
        } catch (IOException e) {
            System.err.println("Session save error: " + e.getMessage());
        }
    }

    public static UserSessionState restoreSession(File sourceFile) {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(sourceFile))) {
            return (UserSessionState) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("Session restoration error: " + e.getMessage());
            return null;
        }
    }

    public static void main(String[] args) {
        File sessionFile = new File("scratch/active_session.ser");
        sessionFile.getParentFile().mkdirs();

        // 1. Create and Configure Active User Session
        UserSessionState activeSession = new UserSessionState("SESS-88492", "USR-9901");
        activeSession.setAttribute("ROLE", "ADMINISTRATOR");
        activeSession.setAttribute("THEME", "DARK_MODE");
        activeSession.setAttribute("CART_ITEMS", "3");

        System.out.println("=== 1. ACTIVE SESSION BEFORE SERVER SHUTDOWN ===");
        activeSession.printSessionSummary();

        // 2. Persist Session to Disk (Server Shutdown Simulation)
        System.out.println("\n=== 2. PERSISTING SESSION TO BINARY STORAGE ===");
        saveSession(activeSession, sessionFile);

        // 3. Restore Session from Disk (Server Restart Simulation)
        System.out.println("\n=== 3. RESTORING SESSION AFTER SERVER RESTART ===");
        UserSessionState restoredSession = restoreSession(sessionFile);

        if (restoredSession != null) {
            restoredSession.printSessionSummary();
        }
    }
}
```

### Explanation

1. **Complex Object Graph Serialization**:
   - `UserSessionState` contains a `Map<String, String>` field. Because `HashMap` and `String` both implement `Serializable`, the entire object graph serializes cleanly.

2. **Excluding Runtime Handles**:
   - `databaseConnectionHandle` is marked `transient` because database sockets cannot be serialized to disk. Upon restoration, primitive transient numerical fields reset to `0`.

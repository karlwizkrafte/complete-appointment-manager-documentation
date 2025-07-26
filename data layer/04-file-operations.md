# File Operations

This document explains how save/load mechanisms work and how file lifecycle is managed in the data layer.

## File Creation and Database Initialization

New .apf files are created through the AppointmentManager:

```java
public boolean createNewFile(String filePath) {
    try {
        // Clear current appointments
        appointments.clear();
        
        // Connect to new database file (this will create it)
        if (databaseManager.connectToDatabase(filePath)) {
            currentFilePath = filePath;
            return true;
        }
        return false;
    } catch (Exception e) {
        System.err.println("Error creating new file: " + e.getMessage());
        return false;
    }
}
```

This method **clears existing data first**, then **creates a new SQLite database file** with the .apf extension. The DatabaseManager automatically creates the table schema when connecting.

## Save Operations with Data Migration

Saving to a new file involves migrating all current data:

```java
public boolean saveToFile(String filePath) {
    try {
        // Ensure .apf extension
        if (!filePath.toLowerCase().endsWith(".apf")) {
            filePath += ".apf";
        }
        
        // Connect to new database file
        if (databaseManager.connectToDatabase(filePath)) {
            currentFilePath = filePath;
            
            // Save all current appointments to the new database
            for (Appointment appointment : appointments) {
                if (!databaseManager.saveAppointment(appointment)) {
                    System.err.println("Failed to save appointment: " + appointment.getTitle());
                    return false;
                }
            }
            return true;
        }
        return false;
    } catch (Exception e) {
        System.err.println("Error saving to file: " + e.getMessage());
        return false;
    }
}
```

This process **automatically adds .apf extension**, **creates a new database connection**, and **iterates through all appointments** to save them individually. If any appointment fails to save, the entire operation fails.

## Load Operations with Data Replacement

Loading replaces all current data with file contents:

```java
public boolean loadFromFile(String filePath) {
    try {
        // Ensure .apf extension
        if (!filePath.toLowerCase().endsWith(".apf")) {
            filePath += ".apf";
        }
        
        // Connect to database file
        if (databaseManager.connectToDatabase(filePath)) {
            currentFilePath = filePath;
            
            // Load appointments from database
            List<Appointment> loadedAppointments = databaseManager.loadAppointments();
            
            // Clear current appointments and add loaded ones
            appointments.clear();
            appointments.addAll(loadedAppointments);
            
            return true;
        }
        return false;
    } catch (Exception e) {
        System.err.println("Error loading from file: " + e.getMessage());
        return false;
    }
}
```

This method **connects to the existing database**, **loads all appointments into a temporary list**, then **replaces the ObservableList contents**. The clear() and addAll() operations trigger UI updates automatically.

## File State Management

The manager tracks file state for UI feedback:

```java
private String currentFilePath;

public String getCurrentFilePath() {
    return currentFilePath;
}

public boolean isFileOpen() {
    return currentFilePath != null && databaseManager.isConnected();
}
```

The `isFileOpen()` method checks **both file path existence and active database connection**, ensuring accurate state reporting even if the database connection fails.

## Auto-Save with Current File

The save() method handles saving to the current file:

```java
public boolean save() {
    if (currentFilePath != null && databaseManager.isConnected()) {
        // Current implementation already saves automatically when adding/updating/deleting
        // This method can be used for explicit saves if needed
        return true;
    }
    return false;
}
```

Since the current implementation uses **automatic persistence** (saves occur immediately with each CRUD operation), this method primarily serves as a **validation check** that the file is still accessible.

## File Closure and Resource Cleanup

Closing files releases all resources:

```java
public void closeFile() {
    databaseManager.closeConnection();
    currentFilePath = null;
    appointments.clear();
}
```

This method performs **complete cleanup**:
1. **Closes database connection** to prevent file locks
2. **Clears file path** to indicate no file is open
3. **Empties the appointments list** to free memory and update UI

## Extension Handling

All file operations ensure proper .apf extension:

```java
// Ensure .apf extension
if (!filePath.toLowerCase().endsWith(".apf")) {
    filePath += ".apf";
}
```

This **case-insensitive check** automatically appends the extension if missing, ensuring consistent file naming and preventing user errors.

## Error Handling and Recovery

File operations include comprehensive error handling:

```java
try {
    // File operation logic
    return true;
} catch (Exception e) {
    System.err.println("Error loading from file: " + e.getMessage());
    return false;
}
```

This pattern **catches all exceptions** (not just SQL exceptions) to handle file system errors, permission issues, and other runtime problems that could occur during file operations.

## Data Integrity During Operations

Critical operations maintain data integrity:

```java
// Clear current appointments and add loaded ones
appointments.clear();
appointments.addAll(loadedAppointments);
```

The **atomic clear-and-replace operation** ensures the UI never shows a partially loaded state. The ObservableList changes trigger a single refresh cycle rather than multiple individual updates.

## File Path Normalization

File paths are normalized consistently:

```java
this.currentFilePath = filePath;  // Store the normalized path with .apf extension
```

The stored path **includes the .apf extension** even if the user didn't provide it, ensuring consistency in file path handling throughout the application.

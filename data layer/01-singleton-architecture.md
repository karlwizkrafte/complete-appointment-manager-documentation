# Singleton Architecture

This document explains how the AppointmentManager singleton pattern works and manages application state.

## Singleton Implementation

The AppointmentManager uses the classic lazy singleton pattern:

```java
public class AppointmentManager {
    private static AppointmentManager instance;
    private ObservableList<Appointment> appointments;
    private DatabaseManager databaseManager;
    private String currentFilePath;

    private AppointmentManager() {
        appointments = FXCollections.observableArrayList();
        databaseManager = new DatabaseManager();
    }

    public static AppointmentManager getInstance() {
        if (instance == null) {
            instance = new AppointmentManager();
        }
        return instance;
    }
}
```

This creates a **single point of data access** throughout the application. The private constructor prevents external instantiation, and `getInstance()` creates the instance only when first requested.

## Observable Data Management

The singleton maintains data in JavaFX's ObservableList for automatic UI updates:

```java
private ObservableList<Appointment> appointments;

public ObservableList<Appointment> getAppointments() {
    return appointments;
}
```

This design enables **automatic UI synchronization** - when data changes, all bound UI components (TableView, filters, status labels) update automatically without manual refresh calls.

## CRUD Operations with Persistence

Each data operation includes both memory and database updates:

```java
public void addAppointment(Appointment appointment) {
    appointments.add(appointment);
    
    // Save to database if connected
    if (databaseManager.isConnected()) {
        if (!databaseManager.saveAppointment(appointment)) {
            System.err.println("Failed to save appointment to database");
        }
    }
}
```

This pattern ensures **data consistency** between the in-memory ObservableList and the persistent SQLite database.

## Update Operations with Index Tracking

Updates handle both old and new appointment data:

```java
public void updateAppointment(int index, Appointment appointment) {
    if (index >= 0 && index < appointments.size()) {
        Appointment oldAppointment = appointments.get(index);
        appointments.set(index, appointment);
        
        // Update in database if connected
        if (databaseManager.isConnected()) {
            if (!databaseManager.updateAppointment(oldAppointment, appointment)) {
                System.err.println("Failed to update appointment in database");
            }
        }
    }
}
```

This method **preserves database consistency** by passing both old and new appointment data to the database layer for proper SQL UPDATE operations.

## Delete Operations with Dual API

The singleton provides two delete methods for different use cases:

```java
public void deleteAppointment(Appointment appointment) {
    appointments.remove(appointment);
    
    // Delete from database if connected
    if (databaseManager.isConnected()) {
        if (!databaseManager.deleteAppointment(appointment)) {
            System.err.println("Failed to delete appointment from database");
        }
    }
}

public void deleteAppointment(int index) {
    if (index >= 0 && index < appointments.size()) {
        Appointment appointment = appointments.get(index);
        deleteAppointment(appointment);
    }
}
```

The first method handles **object-based deletion**, while the second provides **index-based deletion** with bounds checking and delegation to the object-based method.

## State Tracking and File Management

The singleton tracks the current file state:

```java
private String currentFilePath;

public String getCurrentFilePath() {
    return currentFilePath;
}

public boolean isFileOpen() {
    return currentFilePath != null && databaseManager.isConnected();
}
```

This enables **file state awareness** throughout the application, allowing UI components to display current file information and prevent data loss during file operations.

## Bulk Operations

The singleton provides efficient bulk operations:

```java
public void clearAllAppointments() {
    appointments.clear();
    
    // Clear database if connected
    if (databaseManager.isConnected()) {
        if (!databaseManager.clearAllAppointments()) {
            System.err.println("Failed to clear appointments from database");
        }
    }
}
```

This method performs **atomic clear operations** on both memory and database, ensuring consistent state during file creation or major data operations.

## Thread Safety Considerations

While the current implementation is single-threaded (JavaFX Application Thread), the singleton pattern provides a foundation for **thread-safe data access**. All UI operations occur on the JavaFX Application Thread, eliminating concurrency issues.

## Memory Management

The singleton manages memory efficiently through:

```java
public void closeFile() {
    databaseManager.closeConnection();
    currentFilePath = null;
    appointments.clear();
}
```

This method **releases database connections** and clears in-memory data when files are closed, preventing memory leaks and resource conflicts.

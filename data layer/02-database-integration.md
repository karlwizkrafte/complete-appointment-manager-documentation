# Database Integration

This document explains how SQLite database operations work and how .apf files are structured and managed.

## Database Connection and File Format

The application uses SQLite databases with .apf (Appointment File) extension:

```java
public class DatabaseManager {
    private static final String DATABASE_URL_PREFIX = "jdbc:sqlite:";
    private Connection connection;
    private String currentDatabasePath;

    public boolean connectToDatabase(String filePath) {
        try {
            if (!filePath.toLowerCase().endsWith(".apf")) {
                filePath += ".apf";
            }
            
            this.currentDatabasePath = filePath;
            String url = DATABASE_URL_PREFIX + filePath;

            closeConnection();
            
            connection = DriverManager.getConnection(url);
            createAppointmentsTable();
            
            return true;
        } catch (SQLException e) {
            System.err.println("Error connecting to database: " + e.getMessage());
            return false;
        }
    }
}
```

This method **automatically appends .apf extension** and creates the SQLite database file if it doesn't exist. The connection is managed as a single instance variable.

## Database Schema Creation

The appointments table is created with a comprehensive schema:

```java
private void createAppointmentsTable() throws SQLException {
    String createTableSQL = "CREATE TABLE IF NOT EXISTS appointments (" +
        "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
        "title TEXT NOT NULL, " +
        "participant TEXT NOT NULL, " +
        "appointment_date TEXT NOT NULL, " +
        "appointment_time TEXT NOT NULL, " +
        "description TEXT, " +
        "status TEXT NOT NULL DEFAULT 'Scheduled', " +
        "created_at DATETIME DEFAULT CURRENT_TIMESTAMP, " +
        "updated_at DATETIME DEFAULT CURRENT_TIMESTAMP" +
        ")";

    try (Statement stmt = connection.createStatement()) {
        stmt.execute(createTableSQL);
    }
}
```

This schema includes **auto-incrementing primary keys**, **NOT NULL constraints** for required fields, **default values** for status and timestamps, and **audit trail fields** for tracking when records are created and modified.

## Insert Operations with Prepared Statements

New appointments are saved using parameterized queries:

```java
public boolean saveAppointment(Appointment appointment) {
    if (connection == null) {
        System.err.println("No database connection available");
        return false;
    }

    String insertSQL = "INSERT INTO appointments (title, participant, appointment_date, appointment_time, description, status) " +
        "VALUES (?, ?, ?, ?, ?, ?)";

    try (PreparedStatement pstmt = connection.prepareStatement(insertSQL)) {
        pstmt.setString(1, appointment.getTitle());
        pstmt.setString(2, appointment.getParticipant());
        pstmt.setString(3, appointment.getAppointmentDate());
        pstmt.setString(4, appointment.getAppointmentTime());
        pstmt.setString(5, appointment.getDescription());
        pstmt.setString(6, appointment.getStatus());
        
        int rowsAffected = pstmt.executeUpdate();
        return rowsAffected > 0;
    } catch (SQLException e) {
        System.err.println("Error saving appointment: " + e.getMessage());
        return false;
    }
}
```

This uses **prepared statements for SQL injection protection** and **validates success by checking affected row count**.

## Data Retrieval with Ordered Results

Loading appointments includes automatic sorting:

```java
public List<Appointment> loadAppointments() {
    List<Appointment> appointments = new ArrayList<>();
    
    if (connection == null) {
        System.err.println("No database connection available");
        return appointments;
    }

    String selectSQL = "SELECT title, participant, appointment_date, appointment_time, description, status " +
        "FROM appointments " +
        "ORDER BY appointment_date, appointment_time";

    try (Statement stmt = connection.createStatement();
         ResultSet rs = stmt.executeQuery(selectSQL)) {
        
        while (rs.next()) {
            Appointment appointment = new Appointment(
                rs.getString("title"),
                rs.getString("participant"),
                rs.getString("appointment_date"),
                rs.getString("appointment_time"),
                rs.getString("description"),
                rs.getString("status")
            );
            appointments.add(appointment);
        }
    } catch (SQLException e) {
        System.err.println("Error loading appointments: " + e.getMessage());
    }

    return appointments;
}
```

This query **orders results by date and time** and uses **try-with-resources for automatic resource cleanup**. Each ResultSet row is converted to an Appointment object.

## Update Operations with Composite Keys

Updates use multiple fields to identify records since there's no unique business key:

```java
public boolean updateAppointment(Appointment oldAppointment, Appointment newAppointment) {
    if (connection == null) {
        System.err.println("No database connection available");
        return false;
    }

    String updateSQL = "UPDATE appointments " +
        "SET title = ?, participant = ?, appointment_date = ?, appointment_time = ?, " +
        "description = ?, status = ?, updated_at = CURRENT_TIMESTAMP " +
        "WHERE title = ? AND participant = ? AND appointment_date = ? AND appointment_time = ?";

    try (PreparedStatement pstmt = connection.prepareStatement(updateSQL)) {
        // New values
        pstmt.setString(1, newAppointment.getTitle());
        pstmt.setString(2, newAppointment.getParticipant());
        pstmt.setString(3, newAppointment.getAppointmentDate());
        pstmt.setString(4, newAppointment.getAppointmentTime());
        pstmt.setString(5, newAppointment.getDescription());
        pstmt.setString(6, newAppointment.getStatus());
        
        // Old values for WHERE clause
        pstmt.setString(7, oldAppointment.getTitle());
        pstmt.setString(8, oldAppointment.getParticipant());
        pstmt.setString(9, oldAppointment.getAppointmentDate());
        pstmt.setString(10, oldAppointment.getAppointmentTime());
        
        int rowsAffected = pstmt.executeUpdate();
        return rowsAffected > 0;
    } catch (SQLException e) {
        System.err.println("Error updating appointment: " + e.getMessage());
        return false;
    }
}
```

This approach uses **composite key matching** with the original appointment's core fields to identify the correct record for updates. It also **automatically updates the timestamp** on modifications.

## Connection State Management

The database manager tracks connection state:

```java
public boolean isConnected() {
    try {
        return connection != null && !connection.isClosed();
    } catch (SQLException e) {
        return false;
    }
}

public void closeConnection() {
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            System.err.println("Error closing database connection: " + e.getMessage());
        }
        connection = null;
    }
}
```

These methods provide **safe connection checking** and **proper resource cleanup** to prevent database locks and memory leaks.

## Error Handling Strategy

All database operations follow a consistent error handling pattern:

```java
try (PreparedStatement pstmt = connection.prepareStatement(insertSQL)) {
    // Database operation
    int rowsAffected = pstmt.executeUpdate();
    return rowsAffected > 0;
} catch (SQLException e) {
    System.err.println("Error saving appointment: " + e.getMessage());
    return false;
}
```

This pattern **logs errors to stderr** for debugging while **returning boolean success indicators** to calling code, allowing graceful error handling without exposing SQL exceptions to the UI layer.

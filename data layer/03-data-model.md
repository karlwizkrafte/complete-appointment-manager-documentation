# Data Model

This document explains how the JavaFX Properties pattern is implemented in the Appointment class for automatic UI binding.

## JavaFX Properties Architecture

The Appointment class uses StringProperty fields for automatic UI binding:

```java
public class Appointment {
    private final StringProperty title;
    private final StringProperty participant;
    private final StringProperty appointmentDate;
    private final StringProperty appointmentTime;
    private final StringProperty description;
    private final StringProperty status;
}
```

These **final StringProperty fields** enable automatic UI updates when data changes. JavaFX's binding system monitors these properties and refreshes bound UI components automatically.

## Property Initialization

The constructor initializes all properties with SimpleStringProperty instances:

```java
public Appointment(String title, String participant, String appointmentDate, 
                  String appointmentTime, String description, String status) {
    this.title = new SimpleStringProperty(title);
    this.participant = new SimpleStringProperty(participant);
    this.appointmentDate = new SimpleStringProperty(appointmentDate);
    this.appointmentTime = new SimpleStringProperty(appointmentTime);
    this.description = new SimpleStringProperty(description);
    this.status = new SimpleStringProperty(status);
}
```

This pattern **wraps primitive String values** in JavaFX Property objects, enabling change notification and binding capabilities while maintaining simple String-based APIs.

## Default Constructor with Sensible Defaults

The parameterless constructor provides reasonable defaults:

```java
public Appointment() {
    this("", "", "", "", "", "Scheduled");
}
```

This **delegates to the full constructor** with empty strings and a default "Scheduled" status, ensuring all properties are properly initialized even for empty appointment objects.

## Property Access Pattern

Each field follows the JavaFX property accessor pattern:

```java
// Title
public StringProperty titleProperty() { return title; }
public String getTitle() { return title.get(); }
public void setTitle(String title) { this.title.set(title); }
```

This provides **three levels of access**:
1. **Property access** (`titleProperty()`) for binding and listeners
2. **Value getter** (`getTitle()`) for reading current values
3. **Value setter** (`setTitle()`) for updating values

## TableView Integration

The property pattern enables automatic TableView integration:

```java
// In TableViewController
titleColumn.setCellValueFactory(new PropertyValueFactory<>("title"));
```

The `PropertyValueFactory` **automatically calls the property methods** by name, using reflection to find `titleProperty()`. This eliminates the need for custom cell value factories.

## Property Binding for UI Updates

JavaFX components can bind directly to properties:

```java
// Example binding (not in current code, but enabled by the pattern)
textField.textProperty().bind(appointment.titleProperty());
```

This would create **automatic two-way binding** where changes to the appointment title immediately update the text field, and vice versa.

## Change Notification System

Properties support change listeners for custom behavior:

```java
// Example listener (enabled by the pattern)
appointment.titleProperty().addListener((observable, oldValue, newValue) -> {
    System.out.println("Title changed from " + oldValue + " to " + newValue);
});
```

This enables **reactive programming patterns** where UI components and business logic can respond automatically to data changes.

## Memory Efficiency Considerations

The property pattern has memory overhead compared to plain fields:

```java
// Memory usage per appointment:
// 6 StringProperty objects + 6 String values
// vs. 6 String values with plain fields
```

However, this overhead enables **automatic UI synchronization** that would otherwise require manual notification code, making the trade-off worthwhile for UI-bound data.

## String Representation for Debugging

The class includes a comprehensive toString() method:

```java
@Override
public String toString() {
    return String.format("Appointment{title='%s', participant='%s', date='%s', time='%s', status='%s'}", 
                       getTitle(), getParticipant(), getAppointmentDate(), getAppointmentTime(), getStatus());
}
```

This provides **formatted debug output** showing the current values of all properties, useful for logging and debugging data flow issues.

## Immutable Property References

The property fields are declared final:

```java
private final StringProperty title;
```

This ensures **property object immutability** - while the values can change, the property objects themselves cannot be reassigned, providing stability for binding and listener systems.

## Type Safety and Null Handling

All properties use String types, providing consistent data handling:

```java
public String getTitle() { return title.get(); }
```

The `title.get()` call **returns null if the property value is null**, maintaining standard Java null semantics while enabling JavaFX property features.

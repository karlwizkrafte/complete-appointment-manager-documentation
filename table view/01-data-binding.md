# Data Binding

This document explains how appointment data is bound to the TableView and how the MVC pattern is implemented.

## TableView Column Binding

The table columns are bound to Appointment object properties using `PropertyValueFactory`:

```java
@Override
public void initialize(URL location, ResourceBundle resources) {
    appointmentManager = AppointmentManager.getInstance();
    
    // Set up table columns
    titleColumn.setCellValueFactory(new PropertyValueFactory<>("title"));
    participantColumn.setCellValueFactory(new PropertyValueFactory<>("participant"));
    appointmentDateColumn.setCellValueFactory(new PropertyValueFactory<>("appointmentDate"));
    appointmentTimeColumn.setCellValueFactory(new PropertyValueFactory<>("appointmentTime"));
    descriptionColumn.setCellValueFactory(new PropertyValueFactory<>("description"));
    statusColumn.setCellValueFactory(new PropertyValueFactory<>("status"));
}
```

This code:
1. Gets the singleton `AppointmentManager` instance for data access
2. **Maps each column to Appointment properties** using reflection-based `PropertyValueFactory`
3. **Automatically handles data display** without manual cell rendering

## Observable Data Integration

The table uses JavaFX's observable collections for automatic UI updates:

```java
// Prepare search bar
filteredAppointments = new FilteredList<>(appointmentManager.getAppointments(), p -> true);
appointmentTable.setItems(filteredAppointments);
```

This creates a layered data structure:
1. **Base data**: `AppointmentManager.getAppointments()` returns `ObservableList<Appointment>`
2. **Filtered view**: `FilteredList` wraps the base list with a predicate filter
3. **Table binding**: TableView displays the filtered list

## Automatic UI Updates

When the underlying data changes, the UI automatically updates:

```java
@FXML
private void deleteSelectedAppointment() {
    // ... selection and confirmation logic ...
    
    if (result.isPresent() && result.get() == ButtonType.OK) {
        appointmentManager.deleteAppointment(selectedAppointment);
        updateStatusLabelWithSearch();
        statusLabel.setText("Appointment deleted successfully.");
    }
}
```

The `deleteAppointment()` call modifies the `ObservableList`, which automatically:
1. **Updates the FilteredList** (re-applies current filters)
2. **Refreshes the TableView** display
3. **Maintains selection state** where possible

## Data Refresh Mechanism

Manual refresh operations reapply all filters and update status:

```java
@FXML
private void refreshTable() {
    appointmentTable.refresh();
    
    // Reapply all filters to maintain persistence
    applyAllFilters();
    
    statusLabel.setText("Table refreshed.");
}
```

This method:
1. **Forces TableView refresh** to handle any cell rendering updates
2. **Reapplies combined filters** (search + advanced filters)
3. **Updates status label** to reflect current state

## Selection Model Integration

The table uses JavaFX's selection model for item tracking:

```java
@FXML
private void editSelectedAppointment() throws IOException {
    Appointment selectedAppointment = appointmentTable.getSelectionModel().getSelectedItem();
    if (selectedAppointment != null) {
        PopupFormController.setEditingAppointment(selectedAppointment);
        openAppointmentPopup();
    } else {
        showAlert("No Selection", "Please select an appointment to edit.");
    }
}
```

This pattern:
1. **Gets selected item** from the table's selection model
2. **Validates selection exists** before proceeding
3. **Passes data to popup controllers** for editing
4. **Shows user-friendly alerts** for invalid operations

## Property-Based Data Access

The `PropertyValueFactory` relies on JavaBean naming conventions:

```java
// In Appointment class (implied)
public class Appointment {
    private String title;
    private String participant;
    // ... other fields
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    // ... other getters/setters
}
```

This allows:
- **Automatic data binding** without custom cell factories
- **Sortable columns** by default (clicking headers sorts data)
- **Type-safe data access** with compile-time checking

## Status Label Data Binding

The status label reflects real-time data state:

```java
private void updateStatusLabelWithSearch() {
    int filteredCount = filteredAppointments.size();
    int totalCount = appointmentManager.getAppointments().size();
    
    String statusText = "";
    boolean hasSearch = searchField.getText() != null && !searchField.getText().trim().isEmpty();
    boolean hasAdvancedFilters = currentFilterCriteria != null && currentFilterCriteria.hasActiveFilters();
    
    if (!hasSearch && !hasAdvancedFilters) {
        statusText = "Total appointments: " + totalCount;
    } else {
        statusText = "Showing " + filteredCount + " of " + totalCount + " appointments";
        // ... filter status logic
    }
    
    statusLabel.setText(statusText);
}
```

This provides:
- **Real-time count updates** based on filtered vs total data
- **Filter state indication** to inform users about active filters
- **Dynamic status messages** that change based on application state

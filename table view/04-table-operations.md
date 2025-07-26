# Table Operations

This document explains how CRUD (Create, Read, Update, Delete) operations work with the appointment data.

## Create Operation (Add Appointment)

New appointments are created through the popup form:

```java
private void showAddAppointment() throws IOException {
    // Clean up appointment data from popup form
    PopupFormController.setEditingAppointment(null);
    openAppointmentPopup();
}
```

The process:
1. **Clears any existing appointment data** by setting `null`
2. **Opens the popup form** in "add" mode
3. **PopupFormController** handles the actual creation
4. **Table refreshes automatically** when popup closes

## Read Operation (Display and Selection)

Data reading happens through the TableView selection model:

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
1. **Gets selected item** from JavaFX selection model
2. **Validates selection exists** before proceeding
3. **Passes selected data** to editing controller
4. **Provides user feedback** for invalid states

## Update Operations

### Full Appointment Edit
Full editing uses the same popup as creation:

```java
@FXML
private void editSelectedAppointment() throws IOException {
    Appointment selectedAppointment = appointmentTable.getSelectionModel().getSelectedItem();
    if (selectedAppointment != null) {
        PopupFormController.setEditingAppointment(selectedAppointment);
        openAppointmentPopup();
    }
    // ... error handling
}
```

The edit mode:
- **Reuses the add/edit popup form**
- **Pre-populates fields** with existing data
- **Saves changes** to the existing appointment object

### Quick Status Update
Status-only updates use a dedicated smaller popup:

```java
@FXML
private void updateSelectedAppointmentStatus() throws IOException {
    Appointment selectedAppointment = appointmentTable.getSelectionModel().getSelectedItem();
    if (selectedAppointment != null) {
        StatusUpdateController.setSelectedAppointment(selectedAppointment);
        openStatusUpdatePopup();
    } else {
        showAlert("No Selection", "Please select an appointment to update status.");
    }
}
```

This provides:
- **Faster status changes** without full form
- **Focused UI** for common operation
- **Same validation pattern** as other operations

## Delete Operation

Deletion includes confirmation dialog for data safety:

```java
@FXML
private void deleteSelectedAppointment() {
    Appointment selectedAppointment = appointmentTable.getSelectionModel().getSelectedItem();
    if (selectedAppointment != null) {
        Alert confirmAlert = new Alert(Alert.AlertType.CONFIRMATION);
        confirmAlert.setTitle("Confirm Deletion");
        confirmAlert.setHeaderText("Delete Appointment");
        confirmAlert.setContentText("Are you sure you want to delete the appointment '" + 
                                  selectedAppointment.getTitle() + "'?");

        Optional<ButtonType> result = confirmAlert.showAndWait();
        if (result.isPresent() && result.get() == ButtonType.OK) {
            appointmentManager.deleteAppointment(selectedAppointment);
            updateStatusLabelWithSearch();
            statusLabel.setText("Appointment deleted successfully.");
        }
    } else {
        showAlert("No Selection", "Please select an appointment to delete.");
    }
}
```

The deletion process:
1. **Validates selection exists**
2. **Shows confirmation dialog** with appointment title
3. **Waits for user confirmation** using `Optional<ButtonType>`
4. **Performs deletion through AppointmentManager**
5. **Updates status indicators** and provides feedback
6. **Automatic UI refresh** through ObservableList changes

## Batch Operations and Refresh

The refresh operation maintains filter state:

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
- **Forces TableView to redraw** all visible cells
- **Reapplies search and advanced filters**
- **Updates status label** with confirmation message

## Callback from Popup Operations

The table controller receives callbacks after popup operations:

```java
public void onReturnFromPopupForm() {
    refreshTable();
    updateStatusLabelWithSearch();
}
```

This callback:
- **Refreshes the table display**
- **Updates status labels** to reflect changes
- **Maintains filter consistency**

## Data Validation Pattern

Operations consistently validate selection before proceeding:

```java
private void validateSelectionAndExecute(String operationName, Runnable operation) {
    Appointment selectedAppointment = appointmentTable.getSelectionModel().getSelectedItem();
    if (selectedAppointment != null) {
        operation.run();
    } else {
        showAlert("No Selection", "Please select an appointment to " + operationName.toLowerCase() + ".");
    }
}
```

While this pattern isn't explicitly implemented, the validation logic is consistent across:
- Edit operations
- Status updates  
- Delete operations
- Other selection-dependent actions

## Status Label Updates

Operations update the status label to provide user feedback:

```java
private void updateStatusLabelWithSearch() {
    int filteredCount = filteredAppointments.size();
    int totalCount = appointmentManager.getAppointments().size();
    
    // ... status text calculation based on filters ...
    
    statusLabel.setText(statusText);
}
```

Status updates reflect:
- **Current record counts** (filtered vs total)
- **Active filter states** (search, advanced filters)
- **Operation results** (success/failure messages)

## Observable List Integration

All operations work through the `AppointmentManager`'s `ObservableList`:

```java
appointmentManager = AppointmentManager.getInstance();
filteredAppointments = new FilteredList<>(appointmentManager.getAppointments(), p -> true);
```

This ensures:
- **Automatic UI updates** when data changes
- **Filter persistence** across operations
- **Consistent data state** between controller and manager
- **Change propagation** to all connected UI components

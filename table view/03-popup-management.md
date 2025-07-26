# Popup Management

This document explains how modal dialog windows are created, managed, and integrated with the main table view.

## Modal Dialog Architecture

All popup dialogs use a consistent creation pattern with `StageStyle.UNDECORATED`:

```java
private void openAppointmentPopup() throws IOException {
    FXMLLoader loader = new FXMLLoader(getClass().getResource("popupform.fxml"));
    Parent root = loader.load();
    
    Stage popupStage = new Stage();
    popupStage.initStyle(StageStyle.UNDECORATED); // Remove native title bar
    popupStage.initModality(Modality.APPLICATION_MODAL);
    popupStage.initOwner(App.getPrimaryStage());
    
    Scene scene = new Scene(root, 450, 400);
    popupStage.setScene(scene);
    popupStage.setTitle("Appointment Form");
    popupStage.setResizable(false);
    
    PopupFormController controller = loader.getController();
    controller.setPopupStage(popupStage);
    
    popupStage.showAndWait();
    
    refreshTable();
}
```

This pattern:
1. **Loads FXML using FXMLLoader** with resource-relative paths
2. **Creates undecorated modal stage** for consistent styling
3. **Sets parent relationship** for proper window management
4. **Gets controller reference** for bi-directional communication
5. **Uses showAndWait()** to block until dialog closes
6. **Refreshes table** after dialog operations

## Edit vs Add Mode Management

The appointment popup handles both add and edit operations:

```java
// For adding new appointments
private void showAddAppointment() throws IOException {
    // Clean up appointment data from popup form
    PopupFormController.setEditingAppointment(null);
    openAppointmentPopup();
}

// For editing existing appointments
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

This approach:
- **Uses static controller state** to pass data between controllers
- **Null appointment indicates add mode**, non-null indicates edit mode
- **Validates selection exists** before opening edit dialog
- **Provides user feedback** for invalid operations

## Status Update Dialog

The status update popup uses a smaller, focused interface:

```java
private void openStatusUpdatePopup() throws IOException {
    FXMLLoader loader = new FXMLLoader(getClass().getResource("statusupdate.fxml"));
    Parent root = loader.load();
    
    Stage popupStage = new Stage();
    popupStage.initStyle(StageStyle.UNDECORATED); // Remove native title bar
    popupStage.initModality(Modality.APPLICATION_MODAL);
    popupStage.initOwner(App.getPrimaryStage());
    
    Scene scene = new Scene(root, 380, 280);
    popupStage.setScene(scene);
    popupStage.setTitle("Update Status");
    popupStage.setResizable(false);
    
    StatusUpdateController controller = loader.getController();
    controller.setPopupStage(popupStage);
    
    popupStage.showAndWait();
    
    refreshTable();
}
```

Key differences:
- **Smaller dimensions** (380x280 vs 450x400) for focused task
- **Different FXML file** (`statusupdate.fxml`)
- **Specific controller type** (`StatusUpdateController`)

## Filter Dialog Management

The filter popup handles complex filtering criteria:

```java
@FXML
private void openFilterPopup() throws IOException {
    FXMLLoader loader = new FXMLLoader(getClass().getResource("filter.fxml"));
    Parent root = loader.load();
    
    Stage popupStage = new Stage();
    popupStage.initStyle(StageStyle.UNDECORATED); // Remove native title bar
    popupStage.initModality(Modality.APPLICATION_MODAL);
    popupStage.initOwner(App.getPrimaryStage());
    
    Scene scene = new Scene(root, 490, 450);
    popupStage.setScene(scene);
    popupStage.setTitle("Filter Appointments");
    popupStage.setResizable(false);
    
    FilterController controller = loader.getController();
    controller.setPopupStage(popupStage);
    controller.setTableViewController(this);
    
    popupStage.showAndWait();
    
    // Refresh table after filter is applied
    refreshTable();
}
```

Unique aspects:
- **Largest dimensions** (490x450) for complex filter UI
- **Bi-directional controller reference** with `setTableViewController(this)`
- **Filter persistence** through controller communication

## Controller Communication Pattern

Popup controllers receive references to manage their lifecycle:

```java
// In popup controllers
public void setPopupStage(Stage popupStage) {
    this.popupStage = popupStage;
}

// For filter controller specifically
public void setTableViewController(TableViewController tableViewController) {
    this.tableViewController = tableViewController;
}
```

This enables:
- **Popup self-closure** when operations complete
- **Data passing back to parent** controller
- **Filter application** directly to table

## Error Handling and Validation

Selection-based operations include validation:

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

This pattern:
1. **Gets selected item from table**
2. **Validates selection exists**
3. **Passes data to static controller method**
4. **Shows user-friendly error alerts**

## Post-Popup Operations

All popup operations conclude with table refresh:

```java
// After popup closes
refreshTable();

// The refresh method maintains filter state
@FXML
private void refreshTable() {
    appointmentTable.refresh();
    
    // Reapply all filters to maintain persistence
    applyAllFilters();
    
    statusLabel.setText("Table refreshed.");
}
```

This ensures:
- **Data changes are reflected** in the table
- **Filter state persists** across operations
- **Status feedback** is provided to users

## Modal Window Properties

All popups share consistent modal properties:

```java
popupStage.initStyle(StageStyle.UNDECORATED);     // Custom title bar consistency
popupStage.initModality(Modality.APPLICATION_MODAL); // Blocks interaction with parent
popupStage.initOwner(App.getPrimaryStage());      // Proper parent-child relationship
popupStage.setResizable(false);                   // Fixed size for predictable layout
```

This creates:
- **Visual consistency** with main window styling
- **Proper modal behavior** blocking parent interaction
- **Correct window hierarchy** for proper task switching
- **Fixed layouts** preventing UI breaking from resizing

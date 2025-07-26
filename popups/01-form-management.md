# Form Management

This document explains how the PopupFormController handles both add and edit modes with form validation and data binding.

## Static State Management for Mode Detection

The controller uses static fields to determine add vs edit mode:

```java
public class PopupFormController implements Initializable {
    private AppointmentManager appointmentManager;
    private static Appointment editingAppointment = null;
    private boolean isEditMode = false;
    private Stage popupStage;

    @Override
    public void initialize(URL location, ResourceBundle resources) {
        appointmentManager = AppointmentManager.getInstance();
        
        // Check if Existing Appointment
        if (editingAppointment != null) {
            isEditMode = true;
            formTitle.setText("Update Appointment");
            populateFormWithAppointment(editingAppointment);
        } else {
            isEditMode = false;
            formTitle.setText("Appointment Form");
            clearForm();
        }
    }
}
```

This pattern uses a **static field for inter-controller communication**. When `editingAppointment` is null, the form opens in add mode; when non-null, it opens in edit mode with pre-populated data.

## Dynamic Time Selection Components

The form includes composite time selection with multiple ComboBoxes:

```java
// Initialize Time
for (int i = 1; i <= 12; i++) {
    hoursComboBox.getItems().add(String.format("%02d", i));
}
for (int i = 0; i <= 59; i++) {
    minutesComboBox.getItems().add(String.format("%02d", i));
}
ampmComboBox.getItems().addAll("AM", "PM");
```

This creates **12-hour time format selection** with zero-padded formatting. The separate ComboBoxes provide better user experience than free-text time entry.

## Status Dropdown Initialization

The status field includes predefined workflow states:

```java
// Initialize Status
statusComboBox.getItems().addAll(
    "Scheduled", "Confirmed", "In Progress", "Completed", "Cancelled", "No Show"
);
statusComboBox.setValue("Scheduled");
```

This provides **structured status values** with a sensible default, ensuring data consistency across appointments.

## Dual-Mode Save Operation

The save operation handles both add and edit modes:

```java
@FXML
private void saveAppointment() throws IOException {
    if (validateForm()) {
        String title = titleField.getText().trim();
        String participant = participantField.getText().trim();
        String appointmentDate = appointmentDatePicker.getValue() != null ? 
            appointmentDatePicker.getValue().format(DateTimeFormatter.ofPattern("yyyy-MM-dd")) : "";
        String appointmentTime = getFormattedTime();
        String description = descriptionArea.getText().trim();
        String status = statusComboBox.getValue();

        if (isEditMode && editingAppointment != null) {
            // Update appointment
            editingAppointment.setTitle(title);
            editingAppointment.setParticipant(participant);
            editingAppointment.setAppointmentDate(appointmentDate);
            editingAppointment.setAppointmentTime(appointmentTime);
            editingAppointment.setDescription(description);
            editingAppointment.setStatus(status);
            
            showAlert("Success", "Appointment updated successfully!");
        } else {
            // Create appointment
            Appointment newAppointment = new Appointment(
                title, participant, appointmentDate, appointmentTime, description, status
            );
            appointmentManager.addAppointment(newAppointment);
            showAlert("Success", "Appointment added successfully!");
        }

        // Clear editing and close popup
        editingAppointment = null;
        if (popupStage != null) {
            popupStage.close();
        }
    }
}
```

This method **validates form data first**, then **branches based on edit mode**. Edit mode updates the existing appointment object directly, while add mode creates a new appointment through the AppointmentManager.

## Time Formatting Logic

Multiple ComboBox values are combined into a formatted time string:

```java
private String getFormattedTime() {
    String hour = hoursComboBox.getValue();
    String minute = minutesComboBox.getValue();
    String ampm = ampmComboBox.getValue();
    
    if (hour != null && minute != null && ampm != null) {
        return hour + ":" + minute + " " + ampm;
    }
    return "";
}
```

This method **validates all time components are selected** before formatting, returning an empty string if any component is missing.

## Date Formatting with Null Safety

Date picker values are safely converted to strings:

```java
String appointmentDate = appointmentDatePicker.getValue() != null ? 
    appointmentDatePicker.getValue().format(DateTimeFormatter.ofPattern("yyyy-MM-dd")) : "";
```

This **null-safe operation** formats the LocalDate using ISO date format, or returns empty string if no date is selected.

## Form Clearing Mechanism

The clear operation resets all form fields:

```java
@FXML
private void clearForm() {
    titleField.clear();
    participantField.clear();
    appointmentDatePicker.setValue(null);
    hoursComboBox.setValue(null);
    minutesComboBox.setValue(null);
    ampmComboBox.setValue(null);
    descriptionArea.clear();
    statusComboBox.setValue("Scheduled");
}
```

This method **resets all input fields** while maintaining the default "Scheduled" status, providing a clean form state for new appointments.

## Static State Cleanup

The controller cleans up static state when closing:

```java
@FXML
private void switchToTableView() throws IOException {
    editingAppointment = null;
    if (popupStage != null) {
        popupStage.close();
    }
}

// Also in saveAppointment after successful save:
editingAppointment = null;
if (popupStage != null) {
    popupStage.close();
}
```

This **prevents memory leaks** and ensures the next popup opens in the correct mode by clearing the static appointment reference.

## Stage Reference Management

The controller receives its stage reference from the parent:

```java
public void setPopupStage(Stage popupStage) {
    this.popupStage = popupStage;
}
```

This enables **self-closing behavior** and **window dragging functionality** without requiring complex parent-child communication patterns.

## Window Dragging Implementation

The popup supports dragging via custom title bar:

```java
@FXML
private void onTitleBarPressed(MouseEvent event) {
    xOffset = event.getSceneX();
    yOffset = event.getSceneY();
}

@FXML
private void onTitleBarDragged(MouseEvent event) {
    if (popupStage != null) {
        popupStage.setX(event.getScreenX() - xOffset);
        popupStage.setY(event.getScreenY() - yOffset);
    }
}
```

This provides **consistent window behavior** across all popup dialogs, maintaining the custom title bar aesthetic while preserving user expectations for window manipulation.

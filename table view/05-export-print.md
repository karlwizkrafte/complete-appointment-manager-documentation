# Export and Print

This document explains how file operations and printing functionality work in the table view.

## File Loading with Validation

Loading appointments includes unsaved changes detection:

```java
@FXML
private void loadAppointments() {
    // Check if there are unsaved changes
    if (appointmentManager.getAppointments().size() > 0 && !appointmentManager.isFileOpen()) {
        Alert confirmAlert = new Alert(Alert.AlertType.CONFIRMATION);
        confirmAlert.setTitle("Unsaved Changes");
        confirmAlert.setHeaderText("Load New File");
        confirmAlert.setContentText("You have unsaved appointments. Loading a new file will replace them. Continue?");

        Optional<ButtonType> result = confirmAlert.showAndWait();
        if (result.isEmpty() || result.get() != ButtonType.OK) {
            return;
        }
    }

    FileChooser fileChooser = new FileChooser();
    fileChooser.setTitle("Load Appointments");
    fileChooser.getExtensionFilters().add(
        new FileChooser.ExtensionFilter("Appointment Files", "*.apf")
    );

    File file = fileChooser.showOpenDialog(App.getPrimaryStage());
    if (file != null) {
        String filePath = file.getAbsolutePath();
        if (appointmentManager.loadFromFile(filePath)) {
            updateStatusLabelWithSearch();
            statusLabel.setText("Appointments loaded from: " + file.getName());
        } else {
            showAlert("Load Error", "Failed to load appointments from file.");
        }
    }
}
```

This implementation:
1. **Checks for unsaved changes** using `appointmentManager.isFileOpen()`
2. **Shows confirmation dialog** to prevent data loss
3. **Uses FileChooser with .apf filter** for file selection
4. **Delegates actual loading** to `AppointmentManager`
5. **Updates UI status** based on operation result

## File Creation Process

Creating new files includes similar validation:

```java
@FXML
private void newAppointmentFile() {
    // Check if there are unsaved changes
    if (appointmentManager.getAppointments().size() > 0 && !appointmentManager.isFileOpen()) {
        Alert confirmAlert = new Alert(Alert.AlertType.CONFIRMATION);
        confirmAlert.setTitle("Unsaved Changes");
        confirmAlert.setHeaderText("Create New File");
        confirmAlert.setContentText("You have unsaved appointments. Creating a new file will clear them. Continue?");

        Optional<ButtonType> result = confirmAlert.showAndWait();
        if (result.isEmpty() || result.get() != ButtonType.OK) {
            return;
        }
    }

    FileChooser fileChooser = new FileChooser();
    fileChooser.setTitle("Create New Appointment File");
    fileChooser.getExtensionFilters().add(
        new FileChooser.ExtensionFilter("Appointment Files", "*.apf")
    );
    fileChooser.setInitialFileName("new_appointments.apf");

    File file = fileChooser.showSaveDialog(App.getPrimaryStage());
    if (file != null) {
        String filePath = file.getAbsolutePath();
        if (appointmentManager.createNewFile(filePath)) {
            updateStatusLabelWithSearch();
            statusLabel.setText("New appointment file created: " + file.getName());
        } else {
            showAlert("Creation Error", "Failed to create new appointment file.");
        }
    }
}
```

Key features:
- **Pre-sets default filename** with `setInitialFileName()`
- **Uses save dialog** instead of open dialog
- **Same validation pattern** as loading

## Print Job Initialization

Printing uses JavaFX's PrinterJob API:

```java
@FXML
private void printAppointments() {
    PrinterJob printerJob = PrinterJob.createPrinterJob();
    if (printerJob != null && printerJob.showPrintDialog(App.getPrimaryStage())) {
        
        // Set margins
        double leftMargin = 50;  
        double topMargin = 50;  
        double rightMargin = 50;  
        double bottomMargin = 50;  
        
        // Configure page layout
        Printer printer = printerJob.getPrinter();
        PageLayout pageLayout = printer.createPageLayout(
            printer.getDefaultPageLayout().getPaper(),
            PageOrientation.PORTRAIT,
            leftMargin, rightMargin, topMargin, bottomMargin
        );
        printerJob.getJobSettings().setPageLayout(pageLayout);
        
        // Print appointments with pagination
        boolean success = printAppointmentsWithPagination(printerJob, pageLayout);
        
        if (success) {
            printerJob.endJob();
            statusLabel.setText("All " + appointmentManager.getAppointments().size() + " appointments printed successfully across multiple pages.");
        } else {
            statusLabel.setText("Printing failed.");
        }
    } else {
        statusLabel.setText("Printing cancelled or no printer available.");
    }
}
```

This process:
1. **Creates print job** and shows system print dialog
2. **Configures custom margins** (50 units on all sides)
3. **Sets portrait orientation** with proper page layout
4. **Handles pagination** for multiple appointments
5. **Provides user feedback** on success/failure

## Pagination Implementation

Large datasets are split across multiple pages:

```java
private boolean printAppointmentsWithPagination(PrinterJob printerJob, PageLayout pageLayout) {
    int appointmentsPerPage = 10;
    int totalAppointments = appointmentManager.getAppointments().size();
    int totalPages = (int) Math.ceil((double) totalAppointments / appointmentsPerPage);
    
    for (int page = 0; page < totalPages; page++) {
        int startIndex = page * appointmentsPerPage;
        int endIndex = Math.min(startIndex + appointmentsPerPage, totalAppointments);
        
        VBox pageContent = createPageContent(startIndex, endIndex, page + 1, totalPages);
        
        // Content Scaling
        Scale scale = new Scale(0.8, 0.8);
        pageContent.getTransforms().add(scale);
        
        boolean pageSuccess = printerJob.printPage(pageContent);
        
        // Clean up
        pageContent.getTransforms().remove(scale);
        
        if (!pageSuccess) {
            return false;
        }
    }
    
    return true;
}
```

This algorithm:
1. **Calculates pages needed** based on 10 appointments per page
2. **Processes each page separately** with start/end indices
3. **Applies 80% scaling** to fit content properly
4. **Cleans up transforms** after each page to prevent memory leaks
5. **Returns false on any page failure** to stop the process

## Print Page Content Generation

Each page includes header information and appointment data:

```java
private VBox createPageContent(int startIndex, int endIndex, int currentPage, int totalPages) {
    VBox content = new VBox(8);
    
    // Page header
    Label header = new Label("APPOINTMENT MANAGER REPORT");
    header.setFont(Font.font("Arial", FontWeight.BOLD, 16));
    content.getChildren().add(header);
    
    Label groupInfo = new Label("GROUP 3 (CPE 121 FINAL PROJECT)");
    groupInfo.setFont(Font.font("Arial", 10));
    content.getChildren().add(groupInfo);

    Label dateTime = new Label("Generated: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    dateTime.setFont(Font.font("Arial", 10));
    content.getChildren().add(dateTime);
    
    Label pageInfo = new Label("Page " + currentPage + " of " + totalPages);
    pageInfo.setFont(Font.font("Arial", FontWeight.BOLD, 10));
    content.getChildren().add(pageInfo);
    
    // Separator
    Label separator = new Label("=".repeat(250));
    separator.setFont(Font.font("Arial", 10));
    content.getChildren().add(separator);
    
    // ... appointment data content ...
    
    return content;
}
```

Page content includes:
- **Report title** with bold formatting
- **Project attribution** information
- **Generation timestamp** with formatted date/time
- **Page numbering** (current/total)
- **Visual separator** using repeated characters

## File Operation Error Handling

All file operations include comprehensive error handling:

```java
if (appointmentManager.loadFromFile(filePath)) {
    updateStatusLabelWithSearch();
    statusLabel.setText("Appointments loaded from: " + file.getName());
} else {
    showAlert("Load Error", "Failed to load appointments from file.");
}
```

This pattern:
- **Uses boolean return values** from AppointmentManager methods
- **Updates UI on success** with confirmation messages
- **Shows error alerts** on failure with descriptive text
- **Maintains status label consistency** across all operations

## Save Operations Integration

The table view includes menu items for save operations:

```xml
<MenuItem mnemonicParsing="false" onAction="#saveAppointments" text="Save" />
<MenuItem mnemonicParsing="false" onAction="#saveAppointmentsAs" text="Save As..." />
```

These operations:
- **Delegate to AppointmentManager** for actual file writing
- **Maintain current file path** for normal saves
- **Show file chooser** for "Save As" operations
- **Update status feedback** based on operation results

## Print Dialog Integration

The print system uses the system's native print dialog:

```java
if (printerJob != null && printerJob.showPrintDialog(App.getPrimaryStage())) {
    // ... printing logic ...
} else {
    statusLabel.setText("Printing cancelled or no printer available.");
}
```

This provides:
- **Native OS print dialog** for printer selection
- **Print settings access** (copies, page range, etc.)
- **Graceful cancellation handling**
- **Printer availability checking**

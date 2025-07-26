# File Operations

This document explains how file loading works through both drag-and-drop and traditional file chooser.

## Drag and Drop Setup

The drag-and-drop functionality is initialized in the controller's `initialize()` method:

```java
private void setupDragAndDrop() {
    // Set up drag and drop functionality for the drop area
    dropArea.setOnDragOver(this::handleDragOver);
    dropArea.setOnDragDropped(this::handleDragDropped);
    dropArea.setOnDragEntered(this::handleDragEntered);
    dropArea.setOnDragExited(this::handleDragExited);
}
```

This code attaches event handlers to the `dropArea` BorderPane for each stage of the drag-and-drop operation.

## File Validation During Drag

When files are dragged over the drop area, the `handleDragOver` method validates them:

```java
private void handleDragOver(DragEvent event) {
    if (event.getGestureSource() != dropArea && event.getDragboard().hasFiles()) {
        // Check if the dragged file has .apf extension
        List<File> files = event.getDragboard().getFiles();
        if (!files.isEmpty() && files.get(0).getName().toLowerCase().endsWith(".apf")) {
            event.acceptTransferModes(TransferMode.COPY);
        }
    }
    event.consume();
}
```

This method:
1. Checks if the drag source is external (not the drop area itself)
2. Verifies that files are being dragged
3. **Only accepts .apf files** by checking the file extension
4. Sets transfer mode to `COPY` if valid

## Visual Feedback During Drag

The `handleDragEntered` method provides visual feedback based on file validity:

```java
private void handleDragEntered(DragEvent event) {
    if (event.getGestureSource() != dropArea && event.getDragboard().hasFiles()) {
        List<File> files = event.getDragboard().getFiles();
        if (!files.isEmpty() && files.get(0).getName().toLowerCase().endsWith(".apf")) {
            // Change appearance to indicate valid drop target
            dropArea.setStyle("-fx-background-color: #2a4d3a; -fx-background-radius: 5; -fx-border-color: #09ab72; -fx-border-width: 2px; -fx-border-radius: 5;");
            dropLabel.setText("Release to load file");
            dropLabel.setTextFill(javafx.scene.paint.Color.web("#09ab72"));
        } else {
            // Invalid file type
            dropArea.setStyle("-fx-background-color: #4d2a2a; -fx-background-radius: 5; -fx-border-color: #ab0909; -fx-border-width: 2px; -fx-border-radius: 5;");
            dropLabel.setText("Only .apf files allowed");
            dropLabel.setTextFill(javafx.scene.paint.Color.web("#ab0909"));
        }
    }
    event.consume();
}
```

This provides immediate visual feedback:
- **Green styling** for valid .apf files
- **Red styling** for invalid file types
- Dynamic text changes to guide the user

## File Drop Processing

When files are dropped, `handleDragDropped` processes them:

```java
private void handleDragDropped(DragEvent event) {
    Dragboard dragboard = event.getDragboard();
    boolean success = false;

    if (dragboard.hasFiles()) {
        List<File> files = dragboard.getFiles();
        if (!files.isEmpty()) {
            File file = files.get(0);
            if (file.getName().toLowerCase().endsWith(".apf")) {
                // Load the appointment file
                loadAppointmentFile(file);
                success = true;
            }
        }
    }

    event.setDropCompleted(success);
    event.consume();
    
    // Reset the drop area appearance
    resetDropAreaAppearance();
}
```

This method:
1. Extracts files from the dragboard
2. Processes only the first file if multiple are dropped
3. Double-checks the .apf extension
4. Calls `loadAppointmentFile()` for actual loading
5. Reports success/failure to the system
6. Resets the UI appearance

## Traditional File Chooser

The Load button opens a standard file dialog:

```java
@FXML
private void handleLoadButton() {
    FileChooser fileChooser = new FileChooser();
    fileChooser.setTitle("Load Appointments");
    fileChooser.getExtensionFilters().add(
        new FileChooser.ExtensionFilter("Appointment Files", "*.apf")
    );

    File file = fileChooser.showOpenDialog(App.getPrimaryStage());
    if (file != null) {
        loadAppointmentFile(file);
    }
}
```

This code:
1. Creates a `FileChooser` with a descriptive title
2. **Restricts file selection to .apf files only**
3. Shows the dialog modal to the primary stage
4. Processes the selected file if one was chosen

## Unified File Loading

Both drag-and-drop and file chooser use the same `loadAppointmentFile()` method:

```java
private void loadAppointmentFile(File file) {
    AppointmentManager appointmentManager = AppointmentManager.getInstance();
    if (appointmentManager.loadFromFile(file.getAbsolutePath())) {
        try {
            // Navigate to the table view after successful load
            App.setRoot("tableview");
        } catch (IOException e) {
            e.printStackTrace();
            System.err.println("Failed to load tableview.fxml");
        }
    } else {
        System.err.println("Failed to load appointments from file: " + file.getName());
    }
}
```

This ensures consistent behavior regardless of how the file was selected.

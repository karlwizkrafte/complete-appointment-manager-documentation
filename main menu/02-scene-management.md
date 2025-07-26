# Scene Management

This document explains how JavaFX scenes are managed and how navigation between views works.

## Scene Switching Architecture

The application uses a single `Stage` with scene switching rather than opening new windows:

```java
public class App extends Application {
    private static Scene scene;
    private static Stage primaryStage;
    
    static void setRoot(String fxml) throws IOException {
        scene.setRoot(loadFXML(fxml));
    }
}
```

The `setRoot()` method enables seamless navigation by replacing the scene's root node while keeping the same window.

## Navigation from Main Menu

When users interact with main menu buttons, the controller triggers scene changes:

```java
@FXML
private void handleNewButton() {
    try {
        // Navigate to the table view
        App.setRoot("tableview");
    } catch (IOException e) {
        e.printStackTrace();
        System.err.println("Failed to load tableview.fxml");
    }
}
```

This code:
1. Calls `App.setRoot("tableview")` to switch to the table view
2. Handles potential `IOException` if the FXML file can't be loaded
3. Prints error information to stderr for debugging

## File Loading Navigation

When a file is successfully loaded, the app automatically navigates to the table view:

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
        // You could show an error dialog here
    }
}
```

This method:
1. Gets the singleton `AppointmentManager` instance
2. Attempts to load appointments from the file
3. **Only navigates to table view if loading succeeds**
4. Logs errors if file loading or scene switching fails

## Stage Reference Management

The app maintains a static reference to the primary stage for global access:

```java
public static Stage getPrimaryStage() {
    return primaryStage;
}
```

This allows other parts of the application (like file dialogs) to have a parent window:

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

The `showOpenDialog(App.getPrimaryStage())` call ensures the file dialog is modal to the main window and positioned correctly.

## Memory Management

Since scenes are switched by replacing the root node, the previous UI components become eligible for garbage collection, preventing memory leaks from accumulating multiple scenes.

# Application Bootstrap

This document explains how the application starts up and initializes the main menu.

## Application Entry Point

The application starts in the `App.java` class which extends JavaFX's `Application`:

```java
public class App extends Application {
    private static Scene scene;
    private static Stage primaryStage;

    public static void main(String[] args) {
        launch();
    }
}
```

The `main()` method calls `launch()`, which triggers JavaFX to call the `start()` method.

## Stage Initialization

When the application starts, the `start()` method sets up the primary stage:

```java
@Override
public void start(Stage stage) throws IOException {
    primaryStage = stage;
    scene = new Scene(loadFXML("mainmenu"), 1200, 600);
    stage.setTitle("Appointment Manager");
    
    Image icon = new Image(getClass().getResourceAsStream("/cpe121/group3/assets/Appointment-Manager-Logo.png"));
    stage.getIcons().add(icon);
    
    stage.initStyle(StageStyle.UNDECORATED); // disable native title bar
    stage.setScene(scene);
    stage.show();
}
```

This code does several key things:
1. Stores the stage reference globally for later access
2. Creates a scene by loading the "mainmenu" FXML file with dimensions 1200x600
3. Sets the window title and icon
4. **Disables the native title bar** using `StageStyle.UNDECORATED`
5. Shows the window

## FXML Loading Mechanism

The `loadFXML()` method handles loading FXML files:

```java
private static Parent loadFXML(String fxml) throws IOException {
    FXMLLoader fxmlLoader = new FXMLLoader(App.class.getResource(fxml + ".fxml"));
    return fxmlLoader.load();
}
```

This method:
- Creates an `FXMLLoader` instance
- Loads the FXML file from the resources directory
- Returns the loaded UI component tree as a `Parent` node
- Automatically appends ".fxml" to the filename

## Controller Initialization

When the FXML is loaded, JavaFX automatically instantiates the controller specified in the FXML:

```xml
<BorderPane ... fx:controller="cpe121.group3.MainMenuController">
```

The controller's `initialize()` method runs automatically after FXML loading:

```java
@FXML
private void initialize() {
    LocalTime localTime = LocalTime.now();
    int hour = localTime.getHour();
    String greeting;

    if (hour >= 5 && hour < 12) {
        greeting = "Good morning";
    } else if (hour >= 12 && hour < 18) {
        greeting = "Good afternoon";
    } else {
        greeting = "Good evening";
    }
    
    greetLabel.setText(greeting);

    setupWindowDragging();
    setupDragAndDrop();
}
```

This initialization:
1. Calculates a time-based greeting
2. Sets up window dragging functionality
3. Configures drag-and-drop file handling

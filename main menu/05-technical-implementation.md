# 🔧 Technical Implementation

## Code Structure Overview

The Main Menu implementation consists of three primary files that work together to create a cohesive user experience.

## 📁 File Organization

```
Main Menu Implementation:
├── MainMenuController.java     (231 lines) - Business logic
├── mainmenu.fxml              (136 lines) - UI layout  
└── mainmenuButtonStyle.css     (External) - Styling
```

## 🎮 Controller Implementation (`MainMenuController.java`)

### Class Structure & Dependencies

```java
package cpe121.group3;

// Core JavaFX imports
import javafx.application.Platform;
import javafx.fxml.FXML;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.layout.HBox;
import javafx.scene.layout.BorderPane;

// Event handling imports
import javafx.scene.input.MouseEvent;
import javafx.scene.input.DragEvent;
import javafx.scene.input.Dragboard;
import javafx.scene.input.TransferMode;

// File management imports
import javafx.stage.FileChooser;
import javafx.stage.Stage;

// Utility imports
import java.io.IOException;
import java.io.File;
import java.time.LocalTime;
import java.util.List;
```

### 🏗️ Field Declarations

#### FXML Injected Components
```java
@FXML private Label greetLabel;           // Dynamic greeting display
@FXML private Label dropLabel;            // Drag & drop instruction text
@FXML private BorderPane dropArea;        // Drag & drop zone container
@FXML private Button newButton;           // "Open Table" primary action
@FXML private Button exitButton;          // Application exit
@FXML private Button loadButton;          // File browser trigger
@FXML private HBox titleBar;              // Custom window title bar
```

#### State Management Variables
```java
private double xOffset = 0;               // Window drag X coordinate
private double yOffset = 0;               // Window drag Y coordinate
```

### 🚀 Initialization Logic

#### Core Initialization Method
```java
@FXML
private void initialize() {
    // 1. Set up dynamic greeting
    LocalTime localTime = LocalTime.now();
    int hour = localTime.getHour();
    String greeting = calculateGreeting(hour);
    greetLabel.setText(greeting);
    
    // 2. Configure window functionality
    setupWindowDragging();
    
    // 3. Configure drag & drop
    setupDragAndDrop();
}
```

#### Greeting Calculation Logic
```java
Time-Based Greeting Algorithm:
├── if (hour >= 5 && hour < 12)  → "Good morning"
├── if (hour >= 12 && hour < 18) → "Good afternoon"  
└── else                         → "Good evening"

Implementation:
- Uses LocalTime.now() for current system time
- Hour extraction: localTime.getHour()
- Immediate UI update: greetLabel.setText(greeting)
```

### 🪟 Window Management Implementation

#### Window Dragging Setup
```java
private void setupWindowDragging() {
    titleBar.setOnMousePressed(this::handleMousePressed);
    titleBar.setOnMouseDragged(this::handleMouseDragged);
}

private void handleMousePressed(MouseEvent event) {
    // Capture initial click coordinates
    xOffset = event.getSceneX();
    yOffset = event.getSceneY();
}

private void handleMouseDragged(MouseEvent event) {
    // Calculate new window position
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setX(event.getScreenX() - xOffset);
    stage.setY(event.getScreenY() - yOffset);
}
```

#### Window Control Methods
```java
@FXML private void minimizeWindow() {
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setIconified(true);
}

@FXML private void maximizeWindow() {
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setMaximized(!stage.isMaximized());
}

@FXML private void closeWindow() {
    Platform.exit();  // Graceful JavaFX shutdown
}
```

### 📥 Drag & Drop Implementation

#### Event Handler Registration
```java
private void setupDragAndDrop() {
    dropArea.setOnDragOver(this::handleDragOver);
    dropArea.setOnDragDropped(this::handleDragDropped);
    dropArea.setOnDragEntered(this::handleDragEntered);
    dropArea.setOnDragExited(this::handleDragExited);
}
```

#### Drag Validation Logic
```java
private void handleDragOver(DragEvent event) {
    if (event.getGestureSource() != dropArea && 
        event.getDragboard().hasFiles()) {
        
        List<File> files = event.getDragboard().getFiles();
        if (!files.isEmpty() && 
            files.get(0).getName().toLowerCase().endsWith(".apf")) {
            event.acceptTransferModes(TransferMode.COPY);
        }
    }
    event.consume();
}
```

#### Visual Feedback System
```java
private void handleDragEntered(DragEvent event) {
    if (isValidDragSource(event)) {
        List<File> files = event.getDragboard().getFiles();
        
        if (isValidApfFile(files.get(0))) {
            // Valid file feedback
            dropArea.setStyle("""
                -fx-background-color: #2a4d3a; 
                -fx-background-radius: 5; 
                -fx-border-color: #09ab72; 
                -fx-border-width: 2px; 
                -fx-border-radius: 5;
            """);
            dropLabel.setText("Release to load file");
            dropLabel.setTextFill(Color.web("#09ab72"));
        } else {
            // Invalid file feedback  
            dropArea.setStyle("""
                -fx-background-color: #4d2a2a; 
                -fx-background-radius: 5; 
                -fx-border-color: #ab0909; 
                -fx-border-width: 2px; 
                -fx-border-radius: 5;
            """);
            dropLabel.setText("Only .apf files allowed");
            dropLabel.setTextFill(Color.web("#ab0909"));
        }
    }
    event.consume();
}
```

#### File Processing Logic
```java
private void handleDragDropped(DragEvent event) {
    Dragboard dragboard = event.getDragboard();
    boolean success = false;

    if (dragboard.hasFiles()) {
        List<File> files = dragboard.getFiles();
        if (!files.isEmpty()) {
            File file = files.get(0);
            if (file.getName().toLowerCase().endsWith(".apf")) {
                loadAppointmentFile(file);
                success = true;
            }
        }
    }

    event.setDropCompleted(success);
    event.consume();
    resetDropAreaAppearance();
}
```

### 🎯 Action Handlers

#### Navigation Actions
```java
@FXML private void handleNewButton() {
    try {
        App.setRoot("tableview");
    } catch (IOException e) {
        e.printStackTrace();
        System.err.println("Failed to load tableview.fxml");
    }
}

@FXML private void handleLoadButton() {
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

@FXML private void handleExitButton() {
    Platform.exit();
}
```

#### File Loading Logic
```java
private void loadAppointmentFile(File file) {
    AppointmentManager appointmentManager = AppointmentManager.getInstance();
    
    if (appointmentManager.loadFromFile(file.getAbsolutePath())) {
        try {
            App.setRoot("tableview");
        } catch (IOException e) {
            e.printStackTrace();
            System.err.println("Failed to load tableview.fxml");
        }
    } else {
        System.err.println("Failed to load appointments from file: " + 
                          file.getName());
    }
}
```

## 🎨 FXML Layout Implementation (`mainmenu.fxml`)

### Root Container Structure
```xml
<BorderPane 
    prefHeight="600.0" 
    prefWidth="1000.0" 
    style="-fx-background-color: #151515;" 
    stylesheets="@style/mainmenuButtonStyle.css"
    fx:controller="cpe121.group3.MainMenuController">
```

### Title Bar Implementation
```xml
<HBox fx:id="titleBar" 
      onMouseDragged="#onTitleBarDragged" 
      onMousePressed="#onTitleBarPressed"
      style="-fx-background-color: #151515;">
    
    <!-- App Logo -->
    <ImageView fitHeight="26.0" fitWidth="23.0">
        <Image url="@assets/Appointment-Manager-Logo.png" />
    </ImageView>
    
    <!-- App Title -->
    <Label text="Appointment Manager" textFill="WHITE" />
    
    <!-- Spacer -->
    <Region HBox.hgrow="ALWAYS" />
    
    <!-- Window Controls -->
    <HBox alignment="CENTER_RIGHT">
        <Button fx:id="minimizeButton" 
                onAction="#minimizeWindow" 
                text="–" />
        <Button fx:id="maximizeButton" 
                onAction="#maximizeWindow" 
                text="□" />
        <Button fx:id="closeButton" 
                onAction="#closeWindow" 
                text="×" />
    </HBox>
</HBox>
```

### Sidebar Button Layout
```xml
<VBox prefHeight="540.0" prefWidth="214.0" 
      style="-fx-background-color: #202020;">
    
    <!-- Primary Action Button -->
    <Button fx:id="newButton" 
            onAction="#handleNewButton"
            text="Open table"
            style="-fx-background-color: #09ab72; -fx-background-radius: 15;" />
    
    <!-- Secondary Action Button -->
    <Button onAction="#handleLoadButton"
            text="Load from file"
            style="-fx-background-color: white; -fx-background-radius: 50;" />
    
    <!-- Separator -->
    <Separator />
    
    <!-- Exit Button (Bottom Aligned) -->
    <VBox alignment="BOTTOM_LEFT">
        <Button fx:id="exitButton" 
                onAction="#handleExitButton"
                text="Exit" />
    </VBox>
</VBox>
```

### Drag & Drop Zone
```xml
<BorderPane fx:id="dropArea" 
            style="-fx-background-color: #252525; -fx-background-radius: 5;">
    <center>
        <Label fx:id="dropLabel" 
               text="Drop and drop .apf files here" 
               textFill="#6b6b6b" />
    </center>
</BorderPane>
```

## 🔧 Key Technical Patterns

### 1. **FXML Injection Pattern**
```java
@FXML annotation automatically injects FXML components
Controller methods bound via onAction attributes
Automatic lifecycle management by JavaFX framework
```

### 2. **Event-Driven Architecture**
```java
Mouse events for window management
Drag events for file operations  
Button events for navigation
Lifecycle events for initialization
```

### 3. **Singleton Integration**
```java
AppointmentManager.getInstance() provides data access
App.setRoot() manages scene transitions
Shared state across application components
```

### 4. **Error Handling Strategy**
```java
Try-catch blocks for I/O operations
Console logging for debugging
Graceful degradation on failures
User experience preservation
```

### 5. **Resource Management**
```java
External CSS for styling separation
Image resources via classpath URLs
Proper event listener cleanup
Memory-efficient scene transitions
```

---
**Next:** [🎯 User Experience](./06-user-experience.md) - Understand the UX design principles and user flows

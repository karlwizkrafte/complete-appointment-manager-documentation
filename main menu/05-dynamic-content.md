# Dynamic Content

This document explains how the main menu generates dynamic content based on system state.

## Time-Based Greeting System

The main menu displays a time-sensitive greeting that updates based on the current hour:

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
    // ... other initialization
}
```

This logic works as follows:
- **5:00-11:59**: "Good morning"
- **12:00-17:59**: "Good afternoon"  
- **18:00-4:59**: "Good evening" (includes late night/early morning)

The greeting is calculated once when the controller initializes, not continuously updated.

## Drop Area State Management

The drop area dynamically changes appearance based on drag-and-drop interactions:

### Default State
```java
private void resetDropAreaAppearance() {
    dropArea.setStyle("-fx-background-color: #252525; -fx-background-radius: 5;");
    dropLabel.setText("Drop .apf here");
    dropLabel.setTextFill(javafx.scene.paint.Color.web("#686868"));
}
```

This provides a neutral gray appearance with instructional text.

### Valid File Hover State
```java
// Change appearance to indicate valid drop target
dropArea.setStyle("-fx-background-color: #2a4d3a; -fx-background-radius: 5; -fx-border-color: #09ab72; -fx-border-width: 2px; -fx-border-radius: 5;");
dropLabel.setText("Release to load file");
dropLabel.setTextFill(javafx.scene.paint.Color.web("#09ab72"));
```

This creates a green theme indicating the file can be dropped.

### Invalid File Hover State
```java
// Invalid file type
dropArea.setStyle("-fx-background-color: #4d2a2a; -fx-background-radius: 5; -fx-border-color: #ab0909; -fx-border-width: 2px; -fx-border-radius: 5;");
dropLabel.setText("Only .apf files allowed");
dropLabel.setTextFill(javafx.scene.paint.Color.web("#ab0909"));
```

This creates a red theme warning about invalid file types.

## UI State Transitions

The drop area state transitions are triggered by drag events:

```java
private void handleDragEntered(DragEvent event) {
    if (event.getGestureSource() != dropArea && event.getDragboard().hasFiles()) {
        List<File> files = event.getDragboard().getFiles();
        if (!files.isEmpty() && files.get(0).getName().toLowerCase().endsWith(".apf")) {
            // Valid file - show green state
        } else {
            // Invalid file - show red state
        }
    }
    event.consume();
}

private void handleDragExited(DragEvent event) {
    resetDropAreaAppearance();
    event.consume();
}
```

The state automatically resets when:
- The drag operation exits the drop area
- A file is successfully dropped
- The drag operation completes

## Style Application Method

All dynamic styling uses inline CSS applied via the `setStyle()` method:

```java
dropArea.setStyle("-fx-background-color: #2a4d3a; -fx-background-radius: 5; -fx-border-color: #09ab72; -fx-border-width: 2px; -fx-border-radius: 5;");
```

This approach allows runtime style changes without modifying external CSS files. The inline styles override any styles defined in the external stylesheet.

## Text Content Updates

Dynamic text changes are applied to the `dropLabel`:

```java
dropLabel.setText("Release to load file");
dropLabel.setTextFill(javafx.scene.paint.Color.web("#09ab72"));
```

This pattern:
1. Updates the label text content
2. Changes the text color using `Color.web()` with hex color codes
3. Provides immediate visual feedback to users

The `Color.web()` method allows CSS-style hex color specification in Java code, maintaining consistency with the inline CSS styling approach.

# 🚀 Getting Started with Main Menu Development

## Developer Quick Start

This guide helps developers quickly understand, modify, and extend the Main Menu component of the Appointment Manager application.

## 🏃‍♂️ 5-Minute Setup

### Prerequisites Check
```bash
# Verify Java installation
java -version  # Requires Java 11+

# Verify Maven installation  
mvn -version   # Requires Maven 3.6+

# Verify JavaFX availability
javac --module-path /path/to/javafx/lib --add-modules javafx.controls,javafx.fxml
```

### Project Setup
```bash
# Clone or navigate to project
cd appointmentmanager

# Build the project
mvn clean compile

# Run the application (starts with main menu)
mvn javafx:run
```

### Verify Setup
✅ Application opens showing the main menu  
✅ Dark theme with custom title bar  
✅ "Good [morning/afternoon/evening]" greeting displays  
✅ Three buttons: "Open table", "Load from file", "Exit"  
✅ Drag & drop area in center shows "Drop and drop .apf files here"

## 📁 Key Files for Main Menu

### Essential Files
```
Main Menu Implementation:
├── src/main/java/cpe121/group3/
│   └── MainMenuController.java          # Core logic (231 lines)
├── src/main/resources/cpe121/group3/
│   ├── mainmenu.fxml                    # UI layout (136 lines)
│   └── style/mainmenuButtonStyle.css    # Styling
└── Complete Project Documentation/main menu/
    └── [Documentation files]            # This documentation
```

### Important Dependencies
```java
// In MainMenuController.java
import javafx.application.Platform;      // For application lifecycle
import javafx.scene.input.DragEvent;     // For drag & drop functionality
import javafx.stage.FileChooser;         // For file browsing
import java.time.LocalTime;              # For dynamic greeting
```

## 🛠️ Common Development Tasks

### 1. Modify the Greeting Logic
**Location:** `MainMenuController.java` lines 44-54

```java
// Current implementation
private String calculateGreeting(int hour) {
    if (hour >= 5 && hour < 12) {
        return "Good morning";
    } else if (hour >= 12 && hour < 18) {
        return "Good afternoon";
    } else {
        return "Good evening";
    }
}

// Example modification: Add username
private String calculateGreeting(int hour, String username) {
    String timeGreeting = /* existing logic */;
    return timeGreeting + ", " + username;
}
```

### 2. Add New Navigation Button
**Step 1:** Update FXML (`mainmenu.fxml`)
```xml
<!-- Add after existing buttons in sidebar VBox -->
<Button fx:id="settingsButton" 
        onAction="#handleSettingsButton"
        prefHeight="35.0" prefWidth="120.0"
        text="Settings">
    <VBox.margin>
        <Insets left="20.0" top="10.0" />
    </VBox.margin>
</Button>
```

**Step 2:** Update Controller (`MainMenuController.java`)
```java
// Add field declaration
@FXML private Button settingsButton;

// Add event handler
@FXML
private void handleSettingsButton() {
    try {
        App.setRoot("settings");  // Assumes settings.fxml exists
    } catch (IOException e) {
        e.printStackTrace();
        System.err.println("Failed to load settings.fxml");
    }
}
```

### 3. Customize Drag & Drop File Types
**Location:** `MainMenuController.java` lines 78-86

```java
// Current: Only .apf files
if (file.getName().toLowerCase().endsWith(".apf")) {
    event.acceptTransferModes(TransferMode.COPY);
}

// Modified: Multiple file types
String fileName = file.getName().toLowerCase();
if (fileName.endsWith(".apf") || 
    fileName.endsWith(".csv") || 
    fileName.endsWith(".json")) {
    event.acceptTransferModes(TransferMode.COPY);
}
```

### 4. Change Color Scheme
**Location:** `mainmenu.fxml` inline styles and `mainmenuButtonStyle.css`

```css
/* Current primary green: #09ab72 */
/* Change to blue theme */
-fx-background-color: #0972ab;  /* Blue instead of green */

/* Update drag & drop feedback colors */
/* In MainMenuController.java handleDragEntered() */
dropArea.setStyle("-fx-background-color: #2a3d4d; -fx-border-color: #0972ab;");
dropLabel.setTextFill(Color.web("#0972ab"));
```

### 5. Add Keyboard Shortcuts
**Step 1:** Update FXML to capture key events
```xml
<BorderPane onKeyPressed="#handleKeyPressed" focusTraversable="true">
```

**Step 2:** Add handler in Controller
```java
@FXML
private void handleKeyPressed(KeyEvent event) {
    switch (event.getCode()) {
        case ENTER:
        case SPACE:
            handleNewButton();  // Open table on Enter/Space
            break;
        case O:
            if (event.isControlDown()) {
                handleLoadButton();  // Ctrl+O for open file
            }
            break;
        case ESCAPE:
            handleExitButton();  // Escape to exit
            break;
    }
    event.consume();
}
```

## 🧪 Testing Your Changes

### 1. Visual Testing
```bash
# Run application after changes
mvn clean compile javafx:run

# Test checklist:
□ Application launches without errors
□ New UI elements display correctly
□ Color changes applied as expected
□ Buttons respond to clicks
□ Drag & drop behavior works
□ Window controls function properly
```

### 2. Functional Testing
```bash
# Test navigation
□ "Open Table" → Navigate to table view
□ "Load from file" → File chooser opens
□ "Exit" → Application closes gracefully

# Test drag & drop
□ Drag .apf file → Green feedback, successful load
□ Drag other file → Red feedback, rejection
□ Drag multiple files → Only first file processed

# Test window controls
□ Title bar drag → Window moves
□ Minimize → Window to taskbar
□ Maximize → Window fills screen
□ Close → Application exits
```

### 3. Error Testing
```bash
# Test error conditions
□ Missing FXML file → Error logged, app stable
□ Corrupt .apf file → Error logged, stay on menu
□ Network drive file → Proper handling
□ Very large file → Performance acceptable
```

## 🔧 Debugging Tips

### Common Issues & Solutions

#### 1. FXML Loading Errors
```
Error: "Location is not set"
Solution: Check fx:controller attribute in FXML matches class name
Solution: Verify FXML file is in correct resources location
```

#### 2. Button Actions Not Working
```
Error: "No matching method found"
Solution: Check onAction="#methodName" matches @FXML method name
Solution: Ensure method is public and has @FXML annotation
```

#### 3. Drag & Drop Not Responding
```
Error: Events not firing
Solution: Check event handler registration in setupDragAndDrop()
Solution: Verify dropArea is properly injected with fx:id
```

#### 4. Styling Not Applied
```
Error: Styles not visible
Solution: Check CSS file path in stylesheets="@style/..."
Solution: Verify CSS syntax and selectors
```

### Debugging Tools
```java
// Add debug logging to controller methods
System.out.println("Button clicked: " + buttonName);
System.out.println("File dropped: " + file.getName());
System.out.println("Navigation to: " + sceneName);

// Use JavaFX Scene Builder for FXML editing
// Download: https://gluonhq.com/products/scene-builder/

// Use IDE debugging
// Set breakpoints in event handlers
// Step through drag & drop logic
```

## 📚 Learning Resources

### JavaFX Documentation
- [JavaFX FXML Guide](https://openjfx.io/javadoc/11/javafx.fxml/javafx/fxml/doc-files/introduction_to_fxml.html)
- [JavaFX CSS Reference](https://openjfx.io/javadoc/11/javafx.graphics/javafx/scene/doc-files/cssref.html)
- [Drag and Drop Guide](https://docs.oracle.com/javafx/2/drag_drop/jfxpub-drag_drop.htm)

### Related Project Documentation
- [📋 Overview](./01-overview.md) - High-level understanding
- [🏗️ Architecture](./02-architecture.md) - System design patterns
- [🔧 Technical Implementation](./05-technical-implementation.md) - Detailed code structure

### Development Best Practices
```java
Code Style Guidelines:
├── Use @FXML for all injected components
├── Keep event handlers simple (delegate complex logic)
├── Handle exceptions gracefully (don't crash on errors)
├── Use descriptive method and variable names
├── Comment complex logic (especially drag & drop)
└── Follow JavaFX threading rules (UI updates on FX thread)
```

## 🎯 Next Steps

### Immediate Tasks
1. **Familiarize** with existing code structure
2. **Test** the current implementation thoroughly
3. **Make** a small change to verify your setup
4. **Review** the integration points with other components

### Medium-term Goals
1. **Enhance** the user interface based on user feedback
2. **Optimize** performance for file loading operations
3. **Add** new features like recent files list
4. **Implement** additional keyboard shortcuts

### Long-term Vision
1. **Integrate** with external systems (cloud storage, databases)
2. **Develop** plugin architecture for extensibility
3. **Create** automated testing suite
4. **Design** accessibility improvements

---
*You're now ready to work with the Main Menu component! Start with small changes and gradually work up to more complex modifications.*

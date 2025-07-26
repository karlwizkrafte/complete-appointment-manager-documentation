# Window Controls

This document explains how custom window controls work since the native title bar is disabled.

## Custom Title Bar Setup

Since the application uses `StageStyle.UNDECORATED`, it needs custom window controls:

```java
stage.initStyle(StageStyle.UNDECORATED); // disable native title bar
```

The FXML defines a custom title bar with window control buttons:

```xml
<HBox fx:id="titleBar" onMouseDragged="#onTitleBarDragged" onMousePressed="#onTitleBarPressed" 
      prefHeight="60.0" prefWidth="200.0" style="-fx-background-color: #151515;">
   <children>
      <ImageView fitHeight="26.0" fitWidth="23.0" pickOnBounds="true" preserveRatio="true">
         <image>
            <Image url="@assets/Appointment-Manager-Logo.png" />
         </image>
      </ImageView>
      <Label text="Appointment Manager" textFill="WHITE" />
      <Region prefHeight="40.0" prefWidth="50.0" HBox.hgrow="ALWAYS" />
      <HBox alignment="CENTER_RIGHT" spacing="0.0">
         <children>
            <Button fx:id="minimizeButton" onAction="#minimizeWindow" text="–" />
            <Button fx:id="maximizeButton" onAction="#maximizeWindow" text="□" />
            <Button fx:id="closeButton" onAction="#closeWindow" text="×" />
         </children>
      </HBox>
   </children>
</HBox>
```

## Window Dragging Implementation

The controller sets up window dragging during initialization:

```java
private void setupWindowDragging() {
    // Set up drag functionality for the title bar
    titleBar.setOnMousePressed(this::handleMousePressed);
    titleBar.setOnMouseDragged(this::handleMouseDragged);
}
```

Mouse press events record the initial click position:

```java
private void handleMousePressed(MouseEvent event) {
    // Record the initial click position
    xOffset = event.getSceneX();
    yOffset = event.getSceneY();
}
```

Mouse drag events move the window:

```java
private void handleMouseDragged(MouseEvent event) {
    // Get the stage and move it based on mouse movement
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setX(event.getScreenX() - xOffset);
    stage.setY(event.getScreenY() - yOffset);
}
```

This implementation:
1. **Records the click offset** relative to the scene
2. **Calculates new window position** using screen coordinates minus the offset
3. **Maintains smooth dragging** by preserving the relative click position

## Window Control Buttons

Each window control button has a specific function:

### Minimize Button
```java
@FXML
private void minimizeWindow() {
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setIconified(true);
}
```

This minimizes the window to the taskbar.

### Maximize/Restore Button
```java
@FXML
private void maximizeWindow() {
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setMaximized(!stage.isMaximized());
}
```

This toggles between maximized and restored window states.

### Close Button
```java
@FXML
private void closeWindow() {
    Platform.exit();
}
```

This properly shuts down the JavaFX application.

## Alternative Event Handlers

The FXML also defines event handlers directly on the title bar:

```java
@FXML
private void onTitleBarPressed(MouseEvent event) {
    xOffset = event.getSceneX();
    yOffset = event.getSceneY();
}

@FXML
private void onTitleBarDragged(MouseEvent event) {
    Stage stage = (Stage) titleBar.getScene().getWindow();
    stage.setX(event.getScreenX() - xOffset);
    stage.setY(event.getScreenY() - yOffset);
}
```

These are duplicates of the programmatically set handlers, demonstrating two approaches to event handling in JavaFX.

## Stage Reference Access

All window control methods access the stage through the scene graph:

```java
Stage stage = (Stage) titleBar.getScene().getWindow();
```

This pattern:
1. Gets the scene from any UI component
2. Gets the window from the scene
3. Casts to Stage for window operations

This approach works from any controller method without needing to store stage references.

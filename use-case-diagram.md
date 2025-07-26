# Use Case Diagram

This document presents the use case diagram for the Appointment Manager application, showing the interactions between users and the system.

## Use Case Diagram

```mermaid
graph TB
    %% Define the User actor
    User((User))
    
    %% Main Menu Operations (Top-left cluster)
    subgraph MainMenu["🏠 Main Menu Operations"]
        LaunchApp[Launch Application]
        CreateFile[Create New File]
        LoadFile[Load Existing File]
        DragDrop[Drag & Drop File]
        Navigate[Navigate to Table View]
        ExitApp[Exit Application]
    end
    
    %% Appointment Management (Top-right cluster)
    subgraph AppMgmt["📊 Appointment Management"]
        ViewAppts[View Appointments]
        AddAppt[Add New Appointment]
        EditAppt[Edit Appointment]
        DeleteAppt[Delete Appointment]
        UpdateStatus[Update Status]
    end
    
    %% Search & Filter Operations (Bottom-left cluster)
    subgraph SearchFilter["🔍 Search & Filter"]
        SearchAppts[Search Appointments]
        AdvFilters[Apply Advanced Filters]
        ClearFilters[Clear Filters]
        RefreshTable[Refresh Table]
    end
    
    %% File Operations (Bottom-center cluster)
    subgraph FileOps["💾 File Operations"]
        SaveAppts[Save Appointments]
        SaveAsNew[Save As New File]
        PrintAppts[Print Appointments]
        ExportData[Export Data]
    end
    
    %% Window Management (Bottom-right cluster)
    subgraph WindowMgmt["🪟 Window Management"]
        MinWindow[Minimize Window]
        MaxWindow[Maximize Window]
        CloseWindow[Close Window]
        DragWindow[Drag Window]
    end
    
    %% User connections to individual use cases
    User --- LaunchApp
    User --- CreateFile
    User --- LoadFile
    User --- DragDrop
    User --- Navigate
    User --- ExitApp
    
    User --- ViewAppts
    User --- AddAppt
    User --- EditAppt
    User --- DeleteAppt
    User --- UpdateStatus
    
    User --- SearchAppts
    User --- AdvFilters
    User --- ClearFilters
    User --- RefreshTable
    
    User --- SaveAppts
    User --- SaveAsNew
    User --- PrintAppts
    User --- ExportData
    
    User --- MinWindow
    User --- MaxWindow
    User --- CloseWindow
    User --- DragWindow
    
    %% Include relationships (auto-save functionality)
    AddAppt -->|includes| SaveAppts
    EditAppt -->|includes| SaveAppts
    DeleteAppt -->|includes| SaveAppts
    UpdateStatus -->|includes| SaveAppts
    
    %% Extend relationships
    DragDrop -.->|extends| LoadFile
    SaveAsNew -.->|extends| SaveAppts
    
    %% Navigation flow between major functions
    LaunchApp --> Navigate
    Navigate --> ViewAppts
    LoadFile --> ViewAppts
    DragDrop --> ViewAppts
    CreateFile --> ViewAppts
    
    %% Styling
    classDef actor fill:#e1f5fe,stroke:#01579b,stroke-width:3px,font-weight:bold
    classDef usecase fill:#f3e5f5,stroke:#4a148c,stroke-width:1px
    classDef cluster fill:#f8f9fa,stroke:#6c757d,stroke-width:2px
    classDef include fill:#d4edda,stroke:#155724,stroke-width:2px
    classDef extend fill:#fff3cd,stroke:#856404,stroke-width:2px
    classDef navigate fill:#cce5ff,stroke:#0066cc,stroke-width:2px
    
    class User actor
    class LaunchApp,CreateFile,LoadFile,DragDrop,Navigate,ExitApp,ViewAppts,AddAppt,EditAppt,DeleteAppt,UpdateStatus,SearchAppts,AdvFilters,ClearFilters,RefreshTable,SaveAppts,SaveAsNew,PrintAppts,ExportData,MinWindow,MaxWindow,CloseWindow,DragWindow usecase
    class SaveAppts include
    class LoadFile,SaveAppts extend
    class Navigate,ViewAppts navigate
```

## Alternative Simplified View

```mermaid
mindmap
  root((User))
    🏠 Main Menu
      Launch App
      Create File
      Load File
      Drag & Drop
      Navigate
      Exit
    📊 Appointments
      View All
      Add New
      Edit Existing
      Delete
      Update Status
    🔍 Search & Filter
      Real-time Search
      Advanced Filters
      Clear Filters
      Refresh View
    💾 File Operations
      Save Current
      Save As New
      Print Reports
      Export Data
    🪟 Window Controls
      Minimize
      Maximize
      Close
      Drag Move
```

## System Flow Diagram

```mermaid
flowchart TD
    Start([User Starts App]) --> MainMenu{Main Menu}
    
    MainMenu -->|New| CreateFile[Create New File]
    MainMenu -->|Load| LoadFile[Load Existing File]
    MainMenu -->|Drag Drop| DragDrop[Drag & Drop File]
    
    CreateFile --> TableView[Table View]
    LoadFile --> TableView
    DragDrop --> TableView
    
    TableView --> ViewAppt[View Appointments]
    TableView --> Search[Search & Filter]
    TableView --> CRUD[Add/Edit/Delete]
    TableView --> FileOps[Save/Print/Export]
    
    CRUD --> AutoSave[Auto-Save]
    AutoSave --> TableView
    
    Search --> FilteredView[Filtered Results]
    FilteredView --> TableView
    
    FileOps --> Success{Operation Success?}
    Success -->|Yes| TableView
    Success -->|No| Error[Show Error]
    Error --> TableView
    
    TableView -->|Exit| End([Application Closes])
    MainMenu -->|Exit| End
    
    %% Styling
    classDef startEnd fill:#d4edda,stroke:#155724,stroke-width:2px
    classDef process fill:#cfe2ff,stroke:#0a58ca,stroke-width:2px
    classDef decision fill:#fff3cd,stroke:#856404,stroke-width:2px
    classDef error fill:#f8d7da,stroke:#721c24,stroke-width:2px
    
    class Start,End startEnd
    class MainMenu,Success decision
    class CreateFile,LoadFile,DragDrop,TableView,ViewAppt,Search,CRUD,FileOps,AutoSave,FilteredView process
    class Error error
```

## Use Case Categories

### 🏠 Main Menu Operations
- **Launch Application**: Start the appointment manager
- **Create New File**: Initialize a new .apf file
- **Load Existing File**: Open an existing .apf file via file chooser
- **Drag & Drop File**: Load file by dragging .apf file onto application
- **Navigate to Table View**: Switch from main menu to appointment table
- **Exit Application**: Close the application

### 📊 Appointment Management
- **View Appointments**: Display all appointments in table format
- **Add New Appointment**: Create a new appointment record
- **Edit Appointment**: Modify existing appointment details
- **Delete Appointment**: Remove an appointment from the system
- **Update Appointment Status**: Quick status change for appointments

### 🔍 Search & Filter Operations
- **Search Appointments**: Real-time text search across title and participant
- **Apply Advanced Filters**: Use complex filtering criteria
- **Clear Filters**: Remove all active filters
- **Refresh Table**: Manually refresh the appointment display

### 💾 File Operations
- **Save Appointments**: Save current appointments to file
- **Save As New File**: Save appointments to a new .apf file
- **Print Appointments**: Generate printed reports
- **Export Data**: Output appointment data

### 🪟 Window Management
- **Minimize Window**: Minimize application to taskbar
- **Maximize Window**: Toggle between windowed and fullscreen
- **Close Window**: Terminate the application
- **Drag Window**: Move window by dragging title bar

## Use Case Relationships

### Include Relationships
- **Appointment CRUD operations** → **Auto-save**: All data modifications automatically trigger save operations
- This ensures data persistence without explicit user action

### Extend Relationships
- **Drag & Drop File** → **Load Existing File**: Drag & drop provides an alternative method to load files
- **Save As New File** → **Save Appointments**: Save As extends basic save with file chooser functionality

## Primary Actor
- **User**: The person using the appointment manager application
- Single user system (no multi-user scenarios)
- No authentication or authorization requirements

## System Boundary
The system includes:
- JavaFX user interface components
- Data management and persistence layer
- File I/O operations
- Print functionality

The system excludes:
- Network operations
- External system integration
- Multi-user capabilities
- Cloud storage

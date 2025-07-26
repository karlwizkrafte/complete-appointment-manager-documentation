# Appointment Manager - Complete Technical Documentation

This repository contains comprehensive technical documentation explaining how the Appointment Manager works under the hood - actual code implementation, data flow, and technical mechanisms.

## 📊 System Overview Diagrams

- [📋 Use Case Diagram](./use-case-diagram.md) - User interactions and system functionality
- [🗄️ Entity Relationship Diagram](./entity-relationship-diagram.md) - Data structure and relationships

## 🏠 Main Menu Technical Documentation

- [🔧 Application Bootstrap](./main%20menu/01-application-bootstrap.md) - How the app starts and initializes
- [🎯 Scene Management](./main%20menu/02-scene-management.md) - JavaFX scene loading and switching
- [📁 File Operations](./main%20menu/03-file-operations.md) - Drag-and-drop and file loading implementation
- [🖱️ Window Controls](./main%20menu/04-window-controls.md) - Custom title bar and window manipulation
- [⏰ Dynamic Content](./main%20menu/05-dynamic-content.md) - Time-based greetings and UI updates

## 📊 Table View Technical Documentation

- [🔧 Data Binding](./table%20view/01-data-binding.md) - How appointment data is bound to the TableView
- [🔍 Search and Filtering](./table%20view/02-search-filtering.md) - Real-time search and advanced filtering implementation
- [🪟 Popup Management](./table%20view/03-popup-management.md) - Modal dialog creation and lifecycle management
- [📊 Table Operations](./table%20view/04-table-operations.md) - CRUD operations and data manipulation
- [🖨️ Export and Print](./table%20view/05-export-print.md) - File operations and printing functionality

## 🗄️ Data Layer Technical Documentation

- [🏛️ Singleton Architecture](./data%20layer/01-singleton-architecture.md) - AppointmentManager singleton pattern and state management
- [🗄️ Database Integration](./data%20layer/02-database-integration.md) - SQLite database operations and .apf file format
- [📊 Data Model](./data%20layer/03-data-model.md) - JavaFX Properties pattern in Appointment class
- [💾 File Operations](./data%20layer/04-file-operations.md) - Save/load mechanisms and file lifecycle management

## 📝 Popup Controllers Technical Documentation

- [📝 Form Management](./popups/01-form-management.md) - PopupFormController add/edit mode and form lifecycle

## 🏗️ Architecture Overview

The Appointment Manager uses a layered architecture:

```
┌─────────────────────────────────────────┐
│           UI Layer (JavaFX)             │
│  MainMenu → TableView → Popup Dialogs   │
├─────────────────────────────────────────┤
│        Business Logic Layer             │
│     AppointmentManager (Singleton)      │
├─────────────────────────────────────────┤
│         Data Access Layer               │
│    DatabaseManager (SQLite Wrapper)     │
├─────────────────────────────────────────┤
│        Persistence Layer                │
│      .apf Files (SQLite Databases)      │
└─────────────────────────────────────────┘
```

## 🎯 Key Technical Concepts Covered

- **Singleton Pattern** - Single source of truth for application state
- **JavaFX Properties** - Automatic UI binding and change notification
- **Observer Pattern** - ObservableList for automatic UI updates
- **Modal Dialogs** - Popup management and controller communication
- **SQLite Integration** - Database operations and file persistence
- **Prepared Statements** - SQL injection prevention
- **Resource Management** - Memory and connection cleanup
- **Error Handling** - Graceful failure management

## 📚 Documentation Purpose

This documentation serves to:
- **Explain implementation details** with actual code snippets
- **Document design decisions** and architectural patterns
- **Enable efficient maintenance** and future enhancements
- **Facilitate developer onboarding** with deep technical insights
- **Preserve knowledge** about how the system works

## 🔧 Technical Focus

Each document includes:
- **Code snippets** showing actual implementation
- **Detailed explanations** of what each snippet does
- **Follow-up code** building on previous examples
- **Design patterns** and their applications
- **Error handling** and edge cases

---
*This documentation focuses on implementation details and code mechanics rather than user guides or high-level overviews.*

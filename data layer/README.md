# Data Layer Technical Documentation

This documentation explains how the core data management system works under the hood - the data model, singleton pattern, and persistence mechanisms that power the entire application.

## Technical Documentation

- [🏛️ Singleton Architecture](./01-singleton-architecture.md) - AppointmentManager singleton pattern and state management
- [🗄️ Database Integration](./02-database-integration.md) - SQLite database operations and .apf file format
- [📊 Data Model](./03-data-model.md) - JavaFX Properties pattern in Appointment class
- [💾 File Operations](./04-file-operations.md) - Save/load mechanisms and file lifecycle management

## Architecture Overview

The data layer uses a three-tier architecture:
1. **AppointmentManager (Singleton)** - Central data coordinator and API facade
2. **DatabaseManager** - SQLite database operations and SQL query execution  
3. **Appointment (Model)** - JavaFX Properties-based data model with automatic binding

## Data Flow

```
UI Layer (TableView, Forms)
        ↓ (ObservableList binding)
AppointmentManager (Singleton)
        ↓ (CRUD operations)
DatabaseManager (SQLite)
        ↓ (File I/O)
.apf Files (SQLite Database Files)
```

## Key Design Patterns

- **Singleton Pattern** - Single source of truth for appointment data
- **Observer Pattern** - ObservableList automatically updates UI components
- **Properties Pattern** - JavaFX StringProperty enables automatic data binding
- **Facade Pattern** - AppointmentManager simplifies database complexity

---
*Technical documentation focused on data persistence and state management mechanisms.*

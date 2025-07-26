# Table View Technical Documentation

This documentation explains how the Table View works under the hood - the actual code implementation, data flow, and technical mechanisms that power the appointment data management interface.

## Technical Documentation

- [🔧 Data Binding](./01-data-binding.md) - How appointment data is bound to the TableView
- [🔍 Search and Filtering](./02-search-filtering.md) - Real-time search and advanced filtering implementation
- [🪟 Popup Management](./03-popup-management.md) - Modal dialog creation and lifecycle management
- [📊 Table Operations](./04-table-operations.md) - CRUD operations and data manipulation
- [🖨️ Export and Print](./05-export-print.md) - File operations and printing functionality

## Architecture Overview

The table view operates as the main data management interface that:
1. Displays appointment data in a sortable, filterable TableView
2. Provides real-time search functionality with predicate-based filtering
3. Manages modal popup dialogs for data entry and editing
4. Handles file I/O operations for data persistence
5. Implements printing capabilities with custom page layouts

## Code Structure

```
TableViewController.java     → Main table logic and event handling
tableview.fxml              → UI layout with TableView and controls
PopupFormController.java    → Add/edit appointment dialog
FilterController.java       → Advanced filtering dialog
StatusUpdateController.java → Quick status update dialog
```

## Data Flow

```
AppointmentManager (Singleton)
        ↓ (ObservableList)
FilteredList<Appointment>
        ↓ (Predicate filtering)
TableView<Appointment>
        ↓ (User selection)
Modal Popup Dialogs
```

---
*Technical documentation focused on implementation details and code mechanics.*

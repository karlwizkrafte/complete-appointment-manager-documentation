# Popup Controllers Technical Documentation

This documentation explains how the popup dialog controllers work under the hood - form management, validation, and data binding mechanisms that power the appointment management dialogs.

## Technical Documentation

- [📝 Form Management](./01-form-management.md) - PopupFormController add/edit mode and form lifecycle
- [🔍 Filter Controller](./02-filter-controller.md) - Advanced filtering dialog and criteria management
- [⚡ Status Updates](./03-status-updates.md) - Quick status change dialog implementation

## Architecture Overview

The popup system uses modal dialogs with specialized controllers:
1. **PopupFormController** - Handles appointment creation and editing with validation
2. **FilterController** - Manages complex filtering criteria and predicate building
3. **StatusUpdateController** - Provides quick status modification interface

## Controller Communication Pattern

```
TableViewController (Parent)
        ↓ (Static data passing)
PopupController (Child)
        ↓ (Stage management)
Modal Dialog Window
        ↓ (User interaction)
Data Changes
        ↓ (Automatic updates)
ObservableList → TableView
```

## Key Design Patterns

- **Static State Pattern** - Controllers use static fields for data passing
- **Modal Dialog Pattern** - All popups block parent interaction
- **Validation Pattern** - Form validation before data persistence
- **Self-Closing Pattern** - Popups manage their own lifecycle

---
*Technical documentation focused on popup dialog implementation and form management.*

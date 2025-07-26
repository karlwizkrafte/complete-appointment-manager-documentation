# Search and Filtering

This document explains how real-time search and advanced filtering work in the table view.

## Real-Time Search Implementation

The search functionality uses a `FilteredList` with dynamic predicate updates:

```java
private void setupSearchFunctionality() {
    searchField.textProperty().addListener((observable, oldValue, newValue) -> {
        applyAllFilters();
    });
}
```

This code sets up a **property listener** that automatically triggers filtering whenever the search field text changes.

## Combined Filter Architecture

The filtering system combines search and advanced filters:

```java
private void applyAllFilters() {
    filteredAppointments.setPredicate(appointment -> {
        // First check search filter
        boolean matchesSearch = checkSearchFilter(appointment);
        
        // Then check advanced filters
        boolean matchesAdvancedFilters = checkAdvancedFilters(appointment);
        
        // Both must be true
        return matchesSearch && matchesAdvancedFilters;
    });
    
    // Update status label to show filtered results
    updateStatusLabelWithSearch();
}
```

This implementation:
1. **Creates a compound predicate** that requires both search and advanced filters to pass
2. **Updates the FilteredList predicate** which automatically refreshes the table
3. **Updates status labels** to reflect the new filtered state

## Search Filter Logic

The search filter performs case-insensitive partial matching:

```java
private boolean checkSearchFilter(Appointment appointment) {
    String searchText = searchField.getText();
    
    // If search field is empty, return true
    if (searchText == null || searchText.trim().isEmpty()) {
        return true;
    }
    
    searchText = searchText.toLowerCase().trim();
    
    // Search in title and participant fields (case-insensitive)
    String title = appointment.getTitle() != null ? appointment.getTitle().toLowerCase() : "";
    String participant = appointment.getParticipant() != null ? appointment.getParticipant().toLowerCase() : "";
    
    boolean matchesTitle = title.contains(searchText);
    boolean matchesParticipant = participant.contains(searchText);
    
    return matchesTitle || matchesParticipant;
}
```

This method:
1. **Handles empty search gracefully** by returning true (show all)
2. **Normalizes text to lowercase** for case-insensitive matching
3. **Null-safe string operations** to prevent exceptions
4. **Searches multiple fields** (title and participant) with OR logic

## Advanced Filter Processing

Advanced filters use the stored `FilterCriteria` object:

```java
private boolean checkAdvancedFilters(Appointment appointment) {
    // If no advanced filters are active, return true
    if (currentFilterCriteria == null || !currentFilterCriteria.hasActiveFilters()) {
        return true;
    }
    
    // Check each filter condition
    boolean matches = true;
    
    // Title filter
    if (currentFilterCriteria.isTitleFilterEnabled()) {
        String titleFilter = currentFilterCriteria.getTitleFilter();
        if (titleFilter != null && !titleFilter.isEmpty()) {
            String appointmentTitle = appointment.getTitle() != null ? appointment.getTitle() : "";
            matches = matches && matchesText(appointmentTitle, titleFilter, 
                currentFilterCriteria.isCaseSensitive(), currentFilterCriteria.isExactMatch());
        }
    }
    
    // ... other filter checks (participant, date range, status, description)
    
    return matches;
}
```

This creates a **cascading AND filter** where all enabled filters must match.

## Filter Persistence

The current filter state is maintained across operations:

```java
// Apply advanced filters to the table
public void applyAdvancedFilter(FilterCriteria criteria) {
    // Store the current filter criteria to make it persistent
    this.currentFilterCriteria = criteria;
    
    // Apply all filters (search + advanced)
    applyAllFilters();
}
```

This ensures:
- **Filters persist through table refreshes**
- **Search and advanced filters work together**
- **Filter state survives popup operations**

## Text Matching Strategies

The system supports different matching modes:

```java
private boolean matchesText(String text, String filter, boolean caseSensitive, boolean exactMatch) {
    if (text == null || filter == null) {
        return false;
    }
    
    String textToCheck = caseSensitive ? text : text.toLowerCase();
    String filterToCheck = caseSensitive ? filter : filter.toLowerCase();
    
    if (exactMatch) {
        return textToCheck.equals(filterToCheck);
    } else {
        return textToCheck.contains(filterToCheck);
    }
}
```

This provides:
- **Case-sensitive or case-insensitive** matching
- **Exact match or partial match** (contains) modes
- **Null-safe operations** to prevent exceptions

## Date Range Filtering

Date filtering uses string-based date comparison:

```java
// Date range filter
if (currentFilterCriteria.isDateRangeFilterEnabled()) {
    String appointmentDateStr = appointment.getAppointmentDate();
    if (appointmentDateStr != null && !appointmentDateStr.isEmpty()) {
        matches = matches && matchesDateRange(appointmentDateStr, 
            currentFilterCriteria.getFromDate(), currentFilterCriteria.getToDate());
    }
}
```

This handles:
- **Optional date filtering** (only when enabled)
- **Null-safe date access** 
- **Range-based date matching** between from and to dates

## Filter Status Tracking

The status label provides real-time filter feedback:

```java
private void updateStatusLabelWithSearch() {
    int filteredCount = filteredAppointments.size();
    int totalCount = appointmentManager.getAppointments().size();
    
    String statusText = "";
    boolean hasSearch = searchField.getText() != null && !searchField.getText().trim().isEmpty();
    boolean hasAdvancedFilters = currentFilterCriteria != null && currentFilterCriteria.hasActiveFilters();
    
    if (!hasSearch && !hasAdvancedFilters) {
        statusText = "Total appointments: " + totalCount;
    } else {
        statusText = "Showing " + filteredCount + " of " + totalCount + " appointments";
        
        // Add filter info
        if (hasSearch && hasAdvancedFilters) {
            statusText += " (search + filters active)";
        } else if (hasSearch) {
            statusText += " (search active)";
        } else if (hasAdvancedFilters) {
            statusText += " (filters active)";
        }
    }
    
    statusLabel.setText(statusText);
}
```

This provides users with:
- **Current vs total record counts**
- **Active filter indicators**
- **Combined filter state awareness**

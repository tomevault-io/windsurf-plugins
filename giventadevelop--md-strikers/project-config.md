---
trigger: always_on
description: This rule defines the filtering logic, display rules, and "Buy Tickets" button/image display logic for the events listing page (`/events`). It ensures consistent event filtering, recurring event handling, and proper routing for ticket purchases.
---

# Events Page Filtering and Display Rules

## **Overview**
This rule defines the filtering logic, display rules, and "Buy Tickets" button/image display logic for the events listing page (`/events`). It ensures consistent event filtering, recurring event handling, and proper routing for ticket purchases.

## **Problem Solved**
- **Consistent Event Filtering**: Ensures all events displayed meet specific criteria (active, date-based, recurring event handling)
- **Buy Tickets Display Logic**: Determines when and how to show the "Buy Tickets" image/button
- **Payment Flow Routing**: Routes users to the correct checkout page based on event payment configuration
- **Recurring Event Handling**: Properly filters and displays recurring events showing only the next occurrence

---

## **Event Filtering Rules**

### **1. Active Events Only**
- **Rule**: Only events with `isActive === true` are displayed
- **Backend Query**: `isActive.equals=true`
- **Rationale**: Inactive events should not appear in public listings

### **2. Date-Based Filtering**

#### **Future Events (Default View)**
- **Rule**: Show events where `startDate >= today` (including today)
- **Backend Query**: `startDate.greaterThanOrEqual=today` (YYYY-MM-DD format)
- **Sort Order**: `sort=startDate,asc` (earliest first)
- **Display Logic**: Events happening today or in the future

#### **Past Events (Toggle View)**
- **Rule**: Show events where `endDate < today`
- **Backend Query**: `endDate.lessThan=today` (YYYY-MM-DD format)
- **Sort Order**: `sort=startDate,desc` (most recent first)
- **Display Logic**: Events that have already ended

#### **Date Range Search (Overrides Toggle)**
- **Rule**: If user specifies date range, it overrides Future/Past toggle
- **Backend Query**:
  - `startDate.greaterThanOrEqual=searchDateFrom` (if provided)
  - `startDate.lessThanOrEqual=searchDateTo` (if provided)
- **Priority**: Date range search takes precedence over Future/Past toggle

### **3. Title Search Filter**
- **Rule**: Filter events by title containing search term (case-insensitive)
- **Backend Query**: `title.contains=searchTitle` (trimmed, case-insensitive)
- **Combines With**: Date filtering (both filters apply simultaneously)

### **4. Recurring Event Handling**

#### **Recurring Event Detection**
- **Rule**: Event is considered recurring if `isRecurring === true`
- **Series Identification**: Uses `recurrenceSeriesId` or `parentEventId` or `event.id` as series identifier

#### **Next Occurrence Calculation**
- **Rule**: Calculate next occurrence date using `getNextOccurrenceDate(event, todayDate)`
- **Time Window**: Only show next occurrence if it's within 1 year from today
- **Date Update**: Update event's `startDate` to next occurrence date for display purposes

#### **Series Deduplication**
- **Rule**: Only show one event per recurring series (the one with earliest next occurrence)
- **Logic**:
  - First event from series: Add to map
  - Subsequent events from same series: Compare dates, keep earlier occurrence
- **Skip Child Events**: Skip events with `parentEventId` or `recurrenceSeriesId` but `isRecurring === false`

#### **Recurring Event Filtering Flow**
```typescript
// Process events and filter recurring events to show only next occurrence
eventList.forEach((event) => {
  if (isRecurringEvent(event)) {
    const seriesId = event.recurrenceSeriesId || event.parentEventId || event.id;
    const nextOccurrence = getNextOccurrenceDate(event, todayDate);

    if (!nextOccurrence || nextOccurrence > oneYearFromNow) {
      return; // Skip if no next occurrence or beyond 1 year
    }

    // Update event startDate to next occurrence
    const eventWithNextOccurrence = { ...event, startDate: nextOccurrenceStr };

    // Keep only earliest occurrence per series
    const existingSeriesEvent = recurringSeriesMap.get(seriesId);
    if (!existingSeriesEvent || nextOccurrence < new Date(existingSeriesEvent.startDate!)) {
      recurringSeriesMap.set(seriesId, eventWithNextOccurrence);
    }
  } else {
    // Skip child events (have parentEventId/recurrenceSeriesId but not recurring)
    const seriesId = event.recurrenceSeriesId || event.parentEventId;
    if (seriesId) {
      return; // Skip child event
    }
    // Non-recurring event - add directly
    processedEvents.push(event);
  }
});
```

### **5. Pagination**
- **Backend Fetch Size**: `BACKEND_FETCH_SIZE = 50` (fetch more to account for filtering)
- **Display Size**: `EVENTS_PAGE_SIZE = 20` (display 20 events per page after filtering)
- **Rationale**: Fetch more events than displayed because recurring event filtering reduces count

---

## **Buy Tickets Image/Button Display Rules**

### **Display Conditions**

#### **1. Event Must Be Ticketed**
- **Rule**: `event.admissionType?.toUpperCase() === 'TICKETED'`
- **Case Handling**: Case-insensitive check (handles 'TICKETED', 'ticketed', etc.)
- **Rationale**: Only ticketed events should show Buy Tickets option

#### **2. Event Must Be Upcoming**
- **Rule**: Event date must be today or in the future
- **Date Calculation**:
  ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

---
trigger: always_on
description: This rule defines the business logic and UI patterns for the future/past events toggle switch implemented on both admin and public events pages. The toggle allows users to switch between viewing future events (upcoming) and past events (historical), with intelligent auto-switching and informative messaging.
---

# Future / Past Events Toggle Switch Business Rules

## **Overview**
This rule defines the business logic and UI patterns for the future/past events toggle switch implemented on both admin and public events pages. The toggle allows users to switch between viewing future events (upcoming) and past events (historical), with intelligent auto-switching and informative messaging.

## **Problem Solved**
- **User Experience**: Automatically shows past events when no future events exist, preventing empty "No events found" messages
- **Information Clarity**: Provides clear messaging about event availability and how to use the toggle switch
- **Consistent Behavior**: Ensures the same logic works on both admin and public pages (with appropriate customization)
- **Empty State Handling**: Gracefully handles all scenarios (no events, no future events, no past events)

## **Core Business Rules**

### **1. Auto-Switch on Page Load**
- **Rule**: On initial page load, if there are no future events but past events exist, automatically switch to showing past events
- **When**: Only on initial load (page 0, no search filters applied)
- **Purpose**: Prevents showing "No events found" when past events are available
- **Implementation**: Check both future and past event counts on initial load, then auto-switch if needed

### **2. Event Count Tracking**
- **Rule**: Track both future and past event counts separately to determine appropriate messaging
- **State Variables**: 
  - `futureEventCount`: Number of future events (events with `startDate >= today`)
  - `pastEventCount`: Number of past events (events with `endDate < today`)
  - `hasCheckedInitialLoad`: Flag to ensure initial check happens only once
  - `isAutoSwitching`: Flag to prevent double-loading when auto-switching

### **3. Info Box Messages**

#### **No Events at All (Both Future and Past)**
- **Condition**: `futureEventCount === 0 && pastEventCount === 0`
- **Message**: 
  - **Title**: "There are no events listed yet."
  - **Body**: "Please check back again. New events will appear here once they are created. Please use future / past events switch above."
- **Styling**: Blue info box (`bg-blue-50 border border-blue-200`)
- **Icon**: Info icon (blue)

#### **Showing Past Events (No Future Events Exist)**
- **Condition**: `showPastEvents === true && futureEventCount === 0 && pastEventCount > 0`
- **Message**: "Here is the list of recent events. New future events will be added soon. Please use future / past events switch above."
- **Styling**: Amber info box (`bg-amber-50 border border-amber-200`)
- **Icon**: Info icon (amber)
- **Position**: Above the events table

#### **Showing Future Events (No Future Events Exist)**
- **Condition**: `showPastEvents === false && futureEventCount === 0`
- **Message**:
  - **Title**: "No future events created."
  - **Body**: "Please use future / past events switch above."
- **Styling**: Blue info box (`bg-blue-50 border border-blue-200`)
- **Icon**: Info icon (blue)
- **Admin Pages**: Include "Create New Event" button
- **Public Pages**: No action button (public users cannot create events)

### **4. Date Filtering Logic**

#### **Future Events Filter**
- **Query Parameter**: `startDate.greaterThanOrEqual=${today}`
- **Sort**: `startDate,asc` (earliest first)
- **Definition**: Events that start today or in the future (including today)

#### **Past Events Filter**
- **Query Parameter**: `endDate.lessThan=${today}`
- **Sort**: `startDate,desc` (newest first)
- **Definition**: Events that have already ended (ended before today)

### **5. Date Format**
- **Format**: `YYYY-MM-DD` (ISO date format)
- **Calculation**: `new Date().toISOString().split('T')[0]`
- **Usage**: Used for all date comparisons and API queries

## **Implementation Pattern**

### **State Management**
```tsx
// Track event counts for both future and past
const [futureEventCount, setFutureEventCount] = useState<number | null>(null);
const [pastEventCount, setPastEventCount] = useState<number | null>(null);
const [hasCheckedInitialLoad, setHasCheckedInitialLoad] = useState(false);
const [isAutoSwitching, setIsAutoSwitching] = useState(false);
const [showPastEvents, setShowPastEvents] = useState(false);
```

### **Initial Load Check**
```tsx
// On initial load, check both future and past event counts
if (!hasCheckedInitialLoad && page === 0 && !searchTitle && !searchDateFrom && !searchDateTo) {
  // Check future events count
  const futureParams = {
    sort: 'startDate,asc',
    pageNum: 0,
    pageSize: 1, // Just need count, not data
    startDate: today,
  };
  const { totalCount: futureCount } = await fetchEventsFilteredServer(futureParams);
  setFutureEventCount(futureCount);

  // Check past events count
  const pastParams = {
    sort: 'startDate,desc',
    pageNum: 0,
    pageSize: 1, // Just need count, not data
    endDate: today,
  };
  const { totalCount: pastCount } = await fetchEventsFilteredServer(pastParams);
  setPastEventCount(pastCount);

  setHasCheckedInitialLoad(true);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

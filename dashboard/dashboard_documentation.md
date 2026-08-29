# Mobile Dashboard UI/UX Specification Guide

## 1. Overview
This document serves as the architectural blueprint for generating the native mobile application's dashboard. The design follows an **Ultra-Minimalist Flat Design** methodology utilizing the **"Stealth Privacy" (RecordTwin)** theme. It supports a unified Light and Dark mode system and focuses on displaying critical data and actionable shortcuts without clutter.

---

## 2. Layout & Spacing Rules
- **Non-Scrollable Root View**: The main screen itself does not scroll vertically. It acts as a rigid app frame holding distinct components. Only designated interior elements (specifically the Document Management Container) are allowed to scroll.
- **Unified Component Spacing**: Every major container block (Search Bar, Carousel, Quick Actions, Document List) must be separated by exactly **12 units (dp/px) of vertical margin**. This provides a perfectly balanced, flush layout across the entire screen.

---

## 3. Color Scheme (Stealth Privacy Theme)
The application must support dynamic switching between Light Mode and Dark Mode. Implement these exact color mappings:

| Element Layer | Light Mode | Dark Mode |
| :--- | :--- | :--- |
| **Global Background** | `#E6E8EA` (Pale Slate) | `#0A0B0C` (Deep Black) |
| **Main Surfaces (Cards/Containers)** | `#FFFFFF` (Pure White) | `#242629` (Slate Grey) |
| **Nested Surfaces (Inputs/Search)** | `#F3F4F6` (Soft Grey) | `#1A1D21` (Darker Grey) |
| **Primary Accent** | `#4A2884` (Deep Violet) | `#A78BFA` (Electric Purple) |
| **Primary Text** | `#111827` (Near Black) | `#F9FAFB` (Off White) |
| **Secondary Text** | `#6B7280` (Medium Grey) | `#9CA3AF` (Light Grey) |

---

## 4. Components & Functional Logic

### Header
- **Sync Pill (Left)**: Acts as both a passive status indicator and an active button. If a sync is required, it dynamically changes into a clickable action button (e.g., "Sync It"). When tapped, the sync process initiates in the background, and the Analytics Carousel automatically swipes/opens the "Sync Status" slide.
- **Profile Avatar (Right)**: An interactive profile icon utilizing the main surface color scheme. Tapping this will navigate to the User Profile screen (where the Theme Toggle and other settings will live).

### Search Bar
- A nested surface input field allowing the user to search across documents, records, and plots. 

### Analytics Carousel Container
A swipeable, horizontal card block designed to hold multiple "docks" or slides. Users can swipe left or right to move between docks one by one.
- **Smart Launch Behavior**: 
  - If a **Critical Notification** exists upon app launch, the carousel automatically snaps to the Notification dock.
  - If a **Sync is active** upon launch, the carousel snaps to the Sync Status dock.
  - *Priority Override*: If both events occur simultaneously, the Critical Notification dock is strictly prioritized and shown first.
- **Graph Dock Buttons (Wheel Picker)**: On the data analytics slide, the left column utilizes a 3-slot vertical "Wheel Picker". As the user swipes vertically on the picker, the active metric (e.g., Files, Folders, Tags) cycles into the center slot and highlights in the Primary Accent color. Selecting a metric dynamically drives the Data Graph displayed on the right side of the dock.

### Quick Actions Card
A horizontally scrolling ribbon for immediate task execution (e.g., Add File, Scan, etc.).
- **Expand Button (4-Square Grid Icon)**: Located in the container header. Tapping this expands a hidden drawer showing *all* available action buttons. From within this expanded state, users can edit their quick actions, allowing them to add or remove specific shortcut buttons from their minimized dashboard view.

### Document Management Container
The primary vertical scrolling list containing the user's files.
- **Scroll Behavior**: Because the parent screen is locked, this container handles all vertical file scrolling natively. 
- **Filtering**: It employs a Segmented Tab Control (Recent, Pinned, Drafts) at the top to instantly filter the visible files.
- **Bottom Clearance**: The container must calculate an exact `12 unit (dp/px)` clearance gap at its bottom edge to ensure the last item in the list is perfectly visible without colliding with the floating bottom navigation bar.

### Floating Bottom Navigation
A neumorphic, curved navigation bar fixed to the bottom edge of the screen.
- **Scroll Behavior**: To maximize screen real estate for the Document Container, the navigation bar automatically disappears (slides down off-screen) when the user is actively scrolling down the list.
- **Reappearance**: The navigation bar automatically slides back up and reappears after 2 seconds of scroll inactivity or when the user begins scrolling upward.

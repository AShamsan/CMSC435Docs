# UI Choices for Map & Routing — Developer Documentation

Location: `app/routes/routing/route.tsx`, `app/components/OpenLayers/routing/components/`
Route/API: UI Components
Access: Routing Interface

## 1. Purpose
The UI design is focused on maximum map real estate while providing powerful tools for researchers. This document outlines the choice of components, themes, and layouts.

## 2. Functional Overview
- **Maximizing Map Space**: The map is full-screen (`position: absolute`), with UI panels (Sidebar/TopBar) overlayed using glassmorphism and dark themes to minimize visual distraction.
- **Responsive Management**: The "Routing Manager" is a collapsible panel that hides complex list data until the user needs it.
- **Visual Feedback**: Waypoint colors are generated dynamically (`generateWaypointColor`) to distinguish between steps in a route.

## 3. File Structure and Responsibilities
- `ROUTING_TOP_BAR_THEME`: A centralized constant in `route.tsx` that defines the dark-mode aesthetic (e.g., `#1e1e1e` background, `#16a34a` green accents).
- `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`: Uses a "Glass" card style for a premium, modern look.
- `app/components/OpenLayers/routing/ReorderItem.tsx`: Implements drag-and-drop UI for reordering waypoints.

## 4. Workflow (Recent Changes & Why)
- **Top Bar Refactoring**: We moved the routing mode and experiment selectors to a slim top bar. **Reason**: Users felt the sidebar was too cluttered on mobile.
- **Floating Action Buttons (FABs)**: Added FABs for map layer toggles. **Reason**: Quick access without opening a full menu.
- **Tutorial Integration**: Added a "Tutorial" button in the top bar. **Reason**: To help first-time users understand the complex "Loop" vs "Point-to-Point" logic.

## 5. Deployment / Configuration Notes
- **MUI Integration**: The project uses **Material UI (MUI)**. Any new UI elements should follow the existing `SX` patterns to maintain theme consistency.

## 6. Maintenance Guide
- **Updating Themes**: Change the `ROUTING_TOP_BAR_THEME` object in `app/routes/routing/route.tsx`.
- **Modifying Reorder Logic**: Edit `app/components/OpenLayers/routing/ReorderItem.tsx`.

## 7. Known Dependencies
- `@mui/material`: Core UI components.
- `@fortawesome/react-fontawesome`: Used for iconography.
- `framer-motion`: Handles the sidebar reordering animations.

## 8. Troubleshooting
- **Z-Index Issues**: If UI elements appear behind the map, ensure they have a `zIndex` higher than the OpenLayers container (usually > 1).

## 9. References
- `app/routes/routing/route.tsx:L123` (Theme Constants)
- `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`

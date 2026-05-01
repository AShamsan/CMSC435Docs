# UI Choices for Map and Routing - Developer Documentation

Location: `app/routes/routing/route.tsx`, `app/components/OpenLayers/routing/components/`  
Route/API: UI components  
Access: Routing Interface

---

## 1. Purpose

This document explains the main UI design choices for the Routing page and map interface.

The goal of the UI is to give users enough control to create, edit, preview, and start routes while keeping the map easy to use.

Since routing is map-heavy, the design gives the map as much space as possible. Controls are placed as overlays, collapsible panels, or top-level controls so the user can still see the route and campus map clearly.

The UI must work for different types of users:

- Regular users who want to create and start a wellness route.
- Researchers who want to test experiments and environmental layers.
- Admins or developers who need to validate routing behavior.

---

## 2. Functional Overview

The map and routing UI supports:

- Full-screen map interaction.
- A routing sidebar for managing waypoints.
- A top bar for important route controls.
- Route mode selection.
- Experiment selection.
- Environmental layer toggles.
- Route preview.
- Start route workflow.
- Tutorial access.
- Responsive layout for smaller screens.
- Clear visual feedback for waypoints and route state.

The UI is designed to avoid putting every control in one place.

The top bar handles high-level controls.

The sidebar handles route details.

The map remains the main focus.

---

## 3. Main Design Goals

### Keep the map visible

The map is the most important part of the routing interface.

Users need to click locations, inspect route paths, view waypoints, and compare environmental layers.

Because of this, the map uses a full-page layout and UI controls appear on top of it.

---

### Keep controls organized

Routing has several controls, so the UI separates them by purpose.

High-level controls belong in the top bar.

Detailed route management belongs in the sidebar.

Map-specific quick actions can use floating buttons or compact panels.

This prevents the interface from feeling too crowded.

---

### Support mobile and smaller screens

Users may use the app while walking outside.

Because of this, the UI should not depend only on a large desktop layout.

Controls should be collapsible, readable, and easy to tap.

The map should still remain usable on smaller screens.

---

### Give clear visual feedback

The UI should make it obvious what the user has selected.

Users should be able to understand:

- Which waypoints are part of the route.
- What order the waypoints are in.
- Which route mode is active.
- Which experiment is selected.
- Which environmental layers are visible.
- Whether the route is ready to start.

---

## 4. File Structure and Responsibilities

### `app/routes/routing/route.tsx`

This file manages the main page layout and UI state.

Main responsibilities:

- Define the overall Routing page layout.
- Manage route mode state.
- Manage selected experiment state.
- Manage selected visualized layers.
- Manage top bar state.
- Define theme constants.
- Pass state into map and sidebar components.
- Coordinate route start behavior.

Important constant:

`ROUTING_TOP_BAR_THEME`

This controls the main styling for the routing top bar.

---

### `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`

This component displays the route manager.

Main responsibilities:

- Show selected waypoints.
- Allow waypoint removal.
- Allow waypoint reordering.
- Display route-related controls.
- Show route information.
- Keep detailed route editing separate from the top bar.

The sidebar should focus on managing the selected route.

---

### `app/components/OpenLayers/routing/ReorderItem.tsx`

This component supports drag-and-drop waypoint ordering.

Main responsibilities:

- Render a route item inside the reorderable list.
- Allow users to change waypoint order.
- Keep reorder logic separated from the main sidebar.

---

### Routing top bar

The top bar contains important high-level controls.

Examples:

- Route mode selector
- Experiment selector
- Tutorial button
- Layer control entry point
- Start route button if applicable

The top bar should stay compact so it does not cover too much of the map.

---

## 5. UI Layout Choices

### Full-screen map

The map is treated as the main workspace.

It should take up most of the page.

This makes route editing feel visual and interactive.

---

### Overlay controls

Controls are displayed on top of the map instead of taking the map out of view.

This helps preserve map space.

It also makes the interface feel closer to modern map applications.

---

### Dark routing controls

The routing controls use a dark theme with green accents.

Example values:

- Background: `#1e1e1e`
- Accent: `#16a34a`

The dark styling helps controls stand out against the map while matching the EcoWell style.

---

### Collapsible routing manager

The routing manager can be collapsed or hidden.

This is useful when users want to focus on the map after setting up the route.

It is also useful on smaller screens where sidebar space is limited.

---

### Floating action buttons

Floating action buttons can be used for quick map actions.

They are useful for actions that should be available without opening a full panel.

Examples:

- Toggle layers
- Reopen tutorial
- Return to route controls
- Clear route if needed

---

### Tutorial access

The routing workflow can be complex, especially for first-time users.

The UI includes tutorial access to explain:

- Point-to-point mode
- Loop mode
- Waypoint selection
- Experiment selection
- Environmental layer visualization
- Starting a route

This reduces confusion and helps users understand the routing workflow.

---

## 6. Recent Changes and Why

### Top bar refactor

Route mode and experiment selectors were moved to a slim top bar.

Reason:

The sidebar was becoming too crowded, especially on smaller screens.

Moving high-level controls to the top bar keeps the sidebar focused on route details.

---

### Layer controls made easier to access

Layer controls were placed closer to the map experience.

Reason:

Users need to turn environmental data layers on and off while looking at the map.

This helps users compare route behavior with the visible data layers.

---

### Tutorial button added

A tutorial button was added to the top-level interface.

Reason:

The routing workflow has multiple concepts that may not be obvious to new users.

The tutorial helps explain route creation without requiring a developer or researcher to explain it manually.

---

### Better mobile layout

The UI was adjusted to reduce clutter on smaller screens.

Reason:

Users may use the app while walking, so the route and map controls need to stay readable and usable on mobile.

---

## 7. Route UI Responsibilities

### Waypoint display

The UI should clearly show selected waypoints.

Each waypoint should show enough information for the user to understand what it represents.

Possible labels:

- Plant name
- Marker name
- Building name
- Custom waypoint label

---

### Waypoint order

Waypoint order matters because it affects route generation.

The UI should make the order clear and editable.

Drag-and-drop is used to make reordering easier.

---

### Route mode display

The selected route mode should be visible.

The user should know whether they are creating:

- A point-to-point route
- A loop route

This is important because the required inputs may change depending on the mode.

---

### Experiment display

If an experiment is selected, the UI should show it clearly.

Users should know which experiment is influencing route generation.

This is important for researchers testing different environmental priorities.

---

### Layer display

If environmental layers are enabled, the UI should show which layers are active.

This prevents confusion when the map is displaying colored grid data.

---

## 8. Relationship to Routing Architecture

The UI is only one part of the routing system.

The UI should not directly own all routing logic.

Instead:

- UI components display controls.
- Page state tracks selections.
- Map components handle OpenLayers rendering.
- API routes handle route generation and scoring.
- Shared utilities handle route transfer.

This separation makes the system easier to maintain.

---

## 9. Relationship to Explore Map

The Routing page is for planning.

The Explore Map is for following the route.

When the user clicks start, the active route should appear in the Explore Map.

The UI should make that transition feel natural.

Users should also have a way to return to routing if they want to edit the route.

This supports the workflow:

`Create route -> Start route -> Follow route -> Return and edit if needed`

---

## 10. Deployment and Configuration Notes

### MUI usage

The project uses Material UI.

New UI elements should follow the existing MUI `sx` styling patterns when possible.

This keeps the interface visually consistent.

---

### Z-index ordering

UI overlays must have a higher z-index than the OpenLayers map.

If buttons, sidebars, or panels appear behind the map, check their z-index values.

---

### Client-only map behavior

Map UI that depends on OpenLayers should only run on the client.

OpenLayers should not be used directly during server-side rendering.

---

### Mobile layout

Any new UI control should be tested on smaller screens.

A control that works on desktop may cover too much of the map on mobile.

---

## 11. Maintenance Guide

### Updating the top bar theme

Update:

`ROUTING_TOP_BAR_THEME`

Located in:

`app/routes/routing/route.tsx`

Use this when changing:

- Background color
- Accent color
- Border style
- Spacing
- Shadow
- Text color

---

### Updating the sidebar

Edit:

`app/components/OpenLayers/routing/components/RoutingSidebar.tsx`

Use this when changing:

- Waypoint display
- Route list layout
- Remove buttons
- Route controls
- Sidebar spacing
- Collapse behavior

---

### Updating waypoint reorder behavior

Edit:

`app/components/OpenLayers/routing/ReorderItem.tsx`

Use this when changing:

- Drag-and-drop behavior
- Reorder item layout
- Waypoint item animation
- Reorder interaction style

---

### Adding a new UI control

When adding a new control:

1. Decide if it belongs in the top bar, sidebar, or floating map controls.
2. Keep high-level controls in the top bar.
3. Keep route detail controls in the sidebar.
4. Keep quick map actions near the map.
5. Make sure it works on smaller screens.
6. Check that it does not block important map areas.

---

### Updating mobile behavior

When changing mobile layout, test:

- Top bar wrapping
- Sidebar width
- Button size
- Touch target spacing
- Map visibility
- Layer controls
- Route start workflow

---

## 12. Testing Checklist

### Layout

- [ ] Map fills the available page space.
- [ ] Top bar does not cover too much of the map.
- [ ] Sidebar opens and closes correctly.
- [ ] Floating buttons remain clickable.
- [ ] UI does not appear behind the map.

---

### Route controls

- [ ] User can add waypoints.
- [ ] User can remove waypoints.
- [ ] User can reorder waypoints.
- [ ] Route mode selector works.
- [ ] Experiment selector works.
- [ ] Start route flow works.

---

### Layer controls

- [ ] User can enable environmental layers.
- [ ] User can disable environmental layers.
- [ ] Active layers are visually clear.
- [ ] Layer controls do not block important route information.

---

### Mobile behavior

- [ ] Buttons are easy to tap.
- [ ] Text is readable.
- [ ] Sidebar does not cover the whole workflow unnecessarily.
- [ ] Map remains usable.
- [ ] Start route action is easy to find.

---

## 13. Troubleshooting

### UI appears behind the map

Check z-index values.

The OpenLayers map can cover controls if the controls do not have a high enough z-index.

---

### Sidebar feels too crowded

Move high-level controls to the top bar.

Keep the sidebar focused on route management and waypoint details.

---

### Buttons are hard to use on mobile

Check:

- Button size
- Spacing
- Panel width
- Touch target size
- Whether the control should be collapsible

---

### Top bar covers too much of the map

Reduce top bar content.

Move detailed controls into the sidebar or a collapsible menu.

---

### Waypoint colors are confusing

Check waypoint color generation logic.

Waypoint colors should help users distinguish waypoint order.

If colors are too similar, adjust the color generation strategy.

---

### Tutorial is hard to find

Place tutorial access in a visible location such as the top bar or an info icon.

Users should be able to reopen the tutorial after closing it.

---

## 14. Known Dependencies

| Dependency | Usage |
|---|---|
| `@mui/material` | Main UI components and styling |
| `@fortawesome/react-fontawesome` | Icons |
| `framer-motion` | Drag-and-drop and animations |
| OpenLayers | Map rendering |
| React | Component state and layout |

---

## 15. Developer Notes

- Keep the map as the main focus.
- Do not overload the sidebar with every control.
- Use the top bar for high-level choices.
- Use the sidebar for route details.
- Make tutorial access easy to find.
- Test every major UI change on smaller screens.
- Make sure UI overlays do not block route lines or waypoint markers.
- Keep visual design consistent with EcoWell styling.

---

## 16. References

- `app/routes/routing/route.tsx`
- `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`
- `app/components/OpenLayers/routing/ReorderItem.tsx`

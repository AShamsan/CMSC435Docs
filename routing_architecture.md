# Routing Code Architecture - Developer Documentation

Location: `app/routes/routing/`, `app/components/OpenLayers/routing/`  
Route/API: `/routing`  
Access: Main Navbar -> Routes

---

## 1. Purpose

The Routing page is the main interface for creating, editing, previewing, and starting routes in EcoWell.

This page connects the React UI, the OpenLayers map, and the backend routing APIs. Its main purpose is to let users create routes in a clear and interactive way while keeping the routing logic separated from the map rendering and UI components.

The Routing page is used for planning and editing routes. The Explore Map is used for following or walking the route after the user starts it.

The intended workflow is:

`Routing page -> create or edit route -> Explore Map -> start and follow route -> Routing page -> return and edit if needed`

This separation keeps the system easier to understand and maintain. It also prevents the Explore Map from becoming overloaded with route editing tools.

---

## 2. Functional Overview

The Routing page supports several routing features:

- Add waypoints by clicking on the map.
- Detect whether a clicked point is a plant, marker, building, or custom waypoint.
- Display selected waypoints in a routing sidebar.
- Reorder waypoints using drag-and-drop.
- Remove waypoints from the route.
- Select route mode, such as point-to-point or loop.
- Generate routes automatically when route inputs change.
- Preview the generated route on the map.
- Save temporary route state while moving between pages.
- Send an active route to the Explore Map.
- Return from the Explore Map back to the Routing page for editing.
- Show environmental data layers on the map.

The architecture is designed so the main route file controls page state, the map file controls OpenLayers behavior, the sidebar controls route management UI, and backend APIs handle route generation and waypoint resolution.

---

## 3. Main Design Idea

The Routing page should behave like a route planning workspace.

The user should be able to:

1. Choose where the route starts.
2. Add one or more destinations or waypoints.
3. Select route mode.
4. Select an approved experiment if needed.
5. View environmental layers.
6. Preview the generated route.
7. Start the route on the Explore Map.

The route should not feel disconnected when the user switches pages. Because of that, the architecture uses temporary route transfer through `sessionStorage`.

This allows the route to move from the Routing page to the Explore Map without immediately needing to save it permanently in the database.

---

## 4. Page-Level Responsibilities

### Route creation

The page lets users create a route by selecting locations on the map.

Each selected location becomes a route item.

A route item can represent:

- Plant
- Marker
- Building
- Custom waypoint

---

### Route editing

The page lets users update the route after it has been created.

Users can:

- Reorder waypoints.
- Remove waypoints.
- Change routing mode.
- Change selected experiment.
- Turn data layers on or off.
- Regenerate the route.

---

### Route preview

The generated route is drawn on the map so users can inspect it before starting.

This helps users confirm that the route makes sense before sending it to the Explore Map.

---

### Route transfer

When the user starts a route, the active route is transferred to the Explore Map.

This is handled using shared route transfer utilities.

---

### Route recovery

If the user returns from the Explore Map, the Routing page should be able to reload the route for editing.

This makes the workflow feel continuous instead of forcing the user to start over.

---

## 5. File Structure and Responsibilities

### `app/routes/routing/route.tsx`

This is the main Remix route file for the Routing page.

Main responsibilities:

- Render the Routing page layout.
- Manage selected route state.
- Manage selected route mode.
- Manage selected experiment.
- Manage selected visualized layers.
- Track route generation status.
- Trigger route generation when route inputs change.
- Pass route data into map and sidebar components.
- Handle start route behavior.
- Coordinate route transfer to the Explore Map.

This file acts as the page controller.

It should not contain every low-level map or algorithm detail. Instead, it should connect the UI, map, and backend APIs together.

---

### `app/components/OpenLayers/routing/map.client.tsx`

This file contains the client-only OpenLayers map component for the Routing page.

Main responsibilities:

- Initialize the OpenLayers map.
- Handle map click events.
- Display route waypoints.
- Display route lines.
- Display environmental data layers.
- Manage OpenLayers sources and layers.
- Update map rendering when route data changes.
- Resize the map when UI panels open or close.

This file must run only on the client because OpenLayers depends on browser APIs such as `window`.

Any component that uses this map should be wrapped in a client-only setup.

---

### `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`

This component displays the route management sidebar.

Main responsibilities:

- Show selected route waypoints.
- Allow users to remove waypoints.
- Allow users to reorder waypoints.
- Show route controls.
- Display route information.
- Help users understand the current route structure.

The sidebar should focus on detailed route editing, not every global page setting.

---

### `app/components/OpenLayers/routing/ReorderItem.tsx`

This component supports drag-and-drop behavior for waypoints.

Main responsibilities:

- Render a single reorderable waypoint item.
- Support waypoint ordering changes.
- Work with the sidebar route list.

This keeps reorder logic separated from the larger sidebar component.

---

### `app/utils/activeRouteTransfer.ts`

This utility handles temporary route movement between pages.

Main responsibilities:

- Save an active route before navigating to the Explore Map.
- Load a route when returning to the Routing page.
- Clear temporary route data when it is no longer needed.

Important functions:

- `saveActiveRoute`
- `loadEditingRoute`
- `clearEditingRoute`

This file is important because it keeps route transfer logic reusable and avoids duplicating session storage behavior in multiple components.

---

### `app/routes/api.routing.resolveWaypoints.tsx`

This backend API helps convert map clicks into structured route items.

Main responsibilities:

- Receive clicked coordinates.
- Check whether the point is near a known plant.
- Check whether the point is near a known marker.
- Support building or custom waypoint resolution if implemented.
- Return a structured route item to the frontend.

This API helps keep map clicks meaningful. Instead of storing only raw coordinates, the system can connect a waypoint to a real database-backed entity when possible.

---

### `app/routes/api.routing._shared.types.ts`

This file stores shared routing types.

Main responsibilities:

- Define route item types.
- Define shared API payload types.
- Define shared constants.
- Keep frontend and backend routing structures consistent.

This file should be updated whenever the route item structure changes.

---

## 6. Route Item Types

A route item represents one point in the route.

Possible route item types include:

- `plant`
- `marker`
- `building`
- `waypoint`

Each route item should include enough information to display it in the UI and reconstruct it later.

Useful route item fields include:

- `type`
- `id`
- `lat`
- `lng`
- `displayName`
- `plantId`
- `markerId`
- `buildingId`
- `buildingName`

The exact fields depend on the current implementation.

The important idea is that route items should not only be raw coordinates when the system can identify a real object.

---

## 7. Routing Page Workflow

### Step 1: User opens the Routing page

The user navigates to:

`/routing`

The page loads the map, sidebar, route controls, and experiment controls.

If there is an editing route stored in `sessionStorage`, the page may reload it.

---

### Step 2: User clicks on the map

The user clicks a point on the map.

The map component receives the clicked coordinate.

---

### Step 3: Clicked point is resolved

The system checks whether the clicked coordinate is close to a known campus object.

Possible results:

- Plant
- Marker
- Building
- Custom waypoint

If a known object is found, the waypoint uses that object information.

If nothing is found nearby, the point becomes a custom waypoint.

---

### Step 4: Route state updates

The new route item is added to the route state.

The sidebar updates to show the new waypoint.

The map updates to show the waypoint marker.

---

### Step 5: Route generation is triggered

A `useEffect` watches route-related state.

When enough valid route inputs exist, the route generation process runs after a short debounce.

Current debounce:

`400ms`

This prevents unnecessary API calls while the user is still editing.

---

### Step 6: Backend returns route geometry

The backend returns route coordinates.

The map uses those coordinates to draw the route line.

---

### Step 7: User reviews route

The user can inspect the generated route and make changes if needed.

Possible changes:

- Add another waypoint.
- Remove a waypoint.
- Reorder waypoints.
- Change route mode.
- Change experiment selection.
- Toggle environmental layers.

---

### Step 8: User starts route

When the user starts the route, the active route is stored temporarily.

The user is sent to the Explore Map.

The Explore Map loads and displays the active route so the user can follow it.

---

### Step 9: User can return to edit

If the user ends the route or wants to change it, the workflow can return them to the Routing page.

The previous route can be loaded for editing.

---

## 8. Routing Modes

The Routing page supports multiple route modes.

### Point-to-point mode

Point-to-point mode generates a route from a start point to an end point.

This mode is useful when the user already knows where they want to go.

Example:

`Start: McKeldin Library`

`End: Stamp Student Union`

The route engine may still optimize based on selected experiment layers, but the route has a clear destination.

---

### Loop mode

Loop mode generates a route that starts and ends near the same location.

This mode is useful when the user wants to walk around campus for wellness, nature exposure, or exploration without needing a specific destination.

The user may optionally provide a target loop distance.

Example:

`Start: McKeldin Mall`

`Target distance: 1 mile`

The routing engine tries to generate a route that returns near the start point while balancing environmental score and route quality.

---

## 9. Relationship to Experiments

The Routing page can use approved experiments to influence route generation.

An experiment may define:

- Dataset name
- Environmental layers
- Layer weights
- Optimization goal

When an experiment is selected, the Routing page sends its layers and weights to routing APIs.

This allows researchers to test how different environmental priorities affect generated routes.

Example:

`Experiment A: shade weight = 8, biodiversity weight = 5, noise weight = 2`

The routing engine can use this information to prefer route candidates with better combined environmental scores.

---

## 10. Relationship to Map Layer Visualization

The Routing page can also display selected environmental data layers on the map.

This helps users visually understand the data that may influence the route.

For example, if a shade layer is enabled, users can see where high-shade areas exist on campus.

This makes the route generation process more explainable.

The route engine uses data for scoring.

The map layer visualization uses data for display.

They are connected, but they are not the same thing.

---

## 11. State Persistence

The Routing page uses `sessionStorage` for temporary route transfer.

This supports workflows like:

`/routing -> /map -> /routing`

The route can be preserved while the user moves between planning and walking views.

### Important note

`sessionStorage` is temporary.

It should not replace permanent saved routes.

Permanent routes should be stored in the database.

---

## 12. Client-Side Requirements

OpenLayers requires browser APIs.

Because of this, map components should only be rendered on the client.

If map code runs during server-side rendering, it can cause errors because objects like `window` and `document` are not available.

Use a client-only wrapper for map components.

---

## 13. API Separation

The routing architecture uses separate API routes instead of putting all behavior in one large file.

This keeps logic easier to debug and maintain.

Examples of separated responsibilities:

- Waypoint resolution
- Candidate generation
- Candidate scoring
- Dataset loading
- Route transfer types

This makes the system easier to test because each API has a focused purpose.

---

## 14. Deployment and Configuration Notes

### Browser storage

Route transfer depends on `sessionStorage`.

If the browser blocks storage, active route transfer may fail.

---

### OSRM availability

Route generation depends on the routing backend.

If OSRM is down or unreachable, the map may still load, but route generation will fail.

---

### Dataset availability

If the selected experiment uses a dataset, that dataset must be available to the backend.

If the dataset is missing, route scoring may fail or produce weak results.

---

### Client-only map loading

The map should not be rendered directly during server-side rendering.

Make sure map components are loaded only on the client.

---

## 15. Maintenance Guide

### Adding a new waypoint type

To add a new waypoint type:

1. Update the `RouteItem` type.
2. Update the map click handler.
3. Update waypoint resolution logic.
4. Update route reconstruction logic.
5. Update sidebar display labels.
6. Update saved route or transfer behavior if needed.
7. Test creating, starting, returning, and editing the route.

---

### Changing route transfer behavior

If the route transfer format changes:

1. Update `app/utils/activeRouteTransfer.ts`.
2. Update the Routing page where the active route is saved.
3. Update the Explore Map where the active route is loaded.
4. Update the return-to-edit workflow.
5. Test the full page flow.

---

### Updating route generation triggers

If route generation is too slow or too frequent, review the debounce behavior.

Current debounce:

`400ms`

A shorter debounce makes the route update faster but may send too many API requests.

A longer debounce reduces requests but may make the UI feel slower.

---

### Updating map behavior

Map-specific behavior should mainly be updated in:

`app/components/OpenLayers/routing/map.client.tsx`

Avoid putting detailed OpenLayers logic directly inside the main route file.

---

### Updating shared API types

If the shape of route items or API payloads changes, update:

`app/routes/api.routing._shared.types.ts`

This helps prevent frontend and backend mismatches.

---

## 16. Troubleshooting

### Map does not resize correctly

This usually happens when a sidebar, drawer, or overlay changes the available map space.

Call:

`olRefs.current.map.updateSize()`

after the UI layout changes.

---

### Waypoints disappear

Check:

1. Whether `clearEditingRoute()` is being called too early.
2. Whether `sessionStorage` is being overwritten.
3. Whether the route item type is missing required fields.
4. Whether route reconstruction logic handles the item type.

---

### Route does not generate

Check:

1. The route has enough waypoints.
2. Coordinates are valid.
3. OSRM is reachable.
4. The selected points are close to walkable paths.
5. The frontend is sending the expected API payload.
6. The backend is returning route geometry.

---

### Active route does not appear on Explore Map

Check:

1. `saveActiveRoute` is called before navigation.
2. The Explore Map reads the same storage key.
3. The stored route has valid geometry.
4. `clearEditingRoute` is not called before the Explore Map loads.
5. The Explore Map knows how to render the active route.

---

### Returning to edit does not work

Check:

1. The active route is saved before leaving the Explore Map.
2. The Routing page calls the correct loading function.
3. The route item structure is still valid.
4. The route state is being initialized from storage.

---

## 17. Known Dependencies

| Dependency | Usage |
|---|---|
| `ol` | OpenLayers map rendering |
| `framer-motion` | Waypoint reordering animations |
| `sessionStorage` | Temporary route transfer between pages |
| Remix | Route files, API routes, loaders, and actions |
| React | State management and UI rendering |

---

## 18. Developer Notes

- Keep the Routing page focused on planning and editing.
- Keep the Explore Map focused on following the active route.
- Keep route transfer logic reusable.
- Keep OpenLayers logic inside map-specific files.
- Keep shared route types centralized.
- Avoid mixing algorithm scoring logic directly into UI components.

---

## 19. References

- `app/routes/routing/route.tsx`
- `app/components/OpenLayers/routing/map.client.tsx`
- `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`
- `app/components/OpenLayers/routing/ReorderItem.tsx`
- `app/utils/activeRouteTransfer.ts`
- `app/routes/api.routing.resolveWaypoints.tsx`
- `app/routes/api.routing._shared.types.ts`

# Routing Code Architecture — Developer Documentation

Location: `app/routes/routing/`, `app/components/OpenLayers/routing/`
Route/API: `/routing`
Access: Main Navbar -> Routes

## 1. Purpose
The Routing Architecture provides the infrastructure for interactive route building, state persistence between pages, and integration with the mapping engine. It decouples the UI (React), the map state (OpenLayers), and the pathfinding logic (Remix APIs).

## 2. Functional Overview
- **Interactivity**: Users can click the map to add waypoints, which are automatically identified as Plants, Markers, or Buildings.
- **State Persistence**: Routes can be "saved" to `sessionStorage` to allow a user to move to the Explore map and back without losing their unsaved progress.
- **Modular APIs**: The system uses separate API routes for waypoint resolution, candidate generation, and scoring to keep request payloads small and logic focused.

## 3. File Structure and Responsibilities
- `app/routes/routing/route.tsx`: The main route component. Handles layout, user input state, and orchestrates API calls.
- `app/components/OpenLayers/routing/map.client.tsx`: The map interaction layer. Manages OpenLayers sources for markers, routes, and heatmaps.
- `app/components/OpenLayers/routing/components/RoutingSidebar.tsx`: UI for waypoint management, mode selection, and route generation controls.
- `app/utils/activeRouteTransfer.ts`: Logic for `saveActiveRoute` and `loadEditingRoute` using `sessionStorage`.
- `app/routes/api.routing.resolveWaypoints.tsx`: Backend logic to convert coordinate clicks into database-backed entities (PlantInstance, Marker).

## 4. Workflow
1.  **User Interaction**: User clicks the map in `RoutingMapInner`.
2.  **Resolution**: Map emits a click event; code searches `plantsForMapInteractionRef` to find the nearest plant.
3.  **State Update**: `setRoute` adds the new `RouteItem` to the local state.
4.  **Auto-Generation**: A `useEffect` with a 400ms debounce triggers `generateRoute()`.
5.  **Persistence**: If the user navigates away, `saveEditingRoute` is called to stash the current waypoints.

## 5. Deployment / Configuration Notes
- **Client-Side Only**: Map components must be wrapped in `<ClientOnly>` as OpenLayers requires the `window` object.
- **Session Storage**: State transfer requires a browser environment; it will not work for users with disabled local storage.

## 6. Maintenance Guide
- **Adding New Waypoint Types**: Update the `RouteItem` interface and the click handler in `app/routes/routing/route.tsx` to handle new entity types.
- **API Refactoring**: Shared types are located in `app/routes/api.routing._shared.types.ts`.

## 7. Known Dependencies
- `ol` (OpenLayers): Primary map rendering engine.
- `framer-motion`: Used for smooth reordering of waypoints in the sidebar.
- `sessionStorage`: Used for `activeRouteTransfer`.

## 8. Troubleshooting
- **Map Not Resizing**: Ensure `olRefs.current.map.updateSize()` is called after UI panels (like drawers) toggle.
- **Waypoints Disappearing**: Check if `clearEditingRoute()` is being called prematurely in `route.tsx`.

## 9. References
- `app/routes/routing/route.tsx`
- `app/utils/activeRouteTransfer.ts`
- `app/components/OpenLayers/routing/map.client.tsx`
- `app/routes/api.routing._shared.types.ts`

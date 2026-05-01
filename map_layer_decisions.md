# Map Layer Decisions & Grid Logic — Developer Documentation

Location: `app/routes/routing/route.tsx`, `app/components/OpenLayers/helpers/heatmapRenderer.ts`
Route/API: UI State Toggle
Access: Routing Sidebar -> Map Layers

## 1. Purpose
This document explains the design choices regarding the map's visual layers, specifically why we use a grid-based approach for heatmaps and how the user controls these visualizations.

## 2. Functional Overview
- **Grid-Based vs. Heatmap Layer**: We chose discrete 10m polygons over a standard `HeatmapLayer` to provide precise, non-interpolated data views. This ensures researchers see exactly where the data is located.
- **Layer Toggling**: Users can toggle individual experimental features (e.g., "Shade," "Slope," "Flora") on or off to combine different data views.

## 3. File Structure and Responsibilities
- `app/routes/routing/route.tsx`: Manages the `visualizedLayers` state array.
- `app/components/OpenLayers/helpers/heatmapRenderer.ts`: Defines the **10x10 meter** grid size logic.
- `app/components/OpenLayers/routing/components/RoutingFilterPanel.client.tsx`: UI for toggling plant/marker filters which affect the map's interaction layer.

## 4. Workflow
- **Grid Sizing**: The grid size is hardcoded in `renderHeatmapPoints` using:
  ```javascript
  const dx2 = (10 * metersToLon) / 2;
  const dy2 = (10 * metersToLat) / 2;
  ```
  This creates a center-aligned 10m square for every data point.
- **Layer Update**: When a user toggles a layer in the sidebar, `setVisualizedLayers` updates, triggering a re-fetch of `/api/routing/datasetPoints`.

## 5. Deployment / Configuration Notes
- **Grid Size Change**: To change the grid size globally, you must update both the `renderHeatmapPoints` function (Frontend) and ensure the data resolution in your Parquet files supports the new size.

## 6. Maintenance Guide
- **Adding a Grid Toggle**: To implement a user-selectable grid size, add a `gridSize` state to `route.tsx` and pass it as a parameter to the `datasetPoints` API and the `renderHeatmapPoints` function.

## 7. Known Dependencies
- **OpenLayers Projections**: Used to convert meters to map coordinates correctly.

## 8. Troubleshooting
- **Polygons overlapping**: Ensure the `dx2`/`dy2` calculation correctly accounts for the longitude/latitude aspect ratio at your specific latitude.

## 9. References
- `app/components/OpenLayers/helpers/heatmapRenderer.ts:L44-45` (Grid Size Calculation)
- `app/routes/routing/route.tsx:L356` (Visualized Layers State)

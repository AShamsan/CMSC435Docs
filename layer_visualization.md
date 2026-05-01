# Layer Visualization — Developer Documentation

Location: `app/components/OpenLayers/helpers/heatmapRenderer.ts`, `app/routes/api.routing.datasetPoints.tsx`
Route/API: `/api/routing/datasetPoints`
Access: Internal fetch from `RoutingMapInner` and `MapUI`

## 1. Purpose
Layer Visualization provides a visual representation of experimental data (heatmaps) on the map. It allows users to see high-density "interest zones" which inform their routing decisions and experimental analysis.

## 2. Functional Overview
- **Heatmap Rendering**: Unlike traditional blurry heatmaps, this system uses a **grid-based vector approach** for precision.
- **Dynamic Scoring**: Data is fetched from Parquet datasets, normalized, and converted into color-coded polygons.
- **Shared Utilities**: The visualization logic is centralized to ensure the Routing page and the Explore Map show identical data representations.

## 3. File Structure and Responsibilities
- `app/components/OpenLayers/helpers/heatmapRenderer.ts`: The core rendering engine. Contains `renderHeatmapPoints` which creates OpenLayers Polygons.
- `app/routes/api.routing.datasetPoints.tsx`: Backend API that reads dataset files and returns coordinates with calculated scores.
- `app/routes/api.routing._shared.dataset.ts`: Low-level data reader that handles Parquet schema mapping and numeric column extraction.

## 4. Workflow
1.  **Frontend Request**: Map component calls `/api/routing/datasetPoints` with `datasetName` and the list of `layers` (features) to visualize.
2.  **Data Fetching**: Backend reads the `.parquet` file from the `datasets/` folder.
3.  **Normalization**: The score for each point is calculated as the weighted sum of its features, normalized against the dataset's min/max values.
4.  **Geometry Generation**: `renderHeatmapPoints` takes the points and generates 10m x 10m polygons (`Polygon`).
5.  **Coloring**: `getHeatmapColor` applies an `rgba` color based on the score ratio (Green for low, Yellow/Orange for high).

## 5. Deployment / Configuration Notes
- **Grid Size**: Fixed at 10 meters (calculated using `metersToLat/Lon` projection helpers).
- **Z-Index**: Heatmap layers are typically set to `zIndex: 5` to appear above the base map but below markers.

## 6. Maintenance Guide
- **Changing the Color Palette**: Modify the `stops` array in `getHeatmapColor` within `app/components/OpenLayers/helpers/heatmapRenderer.ts`.
- **Performance**: A hard rendering limit of 1000 points is often applied to prevent browser lag during pan/zoom.

## 7. Known Dependencies
- `parquetjs-lite`: For reading the binary dataset format.
- `ol/layer/Vector`: Used for rendering the polygons.

## 8. Troubleshooting
- **Heatmap Not Showing**: Verify the `layers` array sent to the API matches the column names in the Parquet file.
- **Misaligned Grid**: Check the `refLat` calculation in `renderHeatmapPoints`; grid size projection varies by latitude.

## 9. References
- `app/components/OpenLayers/helpers/heatmapRenderer.ts`
- `app/routes/api.routing.datasetPoints.tsx`
- `app/routes/api.routing._shared.dataset.ts`

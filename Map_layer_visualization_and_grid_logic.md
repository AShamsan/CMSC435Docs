# Map Layer Visualization and Grid Logic - Developer Documentation

Location: `app/components/OpenLayers/helpers/heatmapRenderer.ts`, `app/routes/api.routing.datasetPoints.tsx`, `app/routes/api.routing._shared.dataset.ts`, `app/routes/routing/route.tsx`  
Route/API: `/api/routing/datasetPoints`  
Access: Internal fetch from the Routing page and Explore Map  
Related UI Area: Routing Sidebar -> Map Layers

---

## 1. Purpose

This document explains how EcoWell visualizes environmental data layers on the map and why the system uses grid-based visualization instead of a traditional blurry heatmap.

The layer visualization system allows users to see experimental data directly on the campus map. These layers can represent environmental factors such as shade, biodiversity, slope, tree coverage, noise, greenery, or any other approved dataset columns uploaded through the experimentation workflow.

This feature is important because EcoWell is not only a routing application. It is also a research platform. Researchers and users need to understand why certain routes are generated and how environmental data affects route optimization.

The visualization system helps answer questions like:

- Where are the strongest environmental data points?
- Which areas of campus have higher values for selected features?
- Why did the route engine prefer one path over another?
- How do selected experiment layers appear spatially on the map?
- Are the uploaded datasets being displayed correctly?

Instead of showing a vague heatmap, the system renders small grid squares. Each square represents an actual data point or scored location. This makes the visualization more precise and easier to explain during testing, research, and demos.

---

## 2. Why This Documentation Was Combined

This document combines the previous documentation files:

- `layer_visualization.md`
- `map_layer_decisions.md`

These two files were combined because they described the same system from two sides.

`layer_visualization.md` explained how heatmap-style layers are rendered.

`map_layer_decisions.md` explained why the system uses grid-based visualization and how the layer toggles work.

Keeping them separate caused repeated explanations about the 10 meter grid, heatmap rendering, and dataset point visualization. Combining them makes the documentation easier to maintain and gives developers one complete reference for map layer visualization.

---

## 3. Functional Overview

The map layer visualization system allows the frontend to request environmental dataset points, score those points based on selected layers, and render the result on the OpenLayers map.

At a high level, the system works like this:

1. The user selects one or more visual layers.
2. The frontend sends the selected dataset and layers to the backend.
3. The backend reads the dataset file.
4. Dataset values are normalized and scored.
5. The frontend receives scored coordinates.
6. The map renders those coordinates as 10 meter by 10 meter grid squares.
7. The user can visually compare areas of higher and lower environmental value.

The result looks similar to a heatmap, but it is not a traditional heatmap. It is a vector grid layer made from square polygons.

This makes the data easier to reason about because each visible square represents a real scored point instead of a smoothed or blended estimate.

---

## 4. Main Design Decision

### Grid-based visualization instead of a traditional heatmap

The system uses grid-based vector polygons instead of OpenLayers `HeatmapLayer`.

A traditional heatmap blends nearby points together. This can make the map look smooth and visually appealing, but it can also hide the actual structure of the dataset.

For EcoWell, that is a problem because researchers need to see the real distribution of uploaded data. If the system smooths the values too much, it may look like a high-value area exists where there is no actual data point.

Because of that, the project uses 10 meter by 10 meter grid squares.

Each grid square represents a scored data point.

This approach was chosen because:

- It shows where the data actually exists.
- It avoids misleading interpolation.
- It gives researchers a more honest view of the dataset.
- It supports acceptance testing better.
- It makes the route scoring easier to explain.
- It keeps the visualization close to the uploaded dataset resolution.
- It works well with Juanita's preference for visible 10 meter by 10 meter grid-based data.

This decision makes the visualization more useful for research, debugging, and validation.

---

## 5. What the Layer Visualization Shows

The layer visualization can show any numeric dataset columns that are approved and available for routing or experimentation.

Example layer types include:

- Shade
- Biodiversity
- Tree canopy
- Green space
- Slope
- Noise
- Wellness score
- Custom uploaded research columns

The system does not hardcode meaning into every layer. Instead, it uses selected dataset columns and their weights.

This means the same rendering system can support different experiments without changing the map code each time.

---

## 6. Relationship to Routing

The layer visualization system is closely related to the routing engine.

The routing engine uses selected environmental layers to score candidate routes. The visualization system lets the user see those layers on the map.

This helps connect the algorithm to the UI.

For example:

1. A researcher selects an experiment that gives high weight to shade.
2. The map displays shade-related dataset points.
3. The routing engine generates routes that prefer areas with higher shade scores.
4. The user can visually compare the generated route with the displayed shade layer.

This makes the routing decision more transparent.

The visualization does not generate the route by itself. It only displays the dataset values that may influence route scoring.

---

## 7. File Structure and Responsibilities

### `app/components/OpenLayers/helpers/heatmapRenderer.ts`

This file contains the main frontend rendering logic for the map layer visualization.

Main responsibilities:

- Convert scored dataset points into OpenLayers features.
- Create 10 meter by 10 meter square polygons.
- Apply colors based on point scores.
- Return a vector layer that can be added to the OpenLayers map.
- Keep heatmap/grid rendering reusable between the Routing page and Explore Map.

Important functions:

- `renderHeatmapPoints`
- `getHeatmapColor`

This file is the main place to update if the visual style, grid size, or color mapping changes.

---

### `app/routes/api.routing.datasetPoints.tsx`

This API route provides scored dataset points to the frontend.

Main responsibilities:

- Receive the selected dataset name.
- Receive selected layers and weights.
- Load dataset points from the backend.
- Normalize selected feature values.
- Calculate a weighted score for each point.
- Return coordinates and scores to the frontend.

This file connects the map UI to the dataset storage system.

---

### `app/routes/api.routing._shared.dataset.ts`

This file contains shared dataset loading logic.

Main responsibilities:

- Read dataset files from storage.
- Load Parquet data.
- Support CSV fallback when needed.
- Extract numeric columns.
- Map dataset fields into usable route and visualization points.
- Provide reusable dataset utilities for routing APIs.

This file is used by both routing and visualization logic.

---

### `app/routes/routing/route.tsx`

This file manages the Routing page state.

For map layer visualization, it handles:

- Selected experiment state.
- Selected visualized layers.
- Layer toggle behavior.
- Passing active layers into map components.
- Triggering refetches when the layer selection changes.

Important state:

```text
visualizedLayers
setVisualizedLayers
```

This file is important because it decides which layers the user wants to see.

---

### `app/components/OpenLayers/routing/components/RoutingFilterPanel.client.tsx`

This component manages parts of the map filtering UI.

Main responsibilities:

- Show map-related controls.
- Allow users to toggle map layers or filters.
- Help separate UI controls from the main route file.

Depending on the current implementation, this component may handle plant, marker, and layer-related toggles.

---

## 8. Data Flow

The full data flow looks like this:

```text
User selects layer
        |
        v
Routing page updates visualizedLayers state
        |
        v
Map component requests /api/routing/datasetPoints
        |
        v
Backend loads dataset from storage
        |
        v
Backend normalizes selected feature values
        |
        v
Backend calculates weighted point scores
        |
        v
Frontend receives scored coordinates
        |
        v
heatmapRenderer creates 10m x 10m polygons
        |
        v
OpenLayers displays grid layer on the map
```

This flow keeps the frontend focused on rendering and the backend focused on dataset scoring.

---

## 9. Frontend Workflow

### Step 1: User opens map layers

The user opens the layer control area from the Routing page or Explore Map.

The UI displays available environmental layers based on the selected experiment or dataset.

---

### Step 2: User toggles layers

The user enables or disables one or more layers.

For example, the user may turn on:

- Shade
- Biodiversity
- Tree canopy

When the user changes layer selection, the frontend updates the layer state.

---

### Step 3: `visualizedLayers` updates

The Routing page updates the selected layer list.

Example state:

```text
visualizedLayers = ["shade", "biodiversity"]
```

This state controls what the map should fetch and display.

---

### Step 4: Frontend requests dataset points

The map component sends a request to:

```text
/api/routing/datasetPoints
```

The request should include:

- `datasetName`
- selected layers
- selected weights if applicable
- experiment context if needed

---

### Step 5: Map clears old layer

Before drawing the new layer, the map should remove or replace the previous visualization layer.

This prevents multiple stale layers from stacking on top of each other.

---

### Step 6: Map renders updated grid

The frontend receives scored points and passes them to:

```text
renderHeatmapPoints
```

The returned OpenLayers vector layer is added to the map.

---

## 10. Backend Workflow

### Step 1: API receives request

The backend receives a request at:

```text
/api/routing/datasetPoints
```

The request identifies which dataset and layers should be visualized.

---

### Step 2: Dataset is loaded

The backend attempts to load the dataset from the storage system.

The preferred source is usually the optimized dataset file, such as a Parquet file in the `datasets/` directory.

Depending on the implementation, fallback sources may include database rows or uploaded CSV files.

---

### Step 3: Numeric columns are extracted

Only numeric dataset values can be scored and visualized.

Non-numeric columns, date columns, text columns, or metadata columns should not be used as layer scores.

---

### Step 4: Values are normalized

Each selected layer may use a different scale.

For example:

```text
shade: 0 to 100
biodiversity_count: 0 to 20
noise: 30 to 90
slope: 0 to 15
```

These values cannot be compared directly.

The backend normalizes each selected layer so the values are placed on a shared scale.

Usually, this means converting values into the range:

```text
0 to 1
```

Normalization allows the system to combine different environmental factors fairly.

---

### Step 5: Weighted scores are calculated

After normalization, each layer value is multiplied by its selected weight.

General scoring idea:

```text
normalized value * layer weight
```

If multiple layers are selected, the weighted values are added together.

General weighted score idea:

```text
score = (normalized layer 1 * weight 1) + (normalized layer 2 * weight 2) + ...
```

This is the same general idea used by the routing engine's Multi-Criteria Decision Analysis and Weighted Linear Combination approach.

---

### Step 6: Scored points are returned

The backend returns points with:

- latitude
- longitude
- score
- possibly raw or normalized values if needed for debugging

The frontend then uses those points for rendering.

---

## 11. Grid Logic

The visualization uses square polygons instead of point markers.

Each data point is converted into a small square centered on the point coordinate.

Current grid size:

```text
10 meters by 10 meters
```

The grid square is calculated by converting meters into latitude and longitude distance.

Important logic:

```javascript
const dx2 = (10 * metersToLon) / 2;
const dy2 = (10 * metersToLat) / 2;
```

This creates half of the square width and half of the square height.

The square is centered on the original data point.

Conceptually:

```text
center point = dataset coordinate
square width = 10 meters
square height = 10 meters
```

This gives each data point a visible area on the map while keeping the display close to the actual dataset location.

---

## 12. Color Logic

Each grid square receives a color based on its score.

The color is generated by:

```text
getHeatmapColor
```

The current approach uses color stops to show score intensity.

General meaning:

- Low score means lower visual intensity.
- Medium score means moderate visual intensity.
- High score means stronger visual intensity.

The exact colors are controlled in the color stop array inside:

```text
app/components/OpenLayers/helpers/heatmapRenderer.ts
```

If the colors need to be changed for accessibility, demo clarity, or visual consistency, update the color stop logic there.

---

## 13. Layer Ordering

The grid layer should appear above the base map but below the most important interactive elements.

Recommended order:

```text
Base map
Environmental grid layer
Route line
Waypoint markers
Popups and UI controls
```

Typical grid layer z-index:

```text
zIndex: 5
```

If the grid layer covers markers or route lines, reduce its z-index or increase the z-index of the route and marker layers.

---

## 14. Why This Helps Acceptance Testing

The grid visualization is useful for acceptance testing because it makes the data visible and explainable.

During testing, stakeholders can see:

- Whether uploaded data appears on the map.
- Whether selected layers update correctly.
- Whether high-value areas are visible.
- Whether the route appears to respond to environmental data.
- Whether the grid matches the expected 10 meter by 10 meter style.

This is better than only showing route output because it helps testers understand what the algorithm is using.

It also helps identify problems early. For example, if a route does not seem to follow a high-value area, the team can check whether the layer is even visible in that area.

---

## 15. Deployment and Configuration Notes

### Dataset files

Dataset files must exist in the expected dataset storage location.

Common location:

```text
datasets/
```

The dataset name sent from the frontend must match the backend dataset name.

If the dataset name does not match, the API may return no points or low scores.

---

### Coordinate requirements

Dataset rows must include valid coordinate fields.

The visualization cannot render a point if latitude or longitude is missing or invalid.

---

### Numeric layer requirements

Only numeric columns should be used for scoring.

If a selected layer is text, date, boolean, or missing, it may fail or produce incorrect scores.

---

### OSRM is not required for visualization

The layer visualization API does not need OSRM to display data points.

OSRM is required for route generation, but not for drawing the environmental grid layer.

This is useful because developers can debug dataset visualization separately from route generation.

---

### Browser performance

Rendering many polygons can slow down the map.

A hard rendering limit may be used to prevent lag.

A common limit is around:

```text
1000 points
```

If the dataset is very large, consider loading only points in the current map bounds.

---

## 16. Maintenance Guide

### Changing the grid size

To change the grid size globally, update the grid size logic in:

```text
app/components/OpenLayers/helpers/heatmapRenderer.ts
```

Current size:

```text
10 meters
```

If the grid size changes, test the following:

- Visual accuracy
- Map readability
- Performance
- Acceptance testing expectations
- Dataset resolution compatibility

A larger grid may make the map easier to see but less precise.

A smaller grid may be more precise but harder to notice.

---

### Adding a user-selectable grid size

To let users change grid size from the UI:

1. Add a `gridSize` state in `app/routes/routing/route.tsx`.
2. Add a UI control for selecting grid size.
3. Pass `gridSize` to the map component.
4. Pass `gridSize` into `renderHeatmapPoints`.
5. If needed, include `gridSize` in the dataset points API request.
6. Test with multiple grid sizes.

Recommended grid size options:

```text
5 meters
10 meters
25 meters
50 meters
```

Do not add too many options because it may confuse users.

---

### Adding a new visual layer

To add a new visual layer:

1. Make sure the dataset column exists.
2. Make sure the column is numeric.
3. Add the layer to the experiment or layer selection UI.
4. Confirm the selected layer is included in `visualizedLayers`.
5. Confirm the frontend sends it to `/api/routing/datasetPoints`.
6. Confirm the backend includes it in scoring.
7. Confirm the map renders the layer correctly.

---

### Changing the color palette

Update:

```text
getHeatmapColor
```

This function controls how scores map to colors.

When changing colors, consider:

- Accessibility
- Contrast against the base map
- Visibility on mobile
- Whether low and high scores are easy to distinguish
- Whether the route line remains visible above the grid

---

### Improving performance

If the map becomes slow, consider:

1. Reducing the number of rendered points.
2. Rendering only points inside the current map bounds.
3. Using zoom-based rendering.
4. Simplifying polygon geometry.
5. Caching rendered layers.
6. Debouncing layer updates.
7. Avoiding unnecessary re-fetches.

---

### Changing normalization logic

Normalization should be updated carefully because it affects both visualization and route scoring.

If normalization changes, test:

- Single-layer visualization.
- Multi-layer visualization.
- High and low score colors.
- Routing behavior with the same experiment.
- Datasets with very small value ranges.
- Datasets with missing values.

---

### Updating layer toggle behavior

Layer toggles should update the map clearly and quickly.

When changing toggle behavior, confirm:

- Turning a layer on displays it.
- Turning a layer off removes it.
- Switching experiments clears old layers.
- Empty layer selection removes the visualization.
- Multiple selected layers combine correctly.

---

## 17. Testing Checklist

Use this checklist when testing map layer visualization.

### Basic rendering

- [ ] The map loads successfully.
- [ ] The layer control UI appears.
- [ ] Available layers match the selected experiment or dataset.
- [ ] Turning on a layer displays grid squares.
- [ ] Turning off a layer removes grid squares.
- [ ] Switching layers updates the map.

---

### Dataset validation

- [ ] Dataset name matches backend storage.
- [ ] Dataset has valid latitude and longitude.
- [ ] Selected columns are numeric.
- [ ] Missing values do not break rendering.
- [ ] Empty datasets are handled safely.

---

### Visual accuracy

- [ ] Grid squares appear in the correct campus locations.
- [ ] Grid squares are about 10 meters by 10 meters.
- [ ] High score areas are visually distinguishable.
- [ ] Low score areas are visually distinguishable.
- [ ] The grid does not hide route lines or markers.

---

### Performance

- [ ] Map does not freeze when layer is enabled.
- [ ] Panning remains smooth.
- [ ] Zooming remains usable.
- [ ] Large datasets are limited or handled safely.

---

### Routing relationship

- [ ] The same selected layers can influence route scoring.
- [ ] The displayed layer helps explain the generated route.
- [ ] Changing selected layers updates both the visualization and route behavior when applicable.

---

## 18. Troubleshooting

### Heatmap or grid layer is not showing

Check:

1. The selected dataset exists.
2. The dataset name matches the stored file name.
3. The selected layer names match real dataset columns.
4. The selected columns are numeric.
5. The API request to `/api/routing/datasetPoints` succeeds.
6. The API response includes points.
7. The map is adding the returned vector layer.
8. The layer z-index is high enough to appear above the base map.

---

### Layer toggle does not update the map

Check:

1. `visualizedLayers` is updating correctly.
2. The map component receives the new layer list.
3. The dataset points API is called again.
4. Old layers are removed before new layers are added.
5. The selected experiment did not change without resetting layer state.

---

### Grid squares appear in the wrong location

Check:

1. Latitude and longitude fields are correct.
2. Coordinates are not reversed.
3. Projection conversion is correct.
4. The map uses the expected coordinate system.
5. The grid calculation accounts for latitude.

---

### Grid squares look stretched

Longitude and latitude do not convert to meters equally.

Check the conversion helpers in:

```text
renderHeatmapPoints
```

The longitude distance changes depending on latitude, so the grid calculation must account for the campus latitude.

---

### Polygons overlap too much

This may happen if dataset points are close together.

Check:

1. Dataset density.
2. Grid size.
3. Zoom level.
4. Whether duplicate points exist.
5. Whether the grid should be smaller.

---

### Colors look wrong

Check:

1. Score normalization.
2. Layer weights.
3. Color stop logic.
4. Whether all scores are nearly the same.
5. Whether missing values are being treated as zero.

If all values are close together, the map may show very similar colors.

---

### Map becomes slow

This usually means too many polygons are being rendered.

Possible fixes:

1. Add a point limit.
2. Render only visible points.
3. Add zoom-based loading.
4. Simplify the geometry.
5. Avoid frequent re-rendering during map movement.

---

### Layer appears above markers or route lines

Check z-index values.

The environmental grid layer should be below markers and route lines.

---

## 19. Known Dependencies

| Dependency | Usage |
|---|---|
| OpenLayers | Main map rendering system |
| `ol/layer/Vector` | Renders grid polygon layers |
| `ol/source/Vector` | Stores grid polygon features |
| `ol/Feature` | Creates map features |
| `ol/geom/Polygon` | Creates 10 meter by 10 meter grid squares |
| `parquetjs-lite` | Reads Parquet dataset files |
| Remix API routes | Provides dataset points to the frontend |
| React state | Tracks selected visualized layers |

---

## 20. Developer Notes

### Keep rendering logic reusable

The rendering logic should stay inside:

```text
app/components/OpenLayers/helpers/heatmapRenderer.ts
```

This avoids duplicating heatmap/grid rendering logic across the Routing page and Explore Map.

---

### Keep dataset scoring on the backend

The frontend should not be responsible for reading or scoring full datasets.

The backend should handle:

- Dataset loading
- Normalization
- Weighted scoring
- Missing value handling

The frontend should handle:

- Requesting the data
- Rendering the returned points
- Updating the map layer

---

### Keep UI state separate from rendering

The Routing page should manage which layers are selected.

The heatmap renderer should only care about how to draw the points.

This separation makes the system easier to maintain.

---

### Avoid misleading visuals

Do not switch back to a blurry heatmap unless there is a strong reason.

The grid approach was chosen because it better supports research accuracy and acceptance testing.

If a traditional heatmap is added later, it should be optional and clearly labeled as a smoothed visualization.

---

## 21. References

- `app/components/OpenLayers/helpers/heatmapRenderer.ts`
- `app/routes/api.routing.datasetPoints.tsx`
- `app/routes/api.routing._shared.dataset.ts`
- `app/routes/routing/route.tsx`
- `app/components/OpenLayers/routing/components/RoutingFilterPanel.client.tsx`

---

## 22. What Changed From the Previous Two Files

### Combined files

The following two files were combined:

- `layer_visualization.md`
- `map_layer_decisions.md`

New recommended file name:

```text
map_layer_visualization.md
```

---

### Main changes

- Combined repeated explanations about heatmap rendering and grid logic.
- Kept the required `Developer Documentation` title style.
- Added a clearer purpose section.
- Added a stronger explanation of why grid-based visualization is used.
- Explained the relationship between layer visualization and the routing engine.
- Expanded the file responsibility section.
- Added a full frontend workflow.
- Added a full backend workflow.
- Added a dedicated grid logic section.
- Added a dedicated color logic section.
- Added deployment and configuration notes.
- Added a larger maintenance guide.
- Added a testing checklist.
- Added more troubleshooting cases.
- Added developer notes for future maintainers.

---

## 23. Files This Replaces

This combined file should replace:

- `layer_visualization.md`
- `map_layer_decisions.md`

After adding this file, those two old files can be removed to avoid duplicate documentation.

Recommended final structure:

```text
docs/
  map_layer_visualization.md
  routing_architecture.md
  routing_engine_design.md
  ui_design_choices.md
```

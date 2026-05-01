# Routing Engine and Algorithm Design — Developer Documentation

Location: `app/routes/api.routing.*`, `app/routes/api.routing._shared.geo.ts`
Route/API: `/api/routing/generateCandidates`, `/api/routing/scoreCandidates`
Access: Internal API calls from Routing Dashboard and Explore Map

## 1. Purpose
The routing engine is designed to generate and score walking paths based on user-defined constraints (start, end, loop mode, target distance) and experimental data (heatmaps). It exists to provide researchers and users with optimized routes that prioritize areas of high interest (defined by datasets like wellness or biodiversity) while maintaining path efficiency.

## 2. Functional Overview
The engine operates in a multi-stage pipeline:
1.  **Anchor Picking**: Identifying high-value points from datasets near the route.
2.  **Candidate Generation**: Using OSRM to find multiple valid path alternatives between waypoints and anchors.
3.  **Scoring**: Evaluating each candidate based on feature density (heatmap), distance accuracy (loop/point-to-point), and path overlap (to avoid repetitive segments).

## 3. File Structure and Responsibilities
- `app/routes/api.routing.generateCandidates.tsx`: Orchestrates the fetching of raw path alternatives from OSRM based on waypoints and anchors.
- `app/routes/api.routing.scoreCandidates.tsx`: Normalizes and weights the scores of candidate routes to select the "Best Index."
- `app/routes/api.routing._shared.geo.ts`: Core math library for Haversine distances, point-to-segment distance, overlap calculation, and the `scoreRoute` function.
- `app/routes/api.routing._shared.dataset.ts`: Handles loading Parquet/CSV data into memory for spatial scoring.

## 4. Workflow
1.  **Request**: Frontend sends a list of waypoints, routing mode, and active experiment layers.
2.  **Generate Candidates**:
    - `getPointToPointCandidates` or `getLoopCandidates` is called.
    - OSRM is queried for a "Base Path" through waypoints.
    - High-score "Anchors" from the dataset are injected into OSRM queries to create "Detour" alternatives.
3.  **Score Candidates**:
    - `scoreRoute` iterates over each candidate's coordinates.
    - **Feature Score**: Sum of normalized values from dataset points within 220m (`DEFAULT_RADIUS_METERS`) of the path.
    - **Overlap Penalty**: Ratio of the path that repeats segments (calculated in `calculateOverlapRatio` with an 18m threshold).
    - **Distance Error**: Penalty for deviating from the user's `targetDistance`.
4.  **Selection**: The `bestIndex` is returned based on the highest `totalScore`.

## 5. Deployment / Configuration Notes
- **OSRM Service**: Requires access to an OSRM backend (configured via `ROUTING` or `OSRM_UNLOCKED` env vars).
- **Dataset Files**: Parquet files must exist in the `datasets/` directory with names matching the `datasetName`.
- **Environment Variables**: `ROUTING` should point to a valid OSRM base URL (e.g., `https://router.project-osrm.org`).

## 6. Maintenance Guide
- **Adjusting Penalties**: Modify the normalization weights in `app/routes/api.routing.scoreCandidates.tsx` (e.g., `featureW`, `penaltyBudget`).
- **Changing Search Radius**: Update `DEFAULT_RADIUS_METERS` in `app/routes/api.routing._shared.types.ts` to change how far the algorithm looks for data points around a path.
- **Adding Profiles**: Update `profiles` array in `fetchOsrmPath` to support new transport modes (e.g., bike).

## 7. Known Dependencies
- `parquetjs-lite`: Reading optimized spatial datasets.
- `haversine-distance`: (Logic implemented manually in `haversineMeters`) for spherical geometry.
- `node-fetch`: Interfacing with external OSRM APIs.

## 8. Troubleshooting
- **No Candidates Found**: Check OSRM service availability. OSRM may fail if points are too far from a routable path (e.g., middle of a building).
- **Low Feature Scores**: Ensure the `datasetName` matches the filename in `datasets/` and that the experiment layers match column names in the data.

## 9. References
- `app/routes/api.routing.generateCandidates.tsx`
- `app/routes/api.routing.scoreCandidates.tsx`
- `app/routes/api.routing._shared.geo.ts`
- `app/routes/api.routing._shared.dataset.ts`

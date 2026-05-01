# Routing Engine and Algorithm Design - Developer Documentation

Location: `app/routes/api.routing.*`, `app/routes/api.routing._shared.geo.ts`  
Route/API: `/api/routing/generateCandidates`, `/api/routing/scoreCandidates`  
Access: Internal API calls from Routing page and Explore Map

---

## 1. Purpose

The routing engine generates and scores walking routes using user input, campus map data, OSRM route geometry, and environmental experiment datasets.

The main goal is to generate routes that are both usable walking paths and optimized for selected environmental factors.

For EcoWell, this means a route should not only get the user from one place to another. It should also support the selected experiment goal, such as increasing exposure to shade, biodiversity, greenery, tree canopy, or other wellness-related environmental features.

This document explains the algorithm structure, how route candidates are generated, how they are scored, and how the best route is selected.

---

## 2. Functional Overview

The routing engine works in two major stages:

1. Candidate generation
2. Candidate scoring

Candidate generation creates possible route options.

Candidate scoring evaluates those options and selects the best one.

The scoring system follows a Multi-Criteria Decision Analysis approach using Weighted Linear Combination.

This means the engine combines multiple normalized scoring factors into one final score.

The engine can consider:

- Environmental feature score
- User-selected layer weights
- Distance accuracy
- Route overlap
- Target loop distance
- Routing mode
- Dataset availability
- OSRM route feasibility

The purpose is to avoid selecting routes based on only one factor. A route should score well environmentally, but it should also remain practical for the user to walk.

---

## 3. Main Algorithm Idea

The routing engine uses Multi-Criteria Decision Analysis, also called MCDA.

The selected data layers are combined using Weighted Linear Combination, also called WLC.

The simplified scoring idea is:

`Final Score = Weighted Environmental Score - Penalties`

The environmental score comes from selected dataset layers.

The penalties help prevent bad route behavior.

Examples of penalties include:

- Distance penalty
- Overlap penalty
- Target distance error penalty

This matters because a route that passes through high-scoring environmental data may still be bad if it is too long, repeats too much, or ignores the user's requested route mode.

---

## 4. Why MCDA and WLC Are Used

EcoWell routing depends on multiple environmental factors.

For example, an experiment may want to prioritize:

- More shade
- More biodiversity
- Less noise
- More greenery
- Less slope

These features do not all use the same unit or range.

Weighted Linear Combination allows the system to combine them into one score after normalization.

This fits EcoWell because researchers can adjust the weights of different layers and test how those weights affect route selection.

Example:

`totalFeatureScore = shadeScore * shadeWeight + biodiversityScore * biodiversityWeight + noiseScore * noiseWeight`

The exact implementation may include additional normalization and penalties, but the main idea is that weighted factors are combined into one route score.

---

## 5. Why Normalization Is Needed

Environmental layers can have different units and value ranges.

For example:

- Shade may be stored as a percent.
- Biodiversity may be stored as a count.
- Noise may be stored in decibels.
- Slope may be stored as a grade.
- Tree canopy may be stored as coverage.

These values cannot be compared directly.

A biodiversity value of `20` does not mean the same thing as a noise value of `20`.

Because of this, the engine normalizes selected layer values before combining them.

Normalization usually converts values to a shared range:

`0 to 1`

After normalization, the system can apply weights and combine layers more fairly.

This is necessary for MCDA and WLC to work correctly.

---

## 6. File Structure and Responsibilities

### `app/routes/api.routing.generateCandidates.tsx`

This API generates route candidates.

Main responsibilities:

- Receive routing input from the frontend.
- Generate point-to-point candidates.
- Generate loop candidates.
- Query OSRM for route geometry.
- Create base route candidates.
- Add dataset-based anchor points when useful.
- Return multiple possible route options.

This file is responsible for producing route choices, not deciding which one is best.

---

### `app/routes/api.routing.scoreCandidates.tsx`

This API scores route candidates.

Main responsibilities:

- Receive candidate routes.
- Load selected dataset information.
- Calculate environmental feature scores.
- Apply distance and overlap penalties.
- Normalize score components when needed.
- Select the best candidate route.
- Return the best candidate index and score information.

This file is responsible for deciding which route candidate is best.

---

### `app/routes/api.routing._shared.geo.ts`

This file contains shared geographic math utilities.

Main responsibilities:

- Calculate Haversine distance.
- Calculate point-to-segment distance.
- Calculate route overlap.
- Score route geometry against nearby dataset points.
- Support route distance and geometry comparisons.

Important functions may include:

- `haversineMeters`
- `pointToSegmentDistanceMeters`
- `calculateOverlapRatio`
- `scoreRoute`

This file keeps math logic reusable across routing APIs.

---

### `app/routes/api.routing._shared.dataset.ts`

This file handles dataset loading for routing and visualization.

Main responsibilities:

- Load Parquet data.
- Load CSV fallback data if needed.
- Extract numeric columns.
- Return dataset points for scoring.
- Support shared dataset access for route scoring and map visualization.

This file is important because route scoring depends on environmental data being loaded correctly.

---

### `app/routes/api.routing._shared.types.ts`

This file stores shared routing types and constants.

Main responsibilities:

- Define route API payload types.
- Define candidate route types.
- Define scoring-related constants.
- Keep frontend and backend routing structures consistent.

Important constant:

`DEFAULT_RADIUS_METERS`

This controls how far the engine searches around a route when looking for nearby dataset points.

---

## 7. Candidate Generation Workflow

### Step 1: Frontend sends route request

The frontend sends route information to the backend.

The request may include:

- Start point
- End point
- Optional waypoints
- Routing mode
- Target loop distance
- Selected experiment
- Selected layers
- Dataset name

---

### Step 2: Engine chooses route mode

The engine chooses the correct generation method based on the selected mode.

For point-to-point mode, it uses point-to-point candidate generation.

For loop mode, it uses loop candidate generation.

---

### Step 3: OSRM creates a base route

The engine sends the main coordinates to OSRM.

OSRM returns a walkable route geometry.

This becomes the base route candidate.

The base route is important because it gives the engine a normal route to compare against optimized alternatives.

---

### Step 4: Dataset anchors are selected

If experiment data is available, the engine can look for high-scoring dataset points near the route area.

These high-value points can be used as anchors.

An anchor is a point that the engine may try to route through to improve the environmental score.

For example, if a shade-heavy experiment is selected, the engine may prefer anchor points near high-shade areas.

---

### Step 5: Alternative candidates are generated

The engine creates additional route candidates by using different combinations of points and anchors.

Possible candidates include:

- Base route
- Route through one anchor
- Route through multiple anchors
- Loop route alternatives
- Detour route alternatives

These candidates are later scored and compared.

---

## 8. Candidate Scoring Workflow

### Step 1: Each candidate is evaluated

The scoring API receives candidate routes.

Each candidate is evaluated separately.

The goal is to calculate a final score for each candidate.

---

### Step 2: Feature score is calculated

The engine checks dataset points near the route geometry.

Current search radius:

`220 meters`

For each nearby data point, the engine looks at selected layers and weights.

The layer values are normalized and combined into a weighted feature score.

The feature score represents how well the route matches the selected environmental goal.

---

### Step 3: Distance error is calculated

For loop mode, the user may provide a target distance.

The engine compares the candidate route distance to the target distance.

If the candidate is too short or too long, it receives a penalty.

This helps keep loop routes close to what the user requested.

---

### Step 4: Overlap penalty is calculated

The engine checks how much of the route repeats itself.

This is especially important for loop mode.

A loop route that goes out and comes back on the same path may be less useful than a route that forms a better loop.

Current overlap threshold:

`18 meters`

If route segments are close enough to count as repeated, the route receives an overlap penalty.

---

### Step 5: Final score is calculated

The final score combines the environmental feature score and penalties.

Simplified version:

`totalScore = featureScore - penaltyScore`

The route with the highest final score is considered the best route.

---

### Step 6: Best route is selected

After all candidates are scored, the engine returns:

`bestIndex`

The frontend uses this index to display the selected route.

The system may also return score details for debugging or UI explanation.

---

## 9. Scoring Factors

### Environmental feature score

This score measures how well a route passes near high-value dataset points.

It depends on:

- Selected experiment layers
- Layer weights
- Normalized values
- Distance between the route and data points

---

### Distance penalty

This penalty prevents routes from becoming too long or too inefficient.

It is especially important when environmental anchors create detours.

---

### Target distance error

This penalty applies mostly to loop mode.

It measures how far the generated loop is from the user's requested target distance.

---

### Overlap penalty

This penalty discourages routes from repeating the same path too much.

It helps make loop routes feel more like actual loops.

---

### Penalty budget

The scoring API may use a penalty budget to control how strongly penalties affect the final result.

If penalties are too weak, the system may choose routes that are environmentally strong but impractical.

If penalties are too strong, the system may ignore environmental goals and choose only the shortest or simplest route.

---

## 10. Relationship to Experiments

Experiments define which environmental layers matter and how much each one matters.

An experiment may include:

- Dataset name
- Layer list
- Weight for each layer
- Experiment name
- Approval status

Only approved or selected experiments should influence routing behavior.

Example:

`shade weight = 8`

`biodiversity weight = 5`

`noise weight = 2`

The routing engine uses these weights when calculating feature scores.

---

## 11. Relationship to Map Layer Visualization

The routing engine uses selected layers for scoring.

The map layer visualization uses selected layers for display.

They are related, but they are not identical.

The map visualization helps users see where environmental data exists.

The routing engine uses the data to choose better route candidates.

This relationship makes routing more explainable because users can see the data that influenced the selected route.

---

## 12. Deployment and Configuration Notes

### OSRM service

The routing engine requires access to an OSRM backend.

Common environment variables:

- `ROUTING`
- `OSRM_UNLOCKED`

Example OSRM base URL:

`https://router.project-osrm.org`

If OSRM is unreachable, candidate generation may fail.

---

### Dataset files

Dataset files must exist in the expected dataset storage location.

Common location:

`datasets/`

The dataset name sent by the frontend must match the backend dataset name.

If the dataset cannot be found, feature scoring may return low or empty results.

---

### Routing profile

OSRM profiles determine what kind of route is generated.

Possible profiles include:

- Walking
- Driving
- Biking

For EcoWell, walking is the main use case.

---

### Coordinates

All routing coordinates must be valid.

Invalid or reversed latitude and longitude values can cause route generation to fail.

---

## 13. Maintenance Guide

### Adjusting scoring weights

Scoring weights can be changed in:

`app/routes/api.routing.scoreCandidates.tsx`

Look for values related to:

- `featureW`
- `penaltyBudget`
- distance penalty
- overlap penalty

After changing scoring weights, test both point-to-point and loop routing.

---

### Changing the search radius

Update:

`DEFAULT_RADIUS_METERS`

This controls how far the engine looks from the route when finding nearby dataset points.

A larger radius may include more data but can make scores less precise.

A smaller radius may be more precise but may miss useful nearby environmental data.

---

### Changing overlap behavior

Overlap logic is handled in:

`calculateOverlapRatio`

If loop routes repeat too much or are punished too strongly, check:

- Overlap threshold
- Overlap penalty weight
- Candidate generation behavior

---

### Adding a new routing profile

To add a new routing profile:

1. Update the profile list in OSRM request logic.
2. Make sure the OSRM backend supports the profile.
3. Add UI support if users should select it.
4. Test the profile with campus coordinates.

---

### Adding a new scoring factor

To add a new scoring factor:

1. Add the new factor to candidate scoring.
2. Normalize the value if it uses a different scale.
3. Add a weight or penalty value.
4. Include it in the final score calculation.
5. Test whether route selection changes in the expected way.

---

### Updating dataset loading priority

If dataset loading priority changes, update the shared dataset loading file.

The routing engine should use a consistent data source order so results are predictable.

For example:

1. Parquet dataset files
2. Database dataset rows
3. Uploaded CSV fallback

The exact order should match the current project decision.

---

## 14. Testing Checklist

### Candidate generation

- [ ] Point-to-point route generates successfully.
- [ ] Loop route generates successfully.
- [ ] Route with waypoints generates successfully.
- [ ] Route fails safely when OSRM is unavailable.
- [ ] Route fails safely when coordinates are invalid.

---

### Candidate scoring

- [ ] Feature scores change when layer weights change.
- [ ] Distance penalty affects long detours.
- [ ] Overlap penalty affects repeated paths.
- [ ] Best route changes when experiments change.
- [ ] Missing datasets do not crash the application.

---

### Experiment behavior

- [ ] Selected experiment sends the correct dataset name.
- [ ] Selected experiment sends the correct layers.
- [ ] Selected experiment sends the correct weights.
- [ ] Approved experiments work correctly.
- [ ] Empty or invalid experiments are handled safely.

---

### Map relationship

- [ ] Displayed data layers match selected experiment layers.
- [ ] Route behavior roughly matches visible environmental layers.
- [ ] Route line remains visible above data layers.

---

## 15. Troubleshooting

### No candidates are found

Check:

1. OSRM is running or reachable.
2. Coordinates are valid.
3. Points are close to walkable paths.
4. Routing profile is valid.
5. The frontend sends the expected payload.

---

### Route ignores environmental layers

Check:

1. Selected experiment has active layers.
2. Layer names match dataset columns.
3. Dataset values are numeric.
4. Normalization is working.
5. Feature weight is not too low compared to penalties.
6. Dataset points are close enough to the route.

---

### Loop route is too long or too short

Check:

1. Target distance value.
2. Distance penalty weight.
3. Candidate generation logic.
4. OSRM route alternatives.
5. Anchor selection behavior.

---

### Route repeats too much

Check:

1. Overlap calculation.
2. Overlap threshold.
3. Overlap penalty weight.
4. Loop candidate generation.

---

### Feature scores are always low

Check:

1. Dataset name matches stored dataset.
2. Selected layers match dataset columns.
3. Dataset points are near the route.
4. Search radius is not too small.
5. Dataset numeric values are being parsed correctly.

---

### Route scoring feels too aggressive

If environmental scoring dominates too much, reduce feature weight or increase penalties.

If penalties dominate too much, increase feature weight or reduce penalty strength.

---

## 16. Known Dependencies

| Dependency | Usage |
|---|---|
| `parquetjs-lite` | Reads optimized dataset files |
| OSRM | Generates route geometry |
| Remix API routes | Provides backend routing endpoints |
| Shared geo utilities | Distance, overlap, and scoring math |
| Shared dataset utilities | Loads dataset values for scoring |

---

## 17. Developer Notes

- Keep candidate generation separate from candidate scoring.
- Keep geographic math in shared utility files.
- Normalize environmental layers before combining them.
- Do not let environmental score completely ignore route quality.
- Do not let penalties completely erase environmental optimization.
- Test routing behavior with real campus coordinates.
- Test both loop and point-to-point modes after scoring changes.

---

## 18. References

- `app/routes/api.routing.generateCandidates.tsx`
- `app/routes/api.routing.scoreCandidates.tsx`
- `app/routes/api.routing._shared.geo.ts`
- `app/routes/api.routing._shared.dataset.ts`
- `app/routes/api.routing._shared.types.ts`

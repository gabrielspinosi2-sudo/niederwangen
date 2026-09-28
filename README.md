# Niederwangen

A work-in-progress 3D open-world pizzeria / crime simulation set in **Niederwangen, Bern, Switzerland**.

The long-term idea is to let the player build and operate a pizzeria, prepare and deliver pizzas, hire staff, upgrade the business, advertise, buy vehicles and explore the village. The player can stay completely legitimate or take additional risks through illegal activities, with the pizzeria potentially acting as a front for laundering money. Police attention, reputation and the health of the business are intended to tie the legal and illegal sides together.

The repository is currently focused mainly on the **world/map prototype**: recreating Niederwangen at roughly real-world scale using official Swiss geodata, then bringing it into Unreal Engine as a playable 3D environment.

---

## Current project status

The current map pipeline is already working at prototype level:

- A roughly **2.016 km × 2.016 km** section of Niederwangen was defined in QGIS.
- Official Swiss terrain data from **swissALTI3D** was used as the terrain source.
- Official building geometry from **swissBUILDINGS3D** was imported into QGIS and Blender.
- The map was exported from QGIS into Blender for cleanup and scale correction.
- Terrain and building geometry were imported into Unreal Engine.
- A **Third Person** character was added so the map can be explored on foot.
- A proper Unreal **Landscape** heightmap was generated from swissALTI3D.
- swissTLM3D road data has been downloaded and clipped to the map area. The road network currently exists mainly as centerline data and still needs to be turned into actual in-game roads.

The project is still in the **blockout / world-building stage**. Gameplay systems such as the pizzeria, economy, deliveries, NPC schedules, police, crime, vehicles and money laundering are not implemented yet.

---

## Main tools

The project currently uses:

- **Unreal Engine 5.8** — game engine and final world assembly
- **Blender 5.2 LTS** — building cleanup, modelling and asset preparation
- **QGIS 3.44 LTR** — geodata processing
- **Git**
- **Git LFS** — strongly recommended for Unreal/Blender binary assets

Avoid upgrading the Unreal project to a different engine version without coordinating with the team first.

---

## Geodata sources

The current map is based on official Swiss geodata from **swisstopo**.

### swissALTI3D
Used for terrain elevation.

### swissBUILDINGS3D
Used for real-world building footprints, volumes and roof geometry.

### swissTLM3D
Used for the road and path network.

Coordinate reference system used during GIS work:

```text
EPSG:2056
CH1903+ / LV95
```

Current game-map boundary:

```text
xmin = 2594346
xmax = 2596362
ymin = 1196338
ymax = 1198354
```

This gives an area of:

```text
2016 m × 2016 m
```

### Attribution

The project uses swisstopo Open Government Data.

The final game and any public builds should contain an appropriate attribution, for example:

```text
Geodata: © swisstopo
```

Review the official swisstopo terms before redistributing raw geodata or publishing builds.

---

## Terrain pipeline

The terrain currently comes from **swissALTI3D**.

The raster was clipped in QGIS to the map boundary above.

For the Unreal Landscape, the clipped terrain was converted into a:

```text
1009 × 1009
16-bit grayscale heightmap
```

Measured elevation range:

```text
Minimum: 559.43298339844 m
Maximum: 673.279296875 m
Range:   113.84631347656 m
```

The heightmap was normalized to the full 16-bit range.

Current Unreal Landscape scale:

```text
X Scale = 200
Y Scale = 200
Z Scale = 22.2356
```

Why:

```text
1009 height samples = 1008 quads
1008 × 2 m = 2016 m
```

The older terrain imported through QGIS → OBJ → Blender → FBX should be kept as a **reference mesh** for checking alignment. The Unreal Landscape is the long-term terrain solution.

Do not delete the reference terrain until the Landscape, buildings and roads have been verified at multiple points across the map.

---

## Building pipeline

Buildings originated from **swissBUILDINGS3D**.

Current pipeline:

```text
swissBUILDINGS3D
        ↓
      QGIS
        ↓
   3D OBJ export
        ↓
     Blender
        ↓
cleanup / scale / optional separation
        ↓
       FBX
        ↓
  Unreal Engine
```

### Important scale note

The OBJ scene exported from QGIS originally arrived in Blender at a very small scale.

The original terrain was approximately:

```text
0.679 m × 0.679 m
```

instead of approximately:

```text
2016 m × 2016 m
```

The required scale correction was approximately:

```text
2969.07
```

**Terrain and buildings must be scaled together before buildings are separated.**

Do **not** take a separately extracted building and scale only that building by 2969.07. The GIS position is partly encoded in the mesh relative to the shared world origin, so scaling an isolated building around that origin also scales its distance from the origin and moves it to the wrong place.

Correct order:

```text
raw terrain + raw buildings
        ↓
scale complete GIS scene together
        ↓
Apply Scale
        ↓
verify terrain/building alignment
        ↓
only then separate individual buildings
```

---

## Blender conventions

Recommended Blender scene structure:

```text
NIEDERWANGEN
├── TERRAIN
│   └── terrain reference
│
├── BUILDINGS_RAW
│   ├── Building_Solid
│   ├── Building_Solid.001
│   ├── ...
│   └── original GIS building blocks
│
├── BUILDINGS_GAME
│   └── separated / edited game-ready buildings
│
├── ROADS
└── REFERENCE
```

### BUILDINGS_RAW

Treat this as reference data.

Avoid destructive edits. If a building needs to be modified, duplicate or extract it into `BUILDINGS_GAME`.

### Separating buildings

The QGIS OBJ export does not always produce one clean Blender object per building.

Some building solids contain many buildings and many disconnected faces.

Be careful with:

```text
Separate by Loose Parts
Merge by Distance
```

These operations can create huge numbers of objects or damage shading/normals if used blindly.

For important buildings, manually selecting the required geometry and separating it is often safer.

### Normals

Some imported GIS surfaces may have inconsistent normals.

Symptoms in Unreal:

- missing walls
- invisible faces
- very dark faces
- surfaces visible from only one direction

Useful Blender checks:

```text
Viewport Overlays → Face Orientation
```

and:

```text
Edit Mode
A
Shift + N
Recalculate Outside
```

`Two Sided` materials in Unreal can be useful as a debug test, but should not be used as a blanket fix for broken normals.

### Scale / transforms

Before final export:

```text
Rotation = 0 / 0 / 0
Scale    = 1 / 1 / 1
```

Use:

```text
Ctrl + A → Rotation & Scale
```

Do not casually change object origins or locations on GIS-derived objects unless you understand the effect on their map alignment.

---

## Unreal map setup

Suggested Content structure:

```text
Content/
└── Niederwangen/
    ├── Maps/
    ├── Terrain/
    ├── Buildings/
    ├── Roads/
    ├── Materials/
    ├── Vegetation/
    ├── Props/
    ├── Blueprints/
    └── Gameplay/
```

### Character

The project started from the Unreal **Vehicle Template**, but the **Third Person** content pack was later added for walking around the map.

The map should use a Third Person GameMode when testing on foot.

### Collision

The old FBX terrain used:

```text
Collision Complexity:
Use Complex Collision As Simple
```

The newer Unreal Landscape uses Unreal's native landscape collision system and should normally use:

```text
Collision Preset: BlockAll
```

If the player falls through the Landscape, verify:

- the player is actually above the new Landscape and not the old reference terrain
- Landscape collision is enabled
- the Third Person capsule uses the normal Pawn collision preset
- the Landscape collision data has been generated correctly

---

## Roads

Road data currently comes from **swissTLM3D**.

The relevant road layer was clipped in QGIS to the Niederwangen map boundary.

Important: the GIS road data is primarily a **centerline network**, not finished asphalt geometry.

Conceptually:

```text
GIS road centerline
        ↓
Unreal spline
        ↓
road mesh
        ↓
curbs / sidewalks / markings
```

Now that the terrain is becoming a proper Unreal Landscape, the preferred long-term solution is to use **Landscape Splines** or a custom road Blueprint based on normal Unreal Spline Components.

The final road system should eventually support:

- different road widths
- main roads
- residential roads
- narrow access roads
- sidewalks
- curbs
- lane markings
- intersections
- terrain deformation around roads

Do not model kilometres of road manually in Blender.

---

## Planned world-building workflow

Recommended order:

1. Verify Landscape scale and alignment.
2. Verify building alignment against the Landscape.
3. Get Landscape collision working reliably.
4. Build the main road network.
5. Create a Landscape material:
   - grass
   - dirt
   - forest floor
   - gravel
   - rock
6. Detail one small test area first.
7. Add foliage:
   - trees
   - bushes
   - hedges
   - grass
8. Replace important GIS buildings with detailed models.
9. Add props:
   - streetlights
   - traffic signs
   - fences
   - bins
   - hydrants
   - mailboxes
10. Build the pizzeria.
11. Begin actual gameplay systems.

Do not attempt to fully detail all 2 km² at once.

---

## Long-term gameplay concept

### Pizzeria simulation

The player should eventually be able to:

- prepare pizzas
- buy ingredients
- take orders
- deliver food
- improve the kitchen
- expand the menu
- advertise
- hire employees
- hire delivery drivers
- buy delivery vehicles
- manage wages and operating costs
- eventually run a high-end pizzeria

### Open world

The player can explore Niederwangen on foot and by vehicle.

The small map should favor density and familiarity over raw size. Players should gradually recognize homes, customers, shops and recurring NPCs.

### Legal progression

The entire game should remain playable through legitimate business growth.

```text
orders
  ↓
profit
  ↓
upgrades
  ↓
more customers
  ↓
more profit
```

### Crime systems

Optional illegal activities may eventually include:

- burglary
- theft
- fencing stolen goods
- illegal deliveries
- drug dealing
- money laundering through the pizzeria

These mechanics are intended as risky alternatives, not mandatory progression.

### Police / suspicion

Potential systems include:

- witnesses
- CCTV
- police stops
- suspicion level
- investigations
- searches
- confiscation
- damage to the pizzeria's reputation
- arrest / legal consequences

### Reputation

Possible separate reputation systems:

```text
Business Reputation
Underworld Reputation
```

---

## Git setup

Because Unreal and Blender projects contain large binary assets, use **Git LFS**.

Install it before working with the repository:

```bash
git lfs install
```

Then clone the repository normally:

```bash
git clone <repository-url>
cd <repository-folder>
git lfs pull
```

Open the `.uproject` file in the agreed Unreal Engine version.

---

## What should be committed

Typical Unreal project files that should be version controlled:

```text
Config/
Content/
Plugins/        # if project-specific
Source/         # if C++ is added
*.uproject
```

Useful source assets can also be committed if repository policy allows it:

```text
Blender/
QGIS/
SourceAssets/
```

Large binary files should use Git LFS.

Examples:

```text
*.uasset
*.umap
*.fbx
*.blend
*.tif
*.tiff
*.png
*.wav
```

---

## What should NOT be committed

Generated Unreal directories should normally be ignored:

```text
Binaries/
DerivedDataCache/
Intermediate/
Saved/
.vs/
```

Also avoid committing giant temporary GIS downloads unless they are intentionally part of the repository.

The raw swisstopo downloads are reproducible and can be stored outside Git if repository size becomes a problem.

---

## Suggested `.gitignore`

Use Epic's standard Unreal Engine `.gitignore` as the base.

At minimum:

```gitignore
Binaries/
DerivedDataCache/
Intermediate/
Saved/
.vs/

*.VC.db
*.opensdf
*.sdf
*.suo
*.user
*.xcodeproj
*.xcworkspace

# Local / temporary GIS data
GIS_Raw/
00_Geodata_RAW/
```

Do **not** ignore the Unreal `Content/` directory.

---

## Suggested Git LFS configuration

Example `.gitattributes`:

```gitattributes
*.uasset filter=lfs diff=lfs merge=lfs -text
*.umap   filter=lfs diff=lfs merge=lfs -text

*.fbx    filter=lfs diff=lfs merge=lfs -text
*.blend  filter=lfs diff=lfs merge=lfs -text

*.tif    filter=lfs diff=lfs merge=lfs -text
*.tiff   filter=lfs diff=lfs merge=lfs -text
*.png    filter=lfs diff=lfs merge=lfs -text

*.wav    filter=lfs diff=lfs merge=lfs -text
```

Adjust this list if repository storage becomes an issue.

---

## Collaboration rules

Unreal `.uasset` and `.umap` files are binary and do not merge cleanly like source code.

For that reason:

- avoid editing the same map or Blueprint at the same time
- communicate before making large changes to shared assets
- use feature branches
- keep commits focused
- do not upgrade Unreal Engine versions without agreement
- do not reorganize the entire Content folder without coordination
- do not overwrite raw GIS reference data
- keep imported reference assets separate from game-ready assets

Suggested branch naming:

```text
feature/roads
feature/pizzeria
feature/landscape-material
feature/building-cleanup
feature/player
fix/landscape-collision
```

---

## Known issues / unfinished areas

Current prototype issues include:

- Landscape collision still needs to be fully verified.
- Some GIS buildings sit slightly below the terrain.
- Some imported building surfaces may have inconsistent normals.
- Building separation from the raw GIS meshes is still a manual / semi-manual process.
- Road centerlines exist, but the final road system is not implemented.
- Materials and foliage are still largely placeholder.
- The pizzeria itself has not yet been built as a finished gameplay location.
- No final vehicle system is integrated into the Niederwangen gameplay map yet.
- No NPC, economy, delivery, crime or police systems are implemented yet.

Small differences between building bases and terrain are acceptable for blockout geometry. Important buildings should be corrected individually later rather than moving every building globally.

---

## Development philosophy

The project should prioritize a playable vertical slice over trying to finish the entire village at once.

A good near-term milestone is:

```text
one small part of Niederwangen
        +
finished terrain
        +
finished road
        +
vegetation
        +
several detailed buildings
        +
walkable / drivable gameplay
        +
working pizzeria prototype
```

Once that pipeline works, it can be expanded across the rest of the map.

---

## Credits / data

Game project: **Niederwangen**

Map data derived from official Swiss geodata.

```text
Geodata: © swisstopo
```

Additional asset, audio and software credits should be added here as the project grows.

---

## License

No project license has been defined yet.

Until a license is explicitly added, contributors should assume the source code and project assets are **not automatically licensed for redistribution or reuse outside this project**.

A proper project license should be chosen before the repository is made broadly public.

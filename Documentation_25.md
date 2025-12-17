# SUMO2UEBridge Documentation of Workflows
- Date: 29.09.2025

  - *Try to update this file with every progress you make*

## SUMO micro-scale model


<br><br>


## UE geodata-based model and visualizations - Yiru Yang
- Date: 24.11.2025, Week 1-3

### Data In Use

- weekday_sumo  - microscale_with_internals.sumocfg

```
(1) Geodata (Unreal, .uaaset)
        │
        ▼
[Blender + C++ plugin for Blender mesh generation] - Yiru Yang
        │
        ▼
(2) 3D meshes (.fbx, png, .glb -> .uasset) - Yiru Yang
        │
        ▼
[Import into Unreal Engine] - Zhanyi Wu
        │
        ▼
(3) Placed at correct coordinates (Z=0 base) - Zhanyi Wu
        │
        ▼
[Rendered with SUMO traffic data on top]
```

## Initial File Check - Unreal 5.6


<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic1.png" alt="Project 1 Visualization" width="75%">
</p>


## Initial Dataset Types - Blender 4.4

| Data Type                       | File Format                | Description                                     | Purpose in Blender                                            |
| ------------------------------- | -------------------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| **Digital Terrain Model (DTM)** | `.fbx`                     | 3D mesh representing elevation grid of Zurich   | Base ground geometry for your city model                      |
| **Digital Orthophoto (DOP)**    | `.png`                     | 2D satellite/aerial imagery texture             | Color/texture overlay on top of terrain                       |
| **Material File (PBR)**         | `.glb`                     | glTF binary including UV, shaders, lighting     | Recreate lighting/texture nodes in Blender                    |
| **3D Buildings Mesh**           | `.fbx`                     | Simplified or detailed extruded building models | Used for city block visualization or modeling base geometry   |
| **Cesium World Terrain**        | 3D Tileset (`.json/.b3dm`) | Streamed dynamic terrain data                   | Not directly exported; use “Bake to Static Mesh” to convert   |
| **CesiumGeoreference**          | Internal                   | Coordinate anchor                               | Ensures correct geospatial alignment when exporting/importing |


## 📍 Check Your World Coordinates

By default, BlenderGIS only provides two CRS options
  - Web Mercator (EPSG:3857)
  - WGS84 latlon (EPSG:4326)

For Zurich, you need to use the local coordinate system EPSG:2056 (CH1903+ / LV95), add it manually

  - EPSG Reference Website - https://epsg.io/2056

```
EPSG:2056
Name: CH1903+ / LV95
Area: Switzerland
Unit: meter
```

 - Copy the following definition line from the EPSG page

```
+proj=somerc +lat_0=46.95240555555556 +lon_0=7.439583333333333 +k_0=1 +x_0=2600000 +y_0=1200000 +ellps=bessel +towgs84=674.374,15.056,405.346,0,0,0,0 +units=m +no_defs +type=crs
```


Add the CRS in BlenderGIS
  - Return to BlenderGIS → click “Add CRS”
  - Name: EPSG:2056 (Zurich)
  - Definition: (paste the copied line above)
Click Save

Select the Newly Added CRS
```
EPSG:2056 (Zurich)
```

- Final Import Settings

| **Parameter**            | **Value**  |
| ------------------------ | ---------- |
| **Elevation source**     | Geometry   |
| **CRS**                  | EPSG:2056  |
| **Extrusion from field** | Disabled   |
| **Separate objects**     | Disabled   |


## 📍 Why Important

  - This definition is used to correctly project the Swiss LV95 (new national coordinate system) onto WGS84 (latitude and longitude) or WebMercator (online map)
  - Our Zurich data (tree shapefiles, buildings, terrain DEMs) are all created in LV95 coordinates
  - Without the above, he coordinates will be shifted by hundreds of kilometers after importing into Blender (e.g., to France)



## 🌳 TreeGeneration Dataset Overview

| **File**      | **Purpose in the Pipeline**           | **Description**                                                                                                                                                                                                          |
| ------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **TREE.obj**  | **3D Model Geometry**                 | Contains the **3D mesh of a single tree** — this serves as the base geometry that will be **instanced** (duplicated) across all geographic tree positions imported from the shapefile.                                   |
| **TREE.mtl**  | **Material Definition**               | The **material library** associated with `TREE.obj`. It defines surface properties such as texture color, reflectivity, and shading for the tree model when imported into **Blender** or **Unreal Engine**.              |
| **TREES.shp** | **Spatial Point Layer (GIS)**         | A **shapefile containing point coordinates** — each point represents the **location of one tree** in Zurich’s coordinate system. Used by **BlenderGIS** or **QGIS** to position tree instances correctly on the terrain. |
| **TREES.dbf** | **Attribute Table**                   | The **attribute database** linked to `TREES.shp`. It may store optional metadata such as **tree type, height, or radius**. Each record corresponds to one tree point.                                                    |
| **TREES.prj** | **Coordinate Reference System (CRS)** | Defines the **spatial projection and coordinate system** used in the shapefile — here, it specifies **EPSG:2056 (CH1903+ / LV95)**, which ensures correct alignment with Zurich’s real-world geospatial coordinates.     |
| **TREES.shx** | **Index File**                        | A spatial **index file** that accelerates access to the shapefile geometry. It allows BlenderGIS or GIS software to **read tree locations efficiently**.                                                                 |
| **TREES.cpg** | **Character Encoding Specification**  | Describes the **text encoding (e.g., UTF-8 or Latin1)** of the `.dbf` attribute table. This ensures that attribute names and non-ASCII characters are read correctly when importing into **BlenderGIS or Python**.       |


## 🌳 Trees parameters
| **Parameter**                         | **Value**     | **Description (Data Source)**                                                                                    |
| ------------------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------- |
| **CRS (Coordinate Reference System)** | **EPSG:2056** | Zurich coordinate system (CH1903+ / LV95), used for precise geospatial alignment of tree locations.              |
| **Z Elevation**                       | **0**         | Ground elevation level; all tree points are placed on base terrain (Z = 0).                                      |
| **Import as**                         | **Points**    | Each point corresponds to a tree location from `TREES.shp` (GIS point dataset).                                  |
| **Scale**                             | **1.0**       | 1 Blender unit equals 1 meter — ensures consistent scaling with Unreal Engine.                                   |
| **Tree Model Source**                 | **TREE.obj**  | Each tree’s 3D geometry is imported from `TREE.obj`, representing the individual tree model used for instancing. |


```
(TREES.shp + TREES.dbf + TREES.prj)
      ↓
[Polygon or Tree Location Data in EPSG:2056]
      ↓
Converted to centroid points
      ↓
Each point → Instance of TREE.obj
```

  - Extract the center point of each tree canopy from the shapefile, and then use `TREE.obj` to instantiate the entire forest in batches at these points

## Extract the Below data from Unreal -> Blender usage - Zurich

```
┌──────────────────────────────────────────────────────┐
│        Unreal Engine 5.6                             │
│ ───────────────────────────────────────────────────  │
│  Cesium World Terrain  → Base Mesh (ZH_DTM_BASEMAP)  │
│  ZH_DTM_DOP          → Texture Layer (DOP)           │  
│  ZH_BUILDINGS        → Building Mesh (.fbx)          │
│  ZH_TREES            → Tree Mesh (.fbx)              │
│  CesiumGeoreference  → Coordinate System             │
│  CesiumSunSky        → Lighting & Sun                │
└──────────────────────────────────────────────────────┘
                ↓
┌───────────────────────────────────┐
│        Blender 4.4                │
│ ────────────────────────────────  │
│  GE_DGM.fbx   → Terrain mesh      │
│  GE_DOP.png   → Orthophoto texture│
│  rastMat.glb  → PBR material      │
│  Buildings.fbx → Building geometry│
└───────────────────────────────────┘
```

## Blender - .fbx, .png, .glb

| **File Name**     | **Type** | **Purpose**                                    |
| ----------------- | -------- | ---------------------------------------------- |
| **GE_DGM.fbx**    | Mesh     | Zurich terrain geometry (Digital Ground Model) |
| **GE_DOP.png**    | Texture  | Orthophoto imagery for surface texturing       |
| **rastMat.glb**   | Material | Material node reference (PBR / UV mapping)     |
| **Buildings.fbx** | Mesh     | 3D building models for architectural modeling  |



- `GE_DGM.fbx`: Kaydara FBX model, version 7.3 (binary, 2013 standard) — contains the digital terrain mesh geometry for the Zurich area.
- `GE_DOP.png`: PNG raster image, 2531 × 3022 pixels, 8-bit RGBA color, non-interlaced — represents the orthophoto (aerial imagery texture) aligned with the terrain model.
- `rastMat.glb`: glTF 2.0 binary model, 11,188,776 bytes — includes physically-based material nodes (PBR), UV coordinates, and lighting information for terrain rendering.


| **Property**        | **Value**                                                                                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Files**           | `GE_DGM.fbx`, `GE_DOP.png`, `rastMat.glb`                                                                                                                            |
| **Repository Path** | [github.com/egelerj/SUMO2UEBridge/tree/main/Blender_Zurich](https://github.com/egelerj/SUMO2UEBridge/tree/main/Blender_Zurich)                                       |
| **Formats**         | `FBX 2013 (binary v7.3)` — 3D terrain geometry; `PNG RGBA 8-bit` — orthophoto texture; `GLB 2.0 (binary)` — baked PBR material                                       |
| **Validated via**   | `file *` → confirms FBX 7.3 binary / PNG RGBA image / glTF 2.0 binary structure                                                                                      |
| **Includes**        | Terrain geometry (DGM), UV coordinate mapping, baked orthophoto texture (`GE_DOP`), and corresponding material reference (`rastMat`)                                 |
| **Purpose**         | Serves as the **core shared dataset** enabling seamless **Blender ↔ Unreal Engine** interoperability for geospatial 3D terrain visualization, editing, and analysis. |


## 📏 Blender Pipeline - Scale = 0.001 from Unreal

<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic2.png" alt="Project 1 Visualization" width="75%">
</p>

- Get started

```
Shift + C
Fn + . 
```

- rastMat

```
Shift + A
Texture → Image Texture
```

- 3D Buildings - Manual Input Way (Optional, we use .py script to auto generate)

```
(Option A)
🧲 Snap To: Face  
Target: Closest  
Exclude Non-Selectable  
Face Nearest ✅  
Face Project ❌  
Align Rotation ❌  
Backface Culling ❌  
Affect: Move  

(Option B)
Shortcut: G → Z → Ctrl
```

- Avoid Lakes etc.

| Parameters     | Value                |
| -------------- | -------------------- |
| Terrain Object | `GE_DGM`             |
| Dimensions     | X ≈ 505 m, Y ≈ 603 m |
| Z Range        | ~0 – 63 m            |
| Material       | `Material.001`       |
| Metric Scale   | ✅                   |
| Lake Exclusion | Z > 1 m ✅           |


- Wrong Sample of Builing Cubes - 80 buildings

<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic3.png" alt="Project 1 Visualization" width="75%">
</p>

- Correct Sample of Builing Cubes - 10,000 and 40,000 buildings (*estimated 40,000 from Geneva Gov, ~34,000 from Zürich Gov)

<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic4.png" alt="Project 1 Visualization" width="75%">
</p>

<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic5.png" alt="Project 1 Visualization" width="75%">
</p>


## 3D meshes into Unreal - Initial Check - Zurich

- Initial check with Zurich.blend file, pending trees + 3D buildings into Unreal


<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic6.png" alt="Project 1 Visualization" width="75%">
</p>

<br>

## TODO

- Zurich Coordinates aligned assets - 3D Reconstruction Outcome Directly to UE 5.6 → Edit → Plugins - “Point Cloud” , 24-Nov
  - datasets in use - https://polybox.ethz.ch/index.php/s/L8oqRkpSzLXaLTe?path=%2FZUERICH
  - Align with - https://github.com/egelerj/SUMO2UEBridge/tree/main/sumo_net_reprojection
  - the most recent Zurich project version - https://drive.google.com/drive/folders/1RGVTEkG2cInp1ZSMjcY5W3-ATVm4Uffv
  - coordinate check, Digital Zurich 3D Assets Surface Refine for Unreal Render

- Based on 'ZH_MLS.uasset', refine the point clouds rendering surface


## Why Gaussian Splatts Can't Generate Missing Attributes on the current Point Clouds for You

**Gaussian Splatting vs. Diffusion Completion**

- 3D Gaussian Splatting (3DGS) is an explicit surface representation, not a generative model. Each Gaussian primitive stores only the information it learns from data:

  - Position μ (x, y, z)
  - Covariance Σ (shape / orientation)
  - Color c (RGB)
  - Opacity α

- 3DGS can render and fit existing geometry, but it cannot predict missing attributes such as color, normals, or unseen surfaces. If the input point cloud lacks certain attributes, the corresponding Gaussians will simply not contain that information.

📍 To handle incomplete data, Option A - use Generative Models such as Diffusions

| Goal                              | Recommended Step                                                         |
| --------------------------------- | ------------------------------------------------------------------------ |
| Missing geometry / attributes     | → Use **Diffusion → Gaussian** (diffusion completion before 3DGS)        |
| Surface refinement after training | → Use **Gaussian → Diffusion** (denoise or enhance Gaussians after 3DGS) |


📍 Option B - Extending Gaussian Splatting

- e.g., The neighborhood feature propagation method predicts/interpolates missing RGB and geometry, and can be run directly on Gaussian parameters or point clouds

```
def neighborhood_completion(positions, features, k=8):

    # Identify valid (non-NaN) feature entries
    mask_valid = ~torch.isnan(features).any(dim=1)
    if mask_valid.all():
        return features  # nothing to fill

    # Fit nearest neighbor search on valid points
    from sklearn.neighbors import NearestNeighbors
    nbrs = NearestNeighbors(n_neighbors=k).fit(positions[mask_valid])

    # Find neighbors for invalid points
    _, idx = nbrs.kneighbors(positions[~mask_valid])

    # Compute averaged neighbor features
    filled = torch.stack([
        features[mask_valid][inds].mean(dim=0)
        for inds in idx
    ])

    # Replace missing rows
    features[~mask_valid] = filled
    return features
```

**Steps**

- Write a small inspection script to check which attributes exist in your point cloud data (position, color, normals, intensity, etc.). Quantify missing attributes or NaNs — this tells you if you’re missing:
  - geometry (sparse regions)
  - appearance (RGB)
  - surface info (normals / semantics)

- Then decide:

| Stage                    | Input Format                        | Output Format                  | Goal / Effect                                         |
| ------------------------ | ----------------------------------- | ------------------------------ | ----------------------------------------------------- |
| **Diffusion → Gaussian** | `.ply`, `.pcd` → `.ply_completed`   | `.bin` (Gaussians)             | Fill missing data before 3DGS → better initialization |
| **Gaussian → Diffusion** | `.bin` / `.ply` (trained Gaussians) | `.ply_refined`, `.fbx_refined` | Denoise / smooth surfaces → photorealistic rendering  |


## Step 1: Export the Unreal .uasset  
1. Open your Unreal Project (e.g., `UE_ZUERI_cpp.uproject`).  
2. In the Content Browser locate the asset, e.g. `ZH_MLS.uasset`.  
3. Right-click → **Asset Actions → Export**.  
4. Choose a format (e.g., `.fbx` for mesh or `.ply` for point cloud) and export to `/exports/`.  
   ```bash
   /exports/ZH_MLS.fbx  # or ZH_MLS.ply

## Step 2: Geometry Refinement with Diffusion

```
git clone https://xxxx[Diffusion_refinement_Model].git
cd xxxx[Diffusion_refinement_Model]
conda create -n diffusiongs python=3.11
conda activate diffusiongs
pip install -r requirements.txt
```

- Pre-process the exported geometry (mesh or point cloud) as required in the above repository
- Run the refinement for Zurich 3D Assets
📍
```
python run.py --input /exports/ZH_MLS.fbx --output /exports/ZH_MLS_refined.ply
```


- The result /exports/ZH_MLS_refined.ply is a refined point cloud/mesh ready for re-import into Unreal


| **Stage**                                    | **Input File Type**       | **Description**                         | **Your Current Status**        |
| -------------------------------------------- | ------------------------- | --------------------------------------- | -----------------------------  |
| **Raw Scanning**                         | `.las` / `.pcd` / `.ply`  | Original LiDAR point cloud                  | ❌ Not present in your dataset |
| **Point Cloud Registration (Alignment)** | `.ply` / `.pcd`           | Multi-view registered point cloud           | ❌ Missing                     |
| **Reconstruction Model**                 | `.mesh` / `.fbx` / `.obj` | Mesh reconstructed from the point cloud     | ❌ Not yet generated           |
| **Unreal Engine Asset**                  | `.uasset`                 | Unreal Engine asset file                    | ✅ You have `ZH_MLS.uasset`    |



<br><br><br><br><br><br><br><br>


## 💻 Render - Unreal - Zhanyi Wu



<br><br><br><br><br><br><br><br>


## C++ libtraci bridge


<br><br><br><br><br><br><br><br>


## OSC Plugin, Output Stream


- Have A Try on-device


<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic7.png" alt="Project 1 Visualization" width="75%">
</p>

<p align="left">
  <img src="https://github.com/egelerj/SUMO2UEBridge/blob/main/assets/pic8.png" alt="Project 1 Visualization" width="75%">
</p>


<br><br><br><br><br><br><br><br>




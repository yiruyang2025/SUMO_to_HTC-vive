# SUMO2UEBridge

- Supervisors -  Jonas Egeler, Dr. Ulrike Wissen, MR Team
- Bridging SUMO (Simulation of Urban Mobility) with Unreal Engine 5.6 via C++


## Resources
- IVT: https://www.ivt.ethz.ch/ (optional)
- SUMO: Traffic simulation backend, allowing for micro-scale traffic flow simulation
- CARLA: Open-source autonomous driving simulator with SUMO co-simulation feature
- C++ TraCI API: Documentation of C++ TraCI API client library
- [2017 - Microsoft - AirSim](https://github.com/microsoft/AirSim)
  - Open source simulator for autonomous vehicles built on Unreal Engine / Unity, from Microsoft AI & Research


## Softwares
| Module                  | Description                                                                                                    | Language                  |
|------------------------|----------------------------------------------------------------------------------------------------------------|---------------------------|
| SUMO                   | Microscopic traffic simulator that generates vehicles, routes, speeds, and traffic light states.               | C++             |
| TraCI Client (Bridge)  | Uses the TraCI (Traffic Control Interface) API to fetch real-time data from SUMO and send it to Unreal Engine. | C++              |
| Unreal Engine Plugin   | Receives SUMO data inside UE5, spawns vehicle actors, and updates their positions and orientations.            | C++ (Blueprint interface) |

## Plugins for Zurich World Coordinates
  - In Blender (during shapefile import) - use `epsg.io`
    - → find and set the correct coordinate reference system (EPSG:2056)
    - → https://github.com/domlysz/BlenderGIS, for trees, file form in Shapefile (.shp), GeoTIFF
  - In Unreal Engine - use `Cesium Georeference`
    - place and align the model at the correct real-world geographic location
    - CesiumForUnreal，https://cesium.com/platform/cesium-for-unreal/ from Epic Games Launcher Plugin
    - Or directly from https://github.com/CesiumGS/cesium-unreal?utm_source=chatgpt.com and place locally into /Users/xxx/Desktop/UE_5.6/Engine/Plugins/Marketplace/


## Repository

```
SUMO2UEBridge/
├── SUMO2CPP/              # C++ bridge: SUMO ↔ Unreal
│   ├── CMakeLists.txt      
│   ├── src/              
│   └── include/          
│
├── GE_GIS_cpp/            # Unreal visualization
│   ├── Source/             
│   ├── Content/            
│   └── Config/           
│
├── exports_Geneva/        # (No need to use, we switched to Zurich)
│   ├── Geneva.blend
│   ├── buildings_Geneva.py
│   ├── GE_DGM.fbx      
│   ├── GE_DOP.png      
│   └── rastMat.glb
│    
├── Blender_Zurich/        # Workspace for editing any 3D Asset
│   ├── Zurich.blend             
│   └── build_zurich.py    # automation script for Blender
│
├── Unreal_Zurich/         # Final 3D Zurich -> directly to Unreal
│   ├── 3D Point Clouds Reconstruction -> .usset  # aligned using `sumo_net_reprojection`
│   └── Trees.fbx               
│
├── requirements.txt (Optional)       
├── LICENSE
└── README.md
```

**Attached Local Files**

```
MR-Data/
└── ZUERICH/
    ├── base_scenario/
    │   ├── Network/
    │   ├── OD_Development_Morning/
    │   ├── OD_Development_Evening/
    │   ├── Simulation_Morning/
    │   ├── Simulation_Evening/
    │   └── vtype/
    ├── UE_ZUERI_cpp/
    │   ├── Config/
    │   ├── Content/
    │   ├── Source/
    │   └── UE_ZUERI_cpp.uproject
    ├── BaseScenario60sCycleMainRoadsOnly.net.xml
    ├── ZH_MLS.uasset
    ├── README.md
    └── .DS_Store
```

- `Zurich.blend`: Blender3D, saved as 64-bits little endian with version 4.04


## Project Structure
```
L1. Simulation Layer —— SUMO
       ↓ TraCI API
L2. Visualization Layer —— Input into Unreal Engine - Zhanyi Wu
       ↓ OSC Plugin
L3. Auralization Layer —— Audio Engine / GPU Audio SDK
```

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
[Import into Unreal Engine 5.6] - Zhanyi Wu
        │
        ▼
(3) Placed at correct coordinates (Z=0 base) - Zhanyi Wu
        │
        ▼
[Rendered with SUMO traffic data on top]
```



## Quick Start

1. Clone repository
```
cd ~/Desktop
git clone https://github.com/egelerj/SUMO2UEBridge.git
cd SUMO2UEBridge
```

2. Import .blend / .json / .uasset into Unreal
```
# In Unreal Editor:
    Import "Unreal_Zurich/" into Content/Geodata/
# Unreal will automatically convert them to .uasset
```

3. SUMO2UE - C++ bridge
```
cd SUMO2CPP
mkdir build && cd build
cmake ..
make -j8
./build/sumo2uebridge
```

## Unreal Engine to HTC Vive Deployment Guide

## 1. SDK
- https://developer.vive.com/resources/downloads/
- https://developer.vive.com/eu/?utm_source=chatgpt.com
- https://www.vive.com/us/setup/vive-pro-hmd/


## 2. Project Structure
```
/UnrealProjects/
    /YourProject/
        ├── Config/
        ├── Content/
        ├── Plugins/
        ├── Saved/
        ├── Source/
        └── Build/
```

## 3. Asset Optimization and Size Targets
| Category      | Recommended Limit     | Compression                    |
| ------------- | --------------------- | ------------------------------ |
| Texture       | ≤ 4K (≤ 8 MB each)    | BC7 or ASTC                    |
| Mesh          | ≤ 200k triangles      | Nanite / LOD                   |
| Point Cloud   | ≤ 2M points per chunk | Downsample via Open3D          |
| Audio         | ≤ 48 kHz stereo OGG   | Built-in compression           |
| Total Package | ≤ 10 GB               | Split into Pak files if needed |


## 4. Packaging Procedure From Local PC
- Open Unreal Editor
- File → Package Project → Windows (64-bit)
- Packaging Settings:
  - Configuration: Shipping
  - Use Pak File: True
  - Full Rebuild: True
  - Include Crash Reporter: False (optional)
- Output directory
- Test the packaged .exe with SteamVR


## 5. Testing on HTC Vive
- Connect headset:
  - DisplayPort → GPU
  - USB 3.0 → PC

- Start SteamVR and VIVE Console
  - `VivePort` for Viveport Setup
  - Variable FPS (30–120 Hz, ideally 90 / xx Hz for VR)
- Verify base stations and controllers are tracked
- Launch your packaged build
- If VR does not start automatically, set: SteamVR → Settings → Developer → Set SteamVR as OpenXR Runtime


## 6. Deployment
- Option A: Manual
  - Copy the packaged folder to the target PC
  - Create a shortcut to the .exe
  - Maintain versioned folders (e.g., Build_1.0.0)


- Option B: Viveport Submission
  - Create a Vive Developer account
  - Upload build on Viveport Developer Console
  - Include metadata, screenshots, and icon (512×512 px)
  - Recommended package size: ≤ 10 GB
  - Submit for review



## 7. Verification Checklist
| Check                           | Tool                          |
| ------------------------------- | ----------------------------- |
| Frame Rate ≥ 90 FPS             | SteamVR Performance Overlay   |
| No shader stalls                | UE Stat GPU                   |
| Correct scale (1 m = 1 UE unit) | In-scene test                 |
| Spatial audio working           | Steam Audio / Resonance Audio |
| Memory usage < 10 GB            | Resource Monitor              |
| Package size logged             | Saved/Logs/Packaging.log      |


## 8. A New Layer For User Control with Handles
- Trackpad
  - The Trackpad provides a 2D axis input suitable for:
  - Moving forward and backward
  - Strafing left and right
  - Rotating the view
  - Teleporting when pressed
  - Navigating or selecting map layer directions
Trackpad is the primary source of player locomotion.

- Trigger
  - The Trigger button is used for player actions such as jump.


## Troubleshooting

- If buildings or roads appear misaligned, check the coordinate scale (1 m = 100 cm in Unreal) and adjust origin offsets in the Blender export script.
- SUMO acts as the master clock (fixed 10–100 Hz); Unreal runs as the slave clock (variable FPS).
If the visual simulation drifts, verify that the shared buffer is reading the latest SUMO state instead of older frames.

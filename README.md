# SUMO to UE direct bridge (without python) 
Uses UE5.6. [Here's a google drive link to all project zips. Choose most recent one.](https://drive.google.com/drive/folders/1RGVTEkG2cInp1ZSMjcY5W3-ATVm4Uffv?usp=drive_link)
## Instructions for integrating the code into new project
1. The following instructions assume the UE project is named "sumo_bridge_56". When using other names, remember to change all occurences of "SUMO_BRIDGE_56_API" in the code to "YOURNAME_API" later.
2. Navigate to ``sumo_bridge_56/Source/sumo_bridge_56``. 
3. Copy over the entire contents of the ``code`` folder from this repo (i.e. the c++ files and the ``ThirdParty`` subfolder).
4. In the same folder, find ``sumo_bridge_56.Build.cs`` and open it.
   - Add ``"Sockets"``, ``"CesiumRuntime"``, ``"XmlParser"``, ``"OSC"`` to the arguments in the function call ``PublicDependencyModuleNames.AddRange(new string[] {...});``, if it is not there.
   - Add the following function:
   ```
     	private void LoadTraCI(ReadOnlyTargetRules Target) {
          // Get the base path of the ThirdParty directory
          string ThirdPartyPath = Path.GetFullPath(Path.Combine(ModuleDirectory, "ThirdParty"));
          string TraCIPath = Path.Combine(ThirdPartyPath, "TraCI");
  
          // Add the include path for the header files
          PublicIncludePaths.Add(TraCIPath);    
      }
   ```
   to the class as a member function, and call it at the end of ``public sumo_bridge_56(ReadOnlyTargetRules Target) : base(Target) {...}`` with ``Target`` as argument. 
5. Now the project needs to be rebuilt. Close UE and Visual Studio. Go to project root folder ``sumo_bridge_56``. Right click on ``sumo_bridge_56.uproject``, select ``Generate Viual Studio project files``. Open ``sumo_bridge_56.sln`` and build from Visual Studio (usually you can press F5) once. After this, you can go back to building directly from UE editor as usual.
6. Open UE editor. Make an instance of the ``ActorPool``. In the details panel, look at the "Pooling" options.
    - Create the same number of array elements under "Pool Configurations" as the number of vehicle types. 
    - For each vehicle type, attach a blueprint that derives from the ``PooledObject`` as well as meshes to the corresponding array element, and fill out the fields.
7. Make an instance of the DirectSumoBridgeActor. In the details panel, look at the "SUMO Connection" and "SUMO Visualization" options.
    - "Server List" should contain info about the SUMO servers you want to switch between. 
    - "Sumo Step Length" must be the same as the ``step-length`` in the SUMO config file you're using for the server, usually 0.01 for 100Hz.
    - "Vehicle Pool" should be the ``ActorPool`` instance from the last step.
    - "Spawn Radius" is the radius within which vehicles will be rendered around the player. Set to 0 for infinite radius.
    - "Visbility Check Interval" controls how often it is checked whether a vehicle is within spawn radius.
    - "Sumo Offset X/Y" should be calculated as UE project origin + the offset from the (converted) Sumo network file.
    - "Traffic Light Actor Class" should be a blueprint that derives from ``TrafficLightActor``.  
    - "SM Raycast Targets" should include the meshes for the ground & buildings. These meshes should have nanites disallowed.
8. To start a SUMO server, open a terminal and run ``sumo -c "my_sumo_config.sumocfg" --remote-port 4042`` in the folder with the sumocfg files. See below for using checkpoints. The network file needs to be converted with ``reproj.py`` from the main branch. Make sure the started servers correspond to the info you provided in the UE editor, then run the game.
9. Thats it unless I forgot something

## If UE crashes for no reason
Perform step 5 from above. If that doesn't work, delete the folders ``Binaries``, ``Intermediate``, ``Saved``, ``.vs``, and try again.

## SUMO Commands for creating/loading simulation checkpoints
Creating:
```
sumo -c "my_sumo_config.sumocfg" --save-state.times 22000,22500 --save-state.files ckpt_22000.xml,ckpt_22500.xml
```

Loading:
```
sumo -c "my_sumo_config.sumocfg" --remote-port 4044 --load-state ckpt_22000.xml --begin 22000
```
When loading, remember to use the same sumocfg file and set the begin time to the checkpoint's time.

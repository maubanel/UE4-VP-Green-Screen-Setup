# UE4-VP-Green-Screen-Setup
1.  Equipment
      * Unreal Engine 4.26
      * RED Scarlet Camera with 23.976 ouptut to through the BNC
      * Aja Sync going to Camera and Black Magic Card
      * Black Magic Ultra Studio 4K Mini with Thunderbolt
      * First Generation Vive Lighthouses with Second Generation Vive Tracker
2.  Connect camera and black magic a valid sync signal
3.  Create a new blank film project and give it an appropriate name.  This will give you the plugins you need.
4. [Black Magic Setup Per Unreal Documentation](https://docs.unrealengine.com/4.26/en-US/WorkingWithMedia/ProVideoIO/BlackmagicQuickStart/)
     * Add a **Media  | Media Bundle** to a folder called `MB_LiveCamera`.
     * Open media bundle and assigned **Media Source** a `Black Magic Media Source` and `1080 Progressive 23.976`.  
     * Set timecode to `LTC`.
     * Set **Reopen Source on Error** to `true` as the black magic does lose track of hte camera
     * Turn on the RED Camera
     * On the RED camera press **Medu | Settings | Display | Monitor Control** change **LCD** to `HDSDI`.  This sends the output of the camera through the BNC cable
     * On the **Media Bundle**, select **Request Play Media**.

5. [Time Code and Genlock Setup Per Unreal Documtation](https://docs.unrealengine.com/4.26/en-US/WorkingWithMedia/ProVideoIO/TimecodeGenlock/)
     * Make sure you have **Windows | Developer Tools | Timecode Privder** selected and mount in top right
     * Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicTimecodeProvider**.  Call it `BP_BlackMagicTCProvider`. Select the corresponding black magic card in **Media Configuration** as well as **LTC**timecode.
     * Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicCustomTimestep**.  Call it `BP_BlackMagicGenlock`.  Select the corresponding black magic card in **Media Configuration**.
     * Go to **Edit | Project Settings | Engine | General Settings | Timecode Provider** and select **BP_BlackMagicTCProvider**.
     * Go to **Edit | Project Settings | Engine | General Settings | Custom Timestep** and select **BP_BlackMagicGenlock**.
     * Close editor and restart the project
6. [Setup Vive using Live Link UE4 Documentation](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/LiveLinkPlugin/Livelinkxr/)
     * Connect Vive to PC
     * Calibrate vive track with Vive VR
     * Add Live Link XR Plugin: **Edit | PLugins | Live Link XR**.
     * Add **Windows | Live Link** tool.
     * Make sure Steam VR is running and turn on tracker on camera
     * Press **+ Source** and make sure **Track Trackers** is selected and others are not and set the **Loca Update Rate** to `90`.
     * Make sure it adds a Steam VR tracker to the **Subject Name** window.  If not restart unreal with steam trackers on.
     * Press the <kbd>Add</kbd> button.
7. [Create a Blueprint to Make Camera Follow Tracker with Smoothing](https://www.youtube.com/watch?v=jx8cxoW5vnc&t=96s)
     * Add new **Actor Blueprint** called `BP_Camera`.
     * Add a **Live Link Controller** component.
     * Select **Subject Representation** and select the Steam VR tracker.
     * Underneath **Role Controller | Live Link** untick **World Transform**.
     * Add two **Arrow Components**.  Call one **Tracker** and the other **Camera**. Put Camera under tracker.  Change color of camera arrow to blue.
     * Add an offset for the camera arrow.  The **Z** offset on our rig was `-33.02cm`.  The **X** offset at **-2.54cm**. This is based on how far the center of the tracker is to teh center of the sensor on the film camera
     * Go to the **Event Graph** and a **Live Link | On Live LInk Update +**.  This adds a live link update in the blueprint.
     * Add a **Variable** called **Smoothing** and as a **Float**. Make it public. Make the default `15.0`.
     * Add a **Variable** called **CineCamera** of type **Cine Camera Actor | Object Reference**. Make it public.
     * Add a **Variable** called **Last Transform** of type **Transform**. Make it private.
     * Drag a **Camera** to Blueprint and add a **Get World Transform** node.
     * Drag a **Last Transform** to the graph and add a **LERP** node.
     * Plug smoothing amount into **Multiply** node and multiply by **Delta Time** then plug into the **Alpha** of LERP and output of **Get World** to **B** side of lerp.
     * Grab the **Cine Camera** and add a **Set Actor Location and Rotation**. Add execution pin from Live Link update to Set Location and Rotation.  Connect the output of the LERP to the input of the location and rotation.
     *  Reset the new **Last Transform** by setting it to the newest value.
8. Add Camera and Blueprint to game to control virutal camera with RED cam.
     *  Drag **BP_Camera** in game.  Add a **Cine Camera** to the scene and assign it to the **BP_Camera** blueprint.
     *  Adjust **CineCamera Actor | Current Camera Settings | Filmback | Sensor Width** to `27.7 mm` and **Sensor Height** to `14.6 mm`.  Also set the aperture and focal length manually. 
     *  Add empty actor above **BP_Camera** so you can readjust 0,0,0 in world and call it **Camera Origin**.
     *  Add a **Take Recorder** and add **Cin Camera** to the recorder.
     *  Add a plugin called **Edit | Plugins | Time Data Monitor** to the project and reboot Unreal.
     *  Select **Window | Developer | Time Data Monitor**.  Add a time correction (in our case it was `.0125`.
9. Create new Composure Composite
     *  Add a new **Create new Comp** in **Composure Compositing** and name it.
     *  Add a **Layer Element** of **Media** type and call it **Live Camera**.
     *  In Live Camera Input Meida Source and add the Media Texture of the camera. 
     *  Add keys to chroma out the background.
     *  Add comp layer for **Add New Layer Element** a **CG Layer** called `Background Element`.
     *  Create a **Materials** folder.  Add a new **Material** called `M_Composite`.
     *  Go to **Material Domain** and change it to `Post Process`.
     *  Right click and two **Texture Sample Parameter 2D**.  Call them the same as the elements `LiveCamera` and `BackgroundElement`.
     *  Add an **Over** node and plub int the **RGBA** pins to it with the background on the bottom. Connect the output of the **Over** to the **Emissive Color** pin.
     *  Press **Apply**. Assign the materail to an added transform compositing pass.

10. Garbage Matte Setup
     * Create a new Actor Blueprint called `BP_GreenScreen`.
     * Add plane call it `Wall`.
     * Place the blueprint in the level
     * Rotate it to be perpendicular to the camera and in front of it
     * Add a **CG Matte** layer to composure and call it `GarbageMatte`.
     * Open up **M_Composite** and add a **exture Sample Parameter 2D** and call it `GarbageMatte`.
     * Plug int the RGBA output into the Matte input in the **Over** node.
     * Create a new layer and add actors to new layer and call it `GarbageMatteLayer`.
     * Go to **Garbage Matte** in composure and add a **Capture Actors | Actor Set** and select the `GarbageMatte Layer`.
     * Connect the camera source to override and select **Target Actor Camera**.
     * Open up **Background Element**.  Open up **Capture Actors**, select **Exclude** and select **Garbage Matte Layer**.
     * Scale garbage matte to fit space
     * Add floor plane to blueprint and adjust size to match the blue/green screen floor
     * Get scale correct in game, then copy values into the blueprint so it can be reused in other maps/levels.
     * 

11.  Output to external Black Magic
     * Connect the output of the **Black Magic** capture card used with Unreal to another **Black Magic** capture card on a second computer.
     * Add a **Media | Blackmagic Media Output** to the **Black Magic** folder.
     * Double click and selectg the black magic card that is used for unreal at the proper framerate, resolution and timecode.
     * Open up the composite and change the **Output** to `Compositing Media Capture Output`.  Press **+** to add output if needed.
     * Select the **Capture Output** to **Black Magic Media Output**.
     * You will need to convert the image from Linear Color Space to SRGB.
     * Got to [Open Color IO](https://opencolorio.org) and download the **Sample OCIO Configurations** zip file.
     * Go to the composite **Output | Color Conversions**.  Lets change it to `Compositing Open Color IOPass`.
     * Open up **Color Conversion Seettings** and create a new **OpenIO Color Configuration** file and call it `LinearToSRGB`
     * Then press save.  Double click and browse to the downloaded folder and open the **config.ocio** file.
     * Add two **Desired Color FSpaces**.  First is **Linear** and second is **sRGB**.
     * Change **Source Color Space** to **Linear** and change **Destination Color Space** to **sRGB**.

### Topology
```
                                ┌────────────────────┐
                                │                    │
                                │                    │
                                │                    │
                                │       
                                
                                
                                Scarlett │
                                │                    │
                                │                    │
                                │                    │
                                │                    │
                                │                    │
┌──────────────────┐            └─▲───────┬──────────┘
│                  │              │       │
│                  │              │       │
│                  ├──────────────┘       │
│                  │                      │
│ Aja Sync.        │                      │
│                  │                      │
│                  ├──────────────────┐   │
│                  │                  │   │
│                  │                  │   │
└──────────────────┘               ┌──▼───▼─┐
                                   │        │
                                   │ Black Magic
                                   └─────┬───
                                         │
                                  ┌──────▼───────────────┐
                                  │                      │
                                  │                      │
                                  │                      │
                                  │                      │
                                  │   Computer           │
                                  │                      │
                                  │                      │
                                  │                      │
                                  └──────────────────────┘
```

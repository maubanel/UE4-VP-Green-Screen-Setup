# UE4-VP-Green-Screen-Setup
1.  Equipment
      * Unreal Engine 4.26
      * RED Scarlet Camera with 23.976 ouptut to through the BNC
      * Notch for keying
      * Aja Sync going to Camera and Black Magic Card
      * Black Magic Ultra Studio 4K Mini with Thunderbolt
      * First Generation Vive Lighthouses with Second Generation Vive Tracker
      * One Unreal unit is in cm
2.  Connect camera and black magic a valid sync signal
3.  Make sure Notch is runnign on the green screen TV
4.  Create a new blank film project and give it an appropriate name.  This will give you the plugins you need.
5. TBD Sync the video card in Unreal with sync

6. [Time Code and Genlock Setup Per Unreal Documtation](https://docs.unrealengine.com/4.26/en-US/WorkingWithMedia/ProVideoIO/TimecodeGenlock/)
     * Make sure you have **Windows | Developer Tools | Timecode Privder** selected and mount in top right
     * Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicTimecodeProvider**.  Call it `BP_BlackMagicTCProvider`. Select the corresponding black magic card in **Media Configuration** as well as **LTC**timecode.
     * Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicCustomTimestep**.  Call it `BP_BlackMagicGenlock`.  Select the corresponding black magic card in **Media Configuration**.
     * Go to **Edit | Project Settings | Engine | General Settings | Timecode Provider** and select **BP_BlackMagicTCProvider**.
     * Go to **Edit | Project Settings | Engine | General Settings | Custom Timestep** and select **BP_BlackMagicGenlock**.
     * Close editor and restart the project
7. [Setup Vive using Live Link UE4 Documentation](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/LiveLinkPlugin/Livelinkxr/)
     * Connect Vive to PC
     * Calibrate vive track with Vive VR
     * Add Live Link XR Plugin: **Edit | PLugins | Live Link XR**.
     * Add **Windows | Live Link** tool.
     * Make sure Steam VR is running and turn on tracker on camera
     * Press **+ Source** and make sure **Track Trackers** is selected and others are not and set the **Loca Update Rate** to `90`.
     * Press the <kbd>Add</kbd> button.
     * Make sure it adds a Steam VR tracker to the **Subject Name** window.  If not restart unreal with steam trackers on.
    
8. [Create a Blueprint to Make Camera Follow Tracker with Smoothing](https://www.youtube.com/watch?v=jx8cxoW5vnc&t=96s)
     * Add new **Actor Blueprint** called `BP_Camera`.
     * Add a **Live Link Controller** component.
     * Select **Subject Representation** and select the Steam VR tracker.
     * Underneath **Role Controller | Live Link** untick **World Transform**.
     * Add two **Arrow Components**.  Call one **Tracker** and the other **Camera**. Put Camera under tracker.  Change color of camera arrow to blue.
     * Add an offset for the camera arrow.  The **Z** offset on our rig was `-33.02cm`.  The **X** offset at **-2.54cm**. This is based on how far the center of the tracker is to the center of the sensor on the film camera
     * Go to the **Event Graph** and a **Live Link | On Live LInk Update +**.  This adds a live link update in the blueprint.
     * Add a **Variable** called **Smoothing** and as a **Float**. Make it public. Make the default `15.0`.
     * Add a **Variable** called **CineCamera** of type **Cine Camera Actor | Object Reference**. Make it public.
     * Add a **Variable** called **Last Transform** of type **Transform**. Make it private.
     * Drag a **Camera (Arrow?)** to Blueprint and add a **Get World Transform** node.
     * Drag a **Last Transform** to the graph and add a **LERP** node.
     * Plug smoothing amount into **Multiply** node and multiply by **Delta Time** then plug into the **Alpha** of LERP and output of **Get World** to **B** side of lerp.
     * Grab the **Cine Camera** and add a **Set Actor Location and Rotation**. Add execution pin from Live Link update to Set Location and Rotation.  Connect the output of the LERP to the input of the location and rotation.
     *  Reset the new **Last Transform** by setting it to the newest value.
9. Add Camera and Blueprint to game to control virutal camera with RED cam.
     *  Drag **BP_Camera** in game.  Add a **Cine Camera** to the scene and assign it to the **BP_Camera** blueprint in the **World Outliner** in the level you are workign in.
     *  Adjust **CineCamera Actor | Current Camera Settings | Filmback | Sensor Width** to `27.7 mm` and **Sensor Height** to `14.6 mm`.  Also set the aperture and focal length manually. 
     *  Add empty actor above **BP_Camera** so you can readjust 0,0,0 in world and call it **Camera Origin**.
     *  Add a **Take Recorder** and add **Cin Camera** to the recorder.
     *  Add a plugin called **Edit | Plugins | Time Data Monitor** to the project and reboot Unreal.
     *  Select **Window | Developer | Time Data Monitor**.  Add a time correction (in our case it was `.0125`.

10.  Output to external Black Magic
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

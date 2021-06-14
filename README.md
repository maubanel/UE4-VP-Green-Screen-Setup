# UE4-VP-Green-Screen-Setup

* Connect camera and black magic a valid sync signal
* Create a new blank film project
* [Black Magic Quick Start](https://docs.unrealengine.com/4.26/en-US/WorkingWithMedia/ProVideoIO/BlackmagicQuickStart/)
* Add a **Media  | Media Bundle** to a folder called `MB_LiveCamera`.
* Open media bundle and assigned **Media Source** a `Black Magic Media Source` and `1080 Progressive 23.976`.  
* Set timecode to `LTC`.
* Set **Reopen Source on Error** to `true` as the black magic does lose track of hte camera
* Turn on the RED Camera
* On RED **Medu | Settings | Display | Monitor Control** change **LCD** to `HDSDI`.  This sends the output of the camera through the BNC cable
* On the **Media Bundle**, select **Request Play Media**.
* [Time Code and Genlock](https://docs.unrealengine.com/4.26/en-US/WorkingWithMedia/ProVideoIO/TimecodeGenlock/)
* Make sure you have **WIndows | Developer Tools | Timecode Privder** selected and mount in top right
* Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicTimecodeProvider**.  Call it `BP_BlackMagicTCProvider`. Select the corresponding black magic card in **Media Configuration** as well as **LTC**timecode.
* Create a new blueprint in **Content | Blueprints**. Select a **All Classes | BlackMagicCustomTimestep**.  Call it `BP_BlackMagicGenlock`.  Select the corresponding black magic card in **Media Configuration**.
* Go to **Edit | Project Settings | Engine | General Settings | Timecode Provider** and select **BP_BlackMagicTCProvider**.
* Go to **Edit | Project Settings | Engine | General Settings | Custom Timestep** and select **BP_BlackMagicGenlock**.
* Close editor and restart the project
* Go to [Live Link Documentation](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/LiveLinkPlugin/Livelinkxr/)
* Add Live Link XR Plugin: **Edit | PLugins | Live Link XR**.
* Add **Windows | Live Link** tool.
* Make sure Steam VR is running and turn on tracker on camera
* Press **+ Srouce** and make sure **Track Trackers** is selected and others are not and set the **Framerate** to `90`.
* Make sure it adds a Steam VR tracker to the **Subject Name** window.  If not restart unreal with steam trackers on.
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
*  Drag **BP_Camera** in game.  Add a **Cine Camera** to the scene and assign it to the **BP_Camera** blueprint.
*  Add empty actor above **BP_Camera** so you can readjust 0,0,0 in world and call it **Camera Origin**.
*  Add a **Take Recorder** and add **Cin Camera** to the recorder.
*  Add a plugin called **Edit | Plugins | Time Data Monitor** to the project and reboot Unreal.
*  Select **Window | Developer | Time Data Monitor**.  Add a time correction (in our case it was `.0125`.
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

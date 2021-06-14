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
* 

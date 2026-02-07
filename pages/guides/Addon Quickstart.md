\page addon-quickstart Addon Quickstart
This is a full guide on how to get started with the AddonHost and implement your own addon. This guide will remain build-system agnostic and avoid compiler extensions where possible.

\note
This guide will be using the C++ programming language. Creation of addons in languages other than C++ are outside the scope of this guide.  
If you want to program in another language, check out the [community bindings](@ref bindings)

If all you need is an example, see [The Compass Addon](https://github.com/RaidcoreGG/GW2-Compass) (Built with Visual Studio)  
Alternately, see [NexPad](https://github.com/ChristopherJTrent/NexPad) for a GNU make + GCC example using Msys2  

Each addon loaded by nexus is required to export a function matching the signature `AddonDefinition_t* GetAddonDef()`. See [AddonDefinition_t](@ref AddonDefinition_t) for the full struct definition

Example implementation of GetAddonDef
```cpp
AddonDefinition_t AddonDef{};

void AddonLoad(AddonAPI_t*) {} //add a body as needed.
void AddonUnload() {} //add a body as needed.

extern "C" __declspec(dllexport) AddonDefinition_t GetAddonDef()
{
	AddonDef.Signature = -1; //This must be positive and unused if your addon is available on nexus and negative otherwise.
	AddonDef.APIVersion = NEXUS_API_VERSION; //this is defined in nexus.h, and contains the current latest version of nexus.
	AddonDef.Name = "Example Addon";
	AddonDef.Version.Major = 1; //Treat AddonDef.Version as a SemVer compatible version number.
	AddonDef.Version.Minor = 0;
	AddonDef.Version.Build = 0;
	AddonDef.Version.Revision = 1;
	AddonDef.Author = "Raidcore";
	AddonDef.Description = "Example Addon for Documentation";
	AddonDef.Load = *AddonLoad;
	AddonDef.Unload = *AddonUnload;

	//Optional:
	AddonDef.Provider = EUpdateProvider::UP_None; //See EUpdateProvider in nexus.h
	AddonDef.UpdateLink = "https://github.com/RaidcoreGG/GW2-Example-Addon" // this link doesn't actually exist.
}
```
Fields not explicitly marked as optional are required.  
`Load()` will be called when your addon is loaded, and should initialize anything your addon needs. The `AddonAPI_t*` it receives will contain a version of the nexus API matching the one specified in `AddonDef.APIVersion`. See [AddonAPI_t](@ref AddonAPI_t) for the current version of that API.  
`Unload()` will be called when the game shuts down, when your addon is updated, and when it is unloaded. If your addon should not be able to be unloaded at runtime, set `AddonDef.Flags` to `EAddonFlags::DisableHotloading`.

### Addon Signatures  
If your addon is hosted on Raidcore, this should be the unique ID of your addon.  
If your addon is *not* hosted on Raidcore, it should be any negative integer.  

\note
Do not set it to 0, and do not use the signature of another addon.

### API Versions
If you don't use any of Nexus's functions, set APIVersion to 0.  
If APIVersion is a non-zero value, it must be a current or previous `NEXUS_API_VERSION`.  

\warning
Your addon will not load if you set APIVersion to an invalid value.

## The Addon API reference
It is recommended to store the reference to the addon API in a shared place, so you can include it in any files that need it. An example of how to do this is shown below.

### `Shared.h`
```cpp
#pragma once
#include "nexus.h"
//#include "imgui.h"

extern AddonAPI_t* Addon_API;
```

### `Shared.cpp`
```cpp
#include "Shared.h"

AddonAPI_t Addon_API;
```

### `ModuleMain.cpp`
```cpp
void AddonLoad(AddonAPI_t* API) {
	Addon_API = API;

	//Uncomment the next two lines if you are using imgui, and have included it in shared.h
	//ImGui::SetCurrentContext((ImGuiContext*)Addon_API->ImguiContext);
	//ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))Addon_API->ImguiMalloc, (void(*)(void*, void*))Addon_API->ImguiFree);
}
void AddonUnload() {}

extern "C" __declspec(dllexport) AddonDefinition_t GetAddonDef() {
	//... Omitted for brevity, see above for full contents.
}
```

## About unloading
When `AddonDefinition_t::Unload()` is called, you should free any resources you have allocated. E.G. Keybinds.  
Nexus will do its best to clean up after you should you forget, but no guarantee is made that it will do so perfectly.  
To prevent memory leaks, clean up after yourself.  

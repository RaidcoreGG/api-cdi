\page addon-quickstart Addon Quickstart
This is a complete guide on how to get started with the AddonHost and implement your own addon. It is build-system agnostic and avoids compiler-specific extensions where possible.

\note
This guide uses the C++ programming language. Creating addons in languages other than C++ is outside the scope of this guide.
If you want to program in another language, check out the [community bindings](@ref bindings).

If all you need is an example, numerous Nexus addons are open source, notably:
1. [The Compass Addon](https://github.com/RaidcoreGG/GW2-Compass), which includes Visual Studio project files  
2. [NexPad](https://github.com/ChristopherJTrent/NexPad), a GNU Make + GCC example using MSYS2  
3. [TrueWorldCompletion](https://github.com/jsantorek/GW2-TrueWorldCompletion), which uses CMake and Clang (with Docker and Conan as options)

Each addon loaded by Nexus is required to export a function matching the signature `AddonDefinition_t* GetAddonDef()`. See [AddonDefinition_t](@ref AddonDefinition_t) for the full struct definition.

### Example implementation of `GetAddonDef`
```cpp
#include <Nexus.h>

AddonDefinition_t AddonDef{};

void AddonLoad(AddonAPI*) {}   // Add a body as needed.
void AddonUnload() {}          // Add a body as needed.

extern "C" __declspec(dllexport) AddonDefinition* GetAddonDef()
{
    AddonDef.Signature = -1; // Must be positive and unused if hosted on Nexus; negative otherwise.
    AddonDef.APIVersion = NEXUS_API_VERSION; // Defined in Nexus.h; the latest supported API version.
    AddonDef.Name = "Example Addon";
    AddonDef.Version.Major = 1; // Treat AddonDef.Version as a SemVer-compatible version.
    AddonDef.Version.Minor = 0;
    AddonDef.Version.Build = 0;
    AddonDef.Version.Revision = 1;
    AddonDef.Author = "Raidcore";
    AddonDef.Description = "Example Addon for Documentation";
    AddonDef.Load = &AddonLoad;
    AddonDef.Unload = &AddonUnload;

    // Optional:
    AddonDef.Provider = EUpdateProvider::EUpdateProvider_None; // See EUpdateProvider in Nexus.h
    AddonDef.UpdateLink = "https://github.com/RaidcoreGG/GW2-Example-Addon"; // This link doesn't actually exist.

    return &AddonDef;
}
```
Fields not explicitly marked as optional are required.

`Load()` will be called when your addon is loaded and should initialize anything the addon needs. The `AddonAPI*` parameter will contain a version of the Nexus API matching the one specified in `AddonDef.APIVersion`. See [AddonAPI_t](@ref AddonAPI_t) for the current API definition.

`Unload()` will be called when the game shuts down, when your addon is updated, or when it is manually unloaded. If your addon must not be unloaded at runtime, set `AddonDef.Flags` to `EAddonFlags::DisableHotloading`.

### Addon Signatures
If your addon is hosted on Raidcore, this should be the unique ID of your addon.
If your addon is *not* hosted on Raidcore, it should be any negative integer.

\note
Do not set the signature to 0, and do not use the signature of another addon.

### API Versions
If you do not use any Nexus API functions, set APIVersion to 0.
If APIVersion is non-zero, it must be a current or previous `NEXUS_API_VERSION`.

\warning
Your addon will not load if APIVersion is set to an invalid value.

## The Addon API reference
It is recommended to store the addon API reference in a shared location so it can be accessed from any file that needs it. An example is shown below.

### `Shared.h`
```cpp
#pragma once
#include "Nexus.h"
//#include "imgui.h"

extern AddonAPI* Addon_API;
```

### `Shared.cpp`
```cpp
#include "Shared.h"

AddonAPI Addon_API;
```

### `ModuleMain.cpp`
```cpp
void AddonLoad(AddonAPI* API) {
	Addon_API = API;

	// Uncomment the next two lines if you are using ImGui and have included it in Shared.h
	//ImGui::SetCurrentContext((ImGuiContext*)Addon_API->ImguiContext);
	//ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))Addon_API->ImguiMalloc, (void(*)(void*, void*))Addon_API->ImguiFree);
}

void AddonUnload() {}

extern "C" __declspec(dllexport) AddonDefinition* GetAddonDef() {
	// ... Omitted for brevity, see above for full example.
}
```
## About unloading
When `AddonDefinition_t::Unload()` is called, you should free any resources you have allocated (e.g. keybinds).

Nexus will attempt to clean up resources if you forget, but no guarantee is made that cleanup will be complete or correct.

To prevent memory leaks and other issues, always clean up after yourself.

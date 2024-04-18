# Getting started
This is a full guide on how to get started with the AddonHost and implement your own addon.

If you just need an example, not a guide, refer to [the Compass Mod](https://github.com/RaidcoreGG/GW2-Compass).

Also check out the 1:1 port of the [ArcDPS Combatdemo](https://github.com/RaidcoreGG/Nexus-ArcDPS_CombatDemo).

If you just want the definitions look at [`Nexus.h`](https://github.com/RaidcoreGG/RCGG-lib-nexus-api).

Each addon loaded by the host requires to have one function exported `GetAddonDef`. The signature is as simple as it gets, all it has to do is return a pointer to a struct with the addon's definitions: `AddonDefinitions* GetAddonDef()`.

The struct in question:
```cpp
struct AddonVersion
{
	signed short	Major;
	signed short	Minor;
	signed short	Build;
	signed short	Revision;
};

struct AddonDefinition
{
	/* required */
	signed int      Signature;      /* Raidcore Addon ID, set to random unqiue negative integer if not on Raidcore */
	signed int		APIVersion;		/* Determines which AddonAPI struct revision the Loader will pass, use the NEXUS_API_VERSION define from Nexus.h */
	const char*     Name;           /* Name of the addon as shown in the library */
	AddonVersion	Version;
	const char*     Author;         /* Author of the addon */
	const char*     Description;    /* Short description */
	ADDON_LOAD      Load;           /* Pointer to Load Function of the addon */
	ADDON_UNLOAD    Unload;         /* Pointer to Unload Function of the addon. Not required if EAddonFlags::DisableHotloading is set. */
	EAddonFlags     Flags;          /* Information about the addon */

	/* update fallback */
	EUpdateProvider Provider;       /* What platform is the the addon hosted on */
	const char*     UpdateLink;     /* Link to the update resource */
};
```

`Example implementation of GetAddonDef()`
```cpp
AddonDefinition AddonDef{};

extern "C" __declspec(dllexport) AddonDefinition* GetAddonDef()
{
	AddonDef.Signature = 17;
	AddonDef.APIVersion = NEXUS_API_VERSION; // taken from Nexus.h
	AddonDef.Name = "World Compass";
	AddonDef.Version.Major = 1;
	AddonDef.Version.Minor = 0;
	AddonDef.Version.Build = 0;
	AddonDef.Version.Revision = 1;
	AddonDef.Author = "Raidcore";
	AddonDef.Description = "Adds a simple compass widget to the UI, as well as to your character in the world.";
	AddonDef.Load = AddonLoad;
	AddonDef.Unload = AddonUnload;
	AddonDef.Flags = EAddonFlags::None;

	/* not necessary if hosted on Raidcore, but shown anyway for the example also useful as a backup resource */
	AddonDef.Provider = EUpdateProvider::GitHub;
	AddonDef.UpdateLink = "https://github.com/RaidcoreGG/GW2-Compass";

	return &AddonDef;
}
```

Besides **Provider & UpdateLink** all fields are required.
Load() will be called to initialise your addon.
Unload() when the game shuts down, or the addon is updated or unloaded, if your addon shouldn't unload at runtime set EAddonFlags::DisableHotloading or leave `Unload() = 0;`.

It is recommended to keep an Unload function for when the game shut downs however.

> **About addon signatures**:  
If your addon is hosted on Raidcore, this should be the unique ID of your addon.  
If it's not hosted on Raidcore, choose any *negative* integer.  
Do not use 0. Do not use any signature that's used by another addon.

> **About API versions**:  
If you don't use any of the functions Nexus provides, set APIVersion to 0.  
If you use a non-zero value, it has to be one that actually exists or *existed* in the past as defined in NEXUS_API_VERSION in [`Nexus.h`](./Definitions/Nexus.h) as otherwise there will not be a matching version and therefore the addon won't be loaded.

The Load function will receive a struct, matching the requested API version containing all the API functions and resources at your disposal.

`API Revision 1`
```cpp
struct AddonAPI
{
	/* Renderer */
	IDXGISwapChain*				SwapChain;
	ImGuiContext*				ImguiContext;
	void*						ImguiMalloc;
	void*						ImguiFree;
	GUI_ADDRENDER				RegisterRender;
	GUI_REMRENDER				UnregisterRender;

	/* Paths */
	PATHS_GETGAMEDIR			GetGameDirectory;
	PATHS_GETADDONDIR			GetAddonDirectory;
	PATHS_GETCOMMONDIR			GetCommonDirectory;

	/* Minhook */
	MINHOOK_CREATE				CreateHook;
	MINHOOK_REMOVE				RemoveHook;
	MINHOOK_ENABLE				EnableHook;
	MINHOOK_DISABLE				DisableHook;

	/* Logging */
	LOGGER_LOGA					Log;

	/* Events */
	EVENTS_RAISE				RaiseEvent;
	EVENTS_SUBSCRIBE			SubscribeEvent;
	EVENTS_SUBSCRIBE			UnsubscribeEvent;

	/* WndProc */
	WNDPROC_ADDREM				RegisterWndProc;
	WNDPROC_ADDREM				UnregisterWndProc;

	/* Keybinds */
	KEYBINDS_REGISTERWITHSTRING	RegisterKeybindWithString;
	KEYBINDS_REGISTERWITHSTRUCT	RegisterKeybindWithStruct;
	KEYBINDS_UNREGISTER			UnregisterKeybind;

	/* DataLink */
	DATALINK_GETRESOURCE		GetResource;
	DATALINK_SHARERESOURCE		ShareResource;

	/* Textures */
	TEXTURES_GET				GetTexture;
	TEXTURES_LOADFROMFILE		LoadTextureFromFile;
	TEXTURES_LOADFROMRESOURCE	LoadTextureFromResource;
	TEXTURES_LOADFROMURL		LoadTextureFromURL;

	/* Shortcuts */
	QUICKACCESS_ADDSHORTCUT		AddShortcut;
	QUICKACCESS_REMOVE			RemoveShortcut;
	QUICKACCESS_ADDSIMPLE		AddSimpleShortcut;
	QUICKACCESS_REMOVE			RemoveSimpleShortcut;
};
```

It is recommended to store the struct somewhere in a globally accessible variable you can include in all your files. Example below.

Check out the [functions readme](FUNCTIONS.md) on details to each API function.

`Shared.h`
```cpp
#ifndef SHARED_H
#define SHARED_H

extern AddonAPI* ApiDefs;

#endif
```
`Shared.cpp`
```cpp
#include "Shared.h"

AddonAPI* ApiDefs;
```
`ModuleMain.cpp`
```cpp
#include "Shared.h"

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext((ImGuiContext*)APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);
}

void AddonUnload()
{
    /* free some resources */
}
```

> **About addon unloading**:   
Your unload function should free all resources you previously allocated.  
For example, if you added a keybind you should free it here.  
*In case you forget:*  
*Don't worry. Nexus will automatically release leftover references.*  
*Don't make it a habit though.*

The API info should be pretty self-explanatory, but just in case.
AddonAPI contains the SwapChain for custom rendering, the ImguiContext and its Malloc and Free functions, which will be required to be set in your load function, if you want to render ImGui, see above example.

It also contains Minhook functions, so you don't have to add an additional dependency to hook.

Other than those functions, the API struct will also have everything you need to use keybinds, events and resources. More below.

---

# Using Events, Keybinds, Resources and all the other good stuff

## Rendering & Options
Nexus has several Render Callbacks for you to add to. In the API definitions you will find `GUI_ADDRENDER RegisterRender;` and `GUI_REMRENDER UnregisterRender;`
RegisterRender takes the ERenderType for you to define where you want your function to be called. The available options are:
- PreRender
- Render,
- PostRender,
- OptionsRender

Here's a little pseudocode overview where they are called:
```cpp
void Render()
{
	PreRenderCallbacks(); // <- This is where your stuff is called.

	ImGui_ImplWin32_NewFrame();
	ImGui_ImplDX11_NewFrame();
	ImGui::NewFrame();

	RenderCallbacks(); // <- This is where your stuff is called.

	ImGui::EndFrame();
	ImGui::Render();
	Renderer::DeviceContext->OMSetRenderTargets(1, &Renderer::RenderTargetView, NULL);
	ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData());

	PostRenderCallbacks(); // <- This is where your stuff is called.
}
```

The exception is the OptionsRender which will be called in the ImGui window for you to draw your options.

UnregisterRender takes just your callback again and removes it from the callback list.

## Paths
Nexus implements a few functions for you to easily get some important path strings.
Those being:
- `const char* GetGameDirectory();`
	- returns for example "C:\Program Files\Guild Wars 2"
- `const char* GetAddonDirectory(const char* aName);`
	- returns for example "C:\Program Files\Guild Wars 2\addons" if you pass an empty string ("")
	- returns for example "C:\Program Files\Guild Wars 2\addons\Compass" if you pass "Compass"
- `const char* GetCommonDirectory();`
	- returns for example "C:\Program Files\Guild Wars 2\addons\common"

All these directories, obviously will be YOUR game path.

## Minhook
Nexus shares MinHook functions, they will not be explained further.

## Logging
Nexus shares a simple Log function, with which you can print to the ingame window and the log file.

The function takes a LogLevel parameter and the string itself, any formatting will have to be done on the developer's end.

## Keybinds
If every addon implemented their own WndProc and did the same checks all the other addons do, it would get a bit redundant, wouldn't it? You also cannot maintain any sort of consistent behaviour among those addons. That's why Raidcore Nexus defines *keybinds* instead.

To use a keybind you simply have to implement a handler function for keybinds. In this function you define what to do if a given keyword is passed, you can use the same function for multiple Keybinds but you will have to compare against the passed identifier.

`ModuleMain.cpp` - KeybindHandler
```cpp
void ProcessKeybind(const char* aIdentifier)
{
    /* if KB_COMPASS_TOGGLEVIS is passed, we toggle the compass visibility */
	if (strcmp(aIdentifier, "KB_COMPASS_TOGGLEVIS") == 0)
	{
		IsCompassVisible = !IsCompassVisible;
		return;
	}
}
```

After that you can register your keybinds!

```cpp
#include "Shared.h"

void SetupKeybinds()
{
    APIDefs->RegisterKeybind("KB_COMPASS_TOGGLEVIS", ProcessKeybind, "CTRL+C");
}
```

You could also register the same keybind with a struct if you prefer it that way, however keep in mind that **ScanCodes** are used, not **VirtualKeyCodes**.

```cpp
#include "Shared.h"

void SetupKeybinds()
{
	/* set keybinds */
	Keybind kbCompassToggle{};
	kbJackal.Ctrl = true;
	kbJackal.Key = 46;

	APIDefs->RegisterKeybindWithStruct("KB_COMPASS_TOGGLEVIS", ProcessKeybind, kbJackal);
}
```

Now you've set up a keybind that will invoke `KB_COMPASS_TOGGLEVIS` whenever `Ctrl + C` is pressed. Your handler function will receive this and you can do whatever you want with it! If you want to add more keybinds, simply add more checks to your `ProcessKeybind()` function.

## Events
Using events is as simple as keybinds!
You will need a function to handle *this specific* event and somewhere in your code you have to register your event and handler.

`ModuleMain.cpp` - EventHandler
```cpp
void HandleMumbleIdentityUpdate(void* aEventArgs)
{
    /* aEventArgs will be some data from this specific event,
       if you are working with other addons
       THE TYPE SHOULD BE KNOWN */
    /* in this case aEventArgs is a MumbleIdentity struct, refer to the GW2 Wiki if you want to know more, the exact data is not relevant */

    MumbleIdentity identity = static_cast<MumbleIdentity>(aEventArgs);
    Renderer::SetNewFOV(identity.FOV);
}
```
Check out the [events readme](EVENTS.md) for an overview of all default-available events and their payloads.

Subscribing to the event
```cpp
#include "Shared.h"

void SetupEvents()
{
    APIDefs->SubscribeEvent("EV_MUMBLE_IDENTITY_UPDATED", HandleMumbleIdentityUpdate);
}
```

Now you're all set to receive event callbacks! The example event above is actually implemented by Raidcore and you can subscribe to it!

### Raising events
Raising events is also very simple, let's send some random data each time we're looking north!

`ModuleMain.cpp`
```cpp
#include "Shared.h"

void MainFunction()
{
    for(;;)
    {
        if (MumbleLink->Camera.IsLookingNorth())
        {
            const wchar_t* evData = "Pogey, we looked north!";

            ApiDefs.RaiseEvent("EV_COMPASS_LOOKEDNORTH", (void*)evData);
        }
    }
}
```
That's all there is to raising an event, you simply call the raise function with your data passed as a void pointer. If you define events for other addons to use, make sure to document the contents of evData to avoid unnecessary crashes due to casting and memory access violation errors.

---
## Resource & Function sharing
Nexus allows you to share a struct with functions or data, similar to the Mumble API. (It also already shares the Mumble API!)

To get a set of data, for example the MumbleLink, you can use the function from the API definitions. Let's extend the `AddonLoad()` function from above to get the Mumble and Nexus Data.
The `GetResource()` function returns a void pointer, so you will have to cast it to the correct struct.

`ModuleMain.cpp`
```cpp
#include "Shared.h"

MumbleLink* Mumble = nullptr;
NexusLinkData* NexusLink = nullptr;

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);

	// Mumble Link is always shared within Nexus as "DL_MUMBLE_LINK"
	Mumble = (MumbleLink*)APIDefs->GetResource("DL_MUMBLE_LINK");
	NexusLink = (NexusLinkData*)APIDefs->GetResource("DL_NEXUS_LINK");
}
```

Now you can use the data from these pointers.

To register your own Resource you will have to pass an identifier and its size to the `ShareResource()` function, Nexus will allocate that memory for you and you can write to it freely. Let's extend the `Load()` function, once more.

> **Important**  
Your identifier will be usable as it was given e.g. `DL_MYSHAREDDATA`, however internally the process ID will be appended e.g. `DL_MYSHAREDDATA_12345` this ensures multiboxing compatibility.
You don't have to do anything else.

`ModuleMain.cpp`
```cpp
#include "Shared.h"

MumbleLink* Mumble = nullptr;
NexusLinkData* NexusLink = nullptr;

struct SomeData
{
	int Field1;
	short Field2;
	HMODULE Field3;
}

SomeData* someData;

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);

	// Mumble Link is always shared within Nexus as "DL_MUMBLE_LINK"
	Mumble = (MumbleLink*)APIDefs->GetResource("DL_MUMBLE_LINK");
	NexusLink = (NexusLinkData*)APIDefs->GetResource("DL_NEXUS_LINK");

	someData = APIDefs.ShareResource("someData", sizeof(SomeData));
	someData->Field2 = 5;
}
```

> **Please ensure this behaviour in your `Unload()` function**:   
Your unload function should free all resources you previously allocated.  
For example, if you added a keybind you should free it here.  
*In case you forget:*  
*Don't worry. Nexus will automatically release leftover references.*  
*Don't make it a habit though.*
If the addon, which created it unloads, all these bytes should be either overwritten with null, if they are functions pointers, so the addons using it, don't call invalid memory. And if it's static data, it can stay.

## Textures
Nexus allows you to easily load textures and use already loaded ones.
To get a Texture that was already loaded, you can call `GetTexture()` and pass the identifier of the texture, it will return a struct of the texture or a nullptr if it doesn't exist.

For example, if you want to get the Nexus icon, let's add it to the Load() function of our addon again and we show it in the options render callback:

`ModuleMain.cpp`
```cpp
#include "Shared.h"

Texture* nexusIconTexture = nullptr;

// proto
void AddonOptions();

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);

	nexusIconTexture = aApi->GetTexture("ICON_NEXUS");
	aApi->RegisterRender(ERenderType::OptionsRender, AddonOptions);
	// if you want to use that returned texture, make sure it is NOT a nullptr
}

void AddonOptions()
{
	if (nexusIconTexture != nullptr && nexusIconTexture->Resource != nullptr)
	{
		ImGui::Image(nexusIconTexture->Resource, ImVec2(nexusIconTexture->Width, nexusIconTexture->Height));
	}
}
```

Now you loaded the Texture and rendered it to the options window.

To load textures yourself, you can do so via URL, Resource or from File the parameters you pass slightly differ, but the general idea is the same:

You pass an identifier for the the texture, where to find and finally a callback for when it's loaded, since textures aren't loaded immediately.

Here's an example again from within the Load function:

`ModuleMain.cpp`
```cpp
#include "Shared.h"

Texture* myCoolImage = nullptr;

// proto
void ReceiveTexture(const char* aIdentifier, Texture* aTexture);

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);

	aApi->LoadTextureFromFile("TEX_MYCOOLIMAGE", "C:\Program Files\Guild Wars 2\mySuperCoolImage.png", ReceiveTexture);
}

void ReceiveTexture(const char* aIdentifier, Texture* aTexture)
{
	std::string str = aIdentifier;

	if (str == "TEX_MYCOOLIMAGE")
	{
		myCoolImage = aTexture;
	}
}
```

You will receive a call to ReceiveTexture if your image was loaded successfully, usually this happens before the next frame. You can now use your texture in the same way as when getting it.

## QuickAccess
The Quick Access is the little shortcut bar on the top left of your screen. Raidcore Nexus adds to it to make certain functions easily accessible.

![](https://i.imgur.com/n6XKCzd.png)

You can either add your own icon or you can simply add some ImGui Checkboxes or whatever elements you want, when you right-click the Nexus icon.

> If you need help creating an icon that looks the same as the Guild Wars 2 style icons, feel free to contact `deltagw2` on Discord or `Delta.1074` in-game. I'll gladly help you make one.

Let's add an icon shortcut and a simple shortcut, for this we will need a texture for the icon and one for when you're hovering the icon. This code assumes, you've already loaded the textures or more precisely, you can add the shortcut and add the textures later.

Keep in mind, the `AddShortcut()` function, invokes a Keybind, this way you can for example use a shortcut icon to open a window for which you don't have a key set.

`ModuleMain.cpp`
```cpp
#include "Shared.h"

// proto
void RenderShortcut();

void AddonLoad(AddonAPI* aApi)
{
    APIDefs = aApi;

	/* If you are using ImGui you will need to set these. */
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);

	aApi->AddSimpleShortcut("QAS_COMPASS", RenderShortcut);
	aApi->AddShortcut("QA_COMPASS", "ICON_COMPASS", "ICON_COMPASS_HOVER", "KB_COMPASS_TOGGLEVIS", "This icon is for the compass, this is a tooltip.");

	// even though the textures are loaded after adding the shortcut, it will load successfully as it waits for the texture to load
	aApi->LoadTextureFromFile("ICON_COMPASS", "C:\Program Files\Guild Wars 2\ICON_COMPASS.png", ReceiveTexture);
	aApi->LoadTextureFromFile("ICON_COMPASS_HOVER", "C:\Program Files\Guild Wars 2\ICON_COMPASS_HOVER.png", ReceiveTexture);

}

void RenderShortcut()
{
	// two simple checkboxes for when you right click on the nexus icon
	ImGui::Checkbox("Compass Strip", &IsCompassStripVisible);
	ImGui::Checkbox("Compass World", &IsWorldCompassVisible);
}
```
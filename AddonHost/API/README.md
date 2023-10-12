# Getting started
This is a full guide on how to get started with the AddonHost and implement your own addon.

If you just need an example, not a guide, refer to [the Compass Mod](https://github.com/RaidcoreGG/GW2-Compass).

If you just want the definitions look at [`Nexus.h`](./Definitions/Nexus.h).

Each addon loaded by the host requires to have one function exported `GetAddonDef`. The signature is as simple as it gets, all it has to do is return a pointer to a struct with the addon's definitions: `AddonDefinitions* GetAddonDef()`.

The struct in question:
```cpp
struct AddonDefinition
{
	/* required */
	signed int      Signature;      /* Raidcore Addon ID, set to random unqiue *negative* integer if not on raidcore.gg */
	signed int		APIVersion;		/* Determines which AddonAPI struct revision the Loader will pass, use the NEXUS_API_VERSION define from Nexus.h */
	const char*     Name;           /* Name of the addon as shown in the library */
	signed short	VersionMajor;   /* Version info */
	signed short	VersionMinor;
	signed short	VersionBuild;
	signed short	VersionRevision;
	const char*     Author;         /* Author of the addon */
	const char*     Description;    /* Short description */
	ADDON_LOAD      Load;           /* Pointer to Load Function of the addon */
	ADDON_UNLOAD    Unload;         /* Pointer to Unload Function of the addon */
	EAddonFlags     Flags;          /* Information about the addon */

	/* update fallback */
	EUpdateProvider Provider;       /* What platform is the the addon hosted on */
	const char*     UpdateLink;     /* Link to the update resource */
};
```

`Example implementation of GetAddonDef()`
```cpp
AddonDefinition* AddonDef;

extern "C" __declspec(dllexport) AddonDefinition* GetAddonDef()
{
	AddonDef = new AddonDefinition();
	AddonDef->Signature = 17;
	AddonDef->APIVersion = NEXUS_API_VERSION; // 1
	AddonDef->Name = "World Compass";
	Version.Major = 1;
	Version.Minor = 0;
	Version.Build = 0;
	Version.Revision = 1;
	AddonDef->Version = Version;
	AddonDef->Author = "Raidcore";
	AddonDef->Description = "Adds a simple compass widget to the UI, as well as to your character in the world.";
	AddonDef->Load = AddonLoad;
	AddonDef->Unload = AddonUnload;
	AddonDef->Flags = EAddonFlags::None;

	/* not necessary if hosted on Raidcore, but shown anyway for the example also useful as a backup resource */
	AddonDef->Provider = EUpdateProvider::GitHub;
	AddonDef->UpdateLink = "https://github.com/RaidcoreGG/GW2-Compass";

	return AddonDef;
}
```

Besides **Provider & UpdateLink** all fields are required.
Load() will be called to initialise your addon.
Unload() when the game shuts down, or the addon is updated or unloaded.

> **About addon signatures**:  
If your addon is hosted on Raidcore, this should be the unique ID of your addon.  
If it's not hosted on Raidcore, choose any *negative* integer.  
Do not use 0. Do not use any signature that's used by another addon.

> **About API versions**:  
If you don't use any of the functions Nexus provides, set APIVersion to 0.  
If you use a non-zero value, it has to be one that actually exists or *existed* in the past as defined in NEXUS_API_VERSION in [`Nexus.h`](./Definitions/Nexus.h) as otherwise there will not be a matching version and therefore the addon won't be loaded.

The Load function will receive a struct, matching the requested API version containing all the API functions and resources at your disposal.

```cpp
struct AddonAPI
{
	/* Renderer */
	IDXGISwapChain*				SwapChain;
	ImGuiContext*				ImguiContext;
	void*						ImguiMalloc;
	void*						ImguiFree;
	GUI_REGISTER				RegisterRender;
	GUI_UNREGISTER				UnregisterRender;

	/* Minhook */
	MINHOOK_CREATE				CreateHook;
	MINHOOK_REMOVE				RemoveHook;
	MINHOOK_ENABLE				EnableHook;
	MINHOOK_DISABLE				DisableHook;

	/* Logging */
	LOGGER_LOGA					Log;
	LOGGER_ADDREM				RegisterLogger;
	LOGGER_ADDREM				UnregisterLogger;

	/* Events */
	EVENTS_RAISE				RaiseEvent;
	EVENTS_SUBSCRIBE			SubscribeEvent;
	EVENTS_SUBSCRIBE			UnsubscribeEvent;

	/* WndProc */
	WNDPROC_REGISTER			RegisterWndProc;
	WNDPROC_UNREGISTER			UnregisterWndProc;

	/* Keybinds */
	KEYBINDS_REGISTER			RegisterKeybind;
	KEYBINDS_UNREGISTER			UnregisterKeybind;

	/* DataLink */
	DATALINK_GETRESOURCE		GetResource;
	DATALINK_SHARERESOURCE		ShareResource;

	/* Textures */
	TEXTURES_GET				GetTexture;
	TEXTURES_LOADFROMFILE		LoadTextureFromFile;
	TEXTURES_LOADFROMRESOURCE	LoadTextureFromResource;

	/* Shortcuts */
	QUICKACCESS_ADDSHORTCUT		AddShortcut;
	QUICKACCESS_REMOVESHORTCUT  RemoveShortcut;
	QUICKACCESS_ADDSIMPLE		AddSimpleShortcut;
	QUICKACCESS_REMOVESIMPLE	RemoveSimpleShortcut;

	/* API */
		// GW2 API FUNCS
		// LOGITECH API FUNCS
};
```

It is recommended to store the struct somewhere in a globally accessible variable you can include in all your files. Example below.

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
	ImGui::SetCurrentContext(APIDefs->ImguiContext);
	ImGui::SetAllocatorFunctions((void* (*)(size_t, void*))APIDefs->ImguiMalloc, (void(*)(void*, void*))APIDefs->ImguiFree);
}

void AddonUnload()
{
    /* free some resources */
}
```

> **About addon unlolading**:   
Your unload function should free all resources you previously allocated.  
For example, if you added a keybind you should free it here.  
*In case you forget:*  
*Don't worry. Nexus will automatically release leftover references.*  
*Don't make it a habit though.*

The API info should be pretty self-explanatory, but just in case.
AddonAPI contains the SwapChain for custom rendering, the ImguiContext and its Malloc and Free functions, which will be required to be set in your load function, see above example.

It also contains Minhook functions, so you don't have to add an additional dependency to hook, same goes for logging. If you wanna add a custom logger refer to the section Logging.

Other than those functions, the API struct will also have everything you need to use keybinds, events and resources. More below.

If you want to use have an Options window, you should declare it via AddonDefinition->Flags. This will cause Nexus to raise `EV_OPTIONS_CALLED_{Signature}` events to which you can subscribe. More on events in the next section.

---

# Using Events, Keybinds, Resources and all the other good stuff
## Keybinds
If every addon implemented their own WndProc and did the same checks all the other addons do, it would get a bit redundant, wouldn't it? You also cannot maintain any sort of consistent behaviour among those addons. That's why Raidcore defines *keybinds* instead.

To use a keybind you simply have to implement a handler function for keybinds. In this function you define what to do if a given keyword is passed.

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

Subscribing to the event
```cpp
#include "Shared.h"

void SetupEvents()
{
    APIDefs->SubscribeEvent("EV_MUMBLE_IDENTITY_UPDATED",  HandleMumbleIdentityUpdate);
}
```

Now you're all set to receive event callbacks! The example event above is actually implemented by Raidcore and you can subscribe to it!

Here's a list of default events implemented in Raidcore:
1. EV_MUMBLE_IDENTITY_UPDATED
2. EV_WINDOW_RESIZED

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

# Logging
Normally you will only use the Log function inside of the AddonAPI struct.

## Custom Logging
Nexus by default implements a FileLogger, ConsoleLogger, if `-ggconsole` is set, and logging to the ImGui window in game.

If you still want to implement your own custom Logger, let's say you wanna send all log messages to a webhook, you can do so.

Your custom Logger will have to inherit from ILogger:

`ILogger.h`
```cpp
#ifndef ILOGGER_H
#define ILOGGER_H

#include "ELogLevel.h"
#include "LogEntry.h"

#include <mutex>

class ILogger
{
    public:
        ILogger() = default;
        virtual ~ILogger() = default;

        ELogLevel GetLogLevel();
        void SetLogLevel(ELogLevel aLogLevel);

        virtual void LogMessage(LogEntry aLogEntry) = 0;

    protected:
        ELogLevel LogLevel;
        std::mutex MessageMutex;
};

#endif
```

`ILogger.cpp`
```cpp
#include "ILogger.h"

ELogLevel ILogger::GetLogLevel()
{
    return LogLevel;
}

void ILogger::SetLogLevel(ELogLevel aLogLevel)
{
    LogLevel = aLogLevel;
}
```

You can now define your custom logger. Using the example of the console window logger.

```cpp
#include <iostream>
#include <iomanip>
#include "windows.h"

HANDLE hConsole;
FILE* iobuf;

class ConsoleLogger : public virtual ILogger
{
    public:
        ConsoleLogger()
        {
            AllocConsole();
            freopen_s(&iobuf, "CONIN$", "r", stdin);
            freopen_s(&iobuf, "CONOUT$", "w", stderr);
            freopen_s(&iobuf, "CONOUT$", "w", stdout);
            hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
        }
        ~ConsoleLogger()
        {
            FreeConsole();
        }

        LogMessage(LogEntry aLogEntry)
        {
            MessageMutex.lock();

            switch (aLogEntry.LogLevel)
            {
                case ELogLevel::CRITICAL:    SetConsoleTextAttribute(hConsole, 12); break;
                case ELogLevel::WARNING:     SetConsoleTextAttribute(hConsole, 14); break;
                case ELogLevel::INFO:        SetConsoleTextAttribute(hConsole, 10); break;
                case ELogLevel::DEBUG:       SetConsoleTextAttribute(hConsole, 11); break;
                default:                     SetConsoleTextAttribute(hConsole, 7); break;
            }
            
            std::wcout << aLogEntry.ToString();

            MessageMutex.unlock();
        }
};
```

Great! You defined a logger the AddonHost can use. All you have left to do is Register it with the AddonHost by using another function from the AddonAPI.

Extending the AddonLoad function from above:

```cpp
#include "Shared.h"

void AddonLoad(AddonAPI aHostApi)
{
    ApiDefs = aHostApi;
    ImGui::SetCurrentContext(aHostApi.ImguiContext);

    ConsoleLogger* cLog = new ConsoleLogger();
	cLog->SetLogLevel(ELogLevel::ALL);
	ApiDefs->RegisterLogger(cLog);
}
```

The LogLevel of the logger determines what messages it will receive, if you wanna do your own custom filtering, just subscribe to all. You can also later set this again.

That's all you need to do for your custom logging implementation!

# Resource & Function sharing

A pointer to the [MumbleLink for realtime positional and other data](https://wiki.guildwars2.com/wiki/API:MumbleLink).

# Textures

# Localisation

# GW2 API
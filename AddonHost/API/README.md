# Getting started
This is a full guide on how to get started with the AddonHost and implement your own addon.

If you don't need a guide and just an example refer to [the Compass Mod](../Compass).

If you just want the definitions look at [`Definitions.h`](./Definitions.h).

Each addon loaded by the host requires to have one function exported `GetAddonDef` signature is as simple as it gets, all it has to do is return a pointer to a struct with the addon's definitions: `AddonDefinitions* GetAddonDef()`.

The struct in question:
```cpp
typedef void (*ADDON_LOAD)(AddonAPI aHostApi);
typedef void (*ADDON_UNLOAD)();
typedef void (*ADDON_RENDER)();
typedef void (*ADDON_OPTIONS)();

struct AddonDefinition
{
    signed int      Signature;      /* Raidcore Addon ID, set to random negative integer if not on Raidcore */
    const wchar_t*  Name;           /* Name of the addon as shown in the library */
    const wchar_t*  Version;        /* Leave as `__DATE__ L" " __TIME__` to maintain consistency */
    const wchar_t*  Author;         /* Author of the addon */
    const wchar_t*  Description;    /* Short description */
    ADDON_LOAD      Load;           /* Pointer to Load Function of the addon */
    ADDON_UNLOAD    Unload;         /* Pointer to Unload Function of the addon */

    ADDON_RENDER    Render;         /* Present callback to render imgui */
    ADDON_OPTIONS   Options;        /* Options window callback, called when opening options for this addon */

    EUpdateProvider Provider;       /* What platform is the the addon hosted on */
    const wchar_t*  UpdateLink;     /* Link to the update resource */
};
```

`Example implementation of GetAddonDef()`
```cpp
AddonDefinition* AddonDef;

extern "C" __declspec(dllexport) void GetAddonDef()
{
    AddonDef = new AddonDef();
    AddonDef->Signature = 17;
    AddonDef->Name = L"World Compass";
    AddonDef->Version = __DATE__ L" " __TIME__;
    AddonDef->Author = L"Raidcore";
    AddonDef->Description = L"Adds a simple compass widget to the UI, as well as to your character in the world.";
    AddonDef->Load = AddonLoad;
    AddonDef->Unload = AddonUnload;

    AddonDef->Render = AddonRender;
    AddonDef->Options = AddonOptions;

    /* not necessary if hosted on Raidcore, but shown anyway for the  example also useful as a backup resource */
    AddonDef->Provider = EUpdateProvider::GitHub;
    AddonDef->UpdateLink = L"https://github.com/RaidcoreGG/GW2-Compass";

    return AddonDef;
}
```

At a **minimum** each addon must define **Signature, Name, Version, Author, Description** as well as **Load and Unload** functions. Those will be called to initialise and shutdown the addon.

> **About addon signatures**:   
If your addon is hosted on Raidcore, this should be the unique ID of your addon.  
If it's not hosted on Raidcore, choose any *negative* integer.  
Do not use 0.

The Load function will receive a struct containing all the API functions and resources at your disposal.

```cpp
typedef MH_STATUS(__stdcall* MINHOOK_CREATE)(LPVOID pTarget, LPVOID pDetour, LPVOID* ppOriginal);
typedef MH_STATUS(__stdcall* MINHOOK_REMOVE)(LPVOID pTarget);
typedef MH_STATUS(__stdcall* MINHOOK_ENABLE)(LPVOID pTarget);
typedef MH_STATUS(__stdcall* MINHOOK_DISABLE)(LPVOID pTarget);

struct VTableMinhook
{
	MINHOOK_CREATE		CreateHook;
	MINHOOK_REMOVE		RemoveHook;
	MINHOOK_ENABLE		EnableHook;
	MINHOOK_DISABLE		DisableHook;
};

typedef void (*LOGGER_LOGA)(ELogLevel aLogLevel, const char* aFmt, ...);
typedef void (*LOGGER_LOGW)(ELogLevel aLogLevel, const wchar_t* aFmt, ...);
typedef void (*LOGGER_ADDREM)(ILogger* aLogger);

struct VTableLogging
{
	LOGGER_LOGA			LogA;
	LOGGER_LOGW			LogW;
	LOGGER_ADDREM		RegisterLogger;
	LOGGER_ADDREM		UnregisterLogger;
};

typedef void (*EVENTS_RAISE)(const wchar_t* aEventName, void* aEventData);
typedef void (*EVENTS_SUBSCRIBE)(const wchar_t* aEventName, EVENTS_CONSUME aConsumeEventCallback);

typedef void (*KEYBINDS_REGISTER)(const wchar_t* aIdentifier, KEYBINDS_PROCESS aKeybindHandler, const wchar_t* aKeybind);
typedef void (*KEYBINDS_UNREGISTER)(const wchar_t* aIdentifier);

struct AddonAPI
{
	IDXGISwapChain*		SwapChain;
	ImGuiContext*		ImguiContext;
	LinkedMem*			MumbleLink;

	VTableMinhook		MinhookFunctions;
	VTableLogging		LoggingFunctions;

	/* Events */
	EVENTS_RAISE		RaiseEvent;
	EVENTS_SUBSCRIBE	SubscribeEvent;

	/* Keybinds */
	KEYBINDS_REGISTER	RegisterKeybind;

	/* API */
		// GW2 API FUNCS
		// LOGITECH API FUNCS
		// RESOURCE SHARING FUNCS
};
```

It is recommended to store the struct somewhere in a globally accessible variable you can include in all your files. Example below.

`Shared.h`
```cpp
#ifndef SHARED_H
#define SHARED_H

extern AddonAPI ApiDefs;

#endif
```
`Shared.cpp`
```cpp
#include "Shared.h"

AddonAPI ApiDefs;
```
`ModuleMain.cpp`
```cpp
#include "Shared.h"

void AddonLoad(AddonAPI aHostApi)
{
    ApiDefs = aHostApi;
    ImGui::SetCurrentContext(aHostApi.ImguiContext);
}

void AddonUnload()
{
    /* free some resources */
}
```

The API info should be pretty self-explanatory, but just in case.
AddonAPI contains the SwapChain for custom rendering, the ImguiContext, which will be required to be set in your load function, see above example. A pointer to the [MumbleLink for realtime positional and other data](https://wiki.guildwars2.com/wiki/API:MumbleLink).

It also contains Minhook functions, so you don't have to add an additional dependency to hook, same goes for logging. If you wanna add a custom logger refer to the section Logging.

Other than those functions, the API struct will also have everything you need to use keybinds, events and resources. More below.

Lastly you need to define your Rendering and Options functions.
These are simple callbacks to render ImGui.
Options will be called when the Options button within the Addon Library is called.

```cpp
void AddonRender()
{
    /* render some imgui */
}

void AddonOptions()
{
    /* render an options window */
}
```

---

# Using Events, Keybinds, Resources
## Keybinds
If every addon implemented their own WndProc and did the same checks all the other addons do, it would get a bit redundant, wouldn't it? You also cannot maintain any sort of consistent behaviour among those addons. That's why Raidcore defines *keybinds* instead.

To use a keybind you simply have to implement a handler function for keybinds. In this function you define what to do if a given keyword is passed.

`KeybindHandler`
```cpp
void ProcessKeybind(const wchar_t* aIdentifier)
{
    /* if COMPASS_TOGGLEVIS is passed, we toggle the compass visibility */
	if (wcscmp(aIdentifier, L"COMPASS_TOGGLEVIS") == 0)
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
    APIDefs.RegisterKeybind(L"COMPASS_TOGGLEVIS", ProcessKeybind, L"CTRL+C");
}
```

Now you've set up a keybind that will invoke `COMPASS_TOGGLEVIS` whenever Ctrl + C is pressed. Your handler function will receive this and you can do whatever you want with it! If you want to add more keybinds, simply add more checks to your ProcessKeybind() function.

## Events
Using events is as simple as keybinds!
You will need a function to handle this specific event and somewhere in your code you have to register your event and handler.

`EventHandler`
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

`Subscribing to the event`
```cpp
#include "Shared.h"

void SetupEvents()
{
    APIDefs.SubscribeEvent(L"MUMBLE_IDENTITY_UPDATE", HandleMumbleIdentityUpdate);
}
```

Now you're all set to receive event callbacks! The example event above is actually implemented by Raidcore and you can subscribe to it!

### Raising events
Raising events is also very simple, using the same compass function, let's send some random data each time we're looking north!

```cpp
#include "Shared.h"

void MainFunction()
{
    for(;;)
    {
        if (MumbleLink->Camera.IsLookingNorth())
        {
            const wchar_t* evData = "Pogey, we looked north!";

            ApiDefs.RaiseEvent(L"COMPASS_LOOKEDNORTH", (void*)evData);
        }
    }
}
```
That's all there is to raising an event, you simply call the raise function with your data passed as a void pointer. If you define events for other addons to use, make sure to document the contents of evData to avoid unnecessary crashes due to casting errors.

---

# Logging
Normally you will only use the LogA and LogW functions inside of the AddonAPI struct. All you need to know about those, is that you can use them for printf-style logging.

## Custom Logging
The AddonHost by default implements a FileLogger, ConsoleLogger, if `-ggconsole` is set, and logging to the ImGui window in game.

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
	ApiDefs.RegisterLogger(cLog);
}
```

The LogLevel of the logger determines what messages it will receive, if you wanna do your own custom filtering, just subscribe to all. You can also later set this again.

That's all you need to do for your custom logging implementation!
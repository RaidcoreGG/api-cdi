# Functions
This document explains each of the exported functions in the Raidcore Nexus API.
The current API Version is 1.

## Render
### **`GUI_ADDRENDER RegisterRender;`**  
`typedef void (*GUI_ADDRENDER) (ERenderType aRenderType, GUI_RENDER aRenderCallback);`  

Expects you to pass Render type and a render callback.
```cpp
typedef enum ERenderType
{
	ERenderType_PreRender,
	ERenderType_Render,
	ERenderType_PostRender,
	ERenderType_OptionsRender
} ERenderType;
```

These were already detailed in the Readme, but here's a quick overview again:
- `PreRender` is called *before* the dx and imgui frame is initalised.
- `Render` is called *during* the dx and imgui frame.
- `PostRender` is called *after* the dx and imgui frame was ended.
- `OptionsRender` is called *during* the dx and imgui frame and is appended to the Options window.

As for the render callback, that is simply a function like `void Render();` nothing more fancy than that.

### **`GUI_REMRENDER UnregisterRender;`**    
`typedef void (*GUI_REMRENDER) (GUI_RENDER aRenderCallback);`

Expects you to pass the same render callback that you used to register.

## Paths
### **`PATHS_GETGAMEDIR GetGameDirectory;`**
`typedef const char* (*PATHS_GETGAMEDIR)();`

Returns wherever your running Guild Wars 2 resides.
E.g. `"C:\Program Files\Guild Wars 2"`.

### **`PATHS_GETADDONDIR GetAddonDirectory;`**
`typedef const char* (*PATHS_GETADDONDIR)(const char* aName);`

Returns the addon directory of the passed parameter.  
Examples:  
> `GetAddonDir("");`  
Returns `"C:\Program Files\Guild Wars 2\addons"`

Mind the backslash missing when passing an empty string.

> `GetAddonDir("MyVeryFirstNexusAddon");`  
Returns `"C:\Program Files\Guild Wars 2\addons\MyVeryFirstNexusAddon"`

### **`PATHS_GETCOMMONDIR GetCommonDirectory;`**
`typedef const char* (*PATHS_GETCOMMONDIR)();`

Synonymous with `GetAddonDir("common");` meaning `GetCommonDir();` will always return `"C:\Program Files\Guild Wars 2\addons\common"`.

This is the folder intended for shared data, such as caches etc that do not belong to any particular addon.

## MinHook
There won't be any hooking tutorial or how to use these functions here, please read up on MinHook instead.

## Logging
### **`LOGGER_LOGA Log;`**
`typedef void (*LOGGER_LOGA)(ELogLevel aLogLevel, const char* aStr);`

Expects you to pass the log level value and a const char*.
Writing a log message appends to the in-game Log window as well as the Nexus.log file.

LogLevels are as follows:
```cpp
typedef enum ELogLevel
{
	ELogLevel_OFF         = 0,
	ELogLevel_CRITICAL    = 1,
	ELogLevel_WARNING     = 2,
	ELogLevel_INFO        = 3,
	ELogLevel_DEBUG       = 4,
	ELogLevel_TRACE       = 5,
	ELogLevel_ALL
} ELogLevel;
```

## Events

### **`EVENTS_RAISE RaiseEvent;`**
`typedef void (*EVENTS_RAISE)(const char* aIdentifier, void* aEventData);`

You can raise an event which is shared with any subscriber, by passing the identifier or name of your event and a pointer to the data.

Keep in mind, whatever data you have must be valid for the lifetime of the game.
Or you can simply use this function to notify other addons of something having changed and move the responsibility to them.
> Note:  
This API is currently being reworked in Revision 2 such as that you can share data without managing its lifetime.

Example:
`RaiseEvent("EV_WINDOWRESIZED", (void*)pV2WindowSize);`

### **`EVENTS_SUBSCRIBE SubscribeEvent;`**
`typedef void (*EVENTS_SUBSCRIBE)(const char* aIdentifier, EVENT_CONSUME aConsumeEventCallback);`

Expects you to pass the identifier or name of the event for which you want to receive notifications and the function in which you will be processing said event.

Event consume functions are defined as follows:
`typedef void (*EVENT_CONSUME)(void* aEventArgs);`


### **`EVENTS_SUBSCRIBE UnsubscribeEvent;`**
`typedef void (*EVENTS_SUBSCRIBE)(const char* aIdentifier, EVENT_CONSUME aConsumeEventCallback);` (same signature as SubscribeEvent)

Basically the same as subscribing. Declare which event and which function you want to remove.

## WndProc
### **`WNDPROC_ADDREM RegisterWndProc;`**
`typedef void (*WNDPROC_ADDREM)(WNDPROC_CALLBACK aWndProcCallback);`

Expects you to pass your WndProc function.  
WndProc is defined as follows:
`typedef UINT (*WNDPROC_CALLBACK)(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam);`

The only difference to a regular WndProc being that the return value is a UINT instead of LRESULT.

You are expected to return 0 if you consumed the current uMsg or a non-zero value if you didn't do anything with it. In this case it's being passed along to other functions, addons and the game.

### **`WNDPROC_ADDREM UnregisterWndProc;`**
`typedef void (*WNDPROC_ADDREM)(WNDPROC_CALLBACK aWndProcCallback);` (same as RegisterWndProc)

Same as registering a WndProc.
	

## Keybinds
To handle keybinds you registered you will need to define a ProcessKeybinds function or similar with the following signature:  
`typedef void (*KEYBINDS_PROCESS)(const char* aIdentifier);`

The identifier or name, is the name of the keybind you registered.
You can and should use the same function to handle different keybinds, and simply switch depending on the passed identifier.

### **`KEYBINDS_REGISTERWITHSTRING RegisterKeybindWithString;`**
`typedef void (*KEYBINDS_REGISTERWITHSTRING)(const char* aIdentifier, KEYBINDS_PROCESS aKeybindHandler, const char* aKeybind);`

The identifier is the name of the keybind which will be passed to your handle function, which is the second parameter.

Lastly the parameter aKeybind is a simple string which will be parsed as a bind.
It is pretty self explanatory, here are a few examples:
- `RegisterKeybindWithString("KB_MYKEYBIND", MyKBHandlerFunc, "CTRL+C");`
- `RegisterKeybindWithString("KB_MYKEYBIND", MyKBHandlerFunc, "ALT+SHIFT+T");`
- `RegisterKeybindWithString("KB_MYKEYBIND", MyKBHandlerFunc, "G");`
- `RegisterKeybindWithString("KB_MYKEYBIND", MyKBHandlerFunc, "(null)");`
- `RegisterKeybindWithString("KB_MYKEYBIND", MyKBHandlerFunc, "");`

> Note:  
Passing `"(null)"` or an empty string results in the keybind not being bound by default. You can do this if you don't necessarily want a default bind, but expect the user to set one if they want to do so.

### **`KEYBINDS_REGISTERWITHSTRUCT RegisterKeybindWithStruct;`**
`typedef void (*KEYBINDS_REGISTERWITHSTRUCT)(const char* aIdentifier, KEYBINDS_PROCESS aKeybindHandler, Keybind aKeybind);`

This functions differs slightly from `RegisterKeybindWithString`.  
All you have to change is the third parameter which takes a struct instead:
```cpp
typedef struct Keybind
{
	unsigned short	Key;
	bool			Alt;
	bool			Ctrl;
	bool			Shift;
} Keybind;
```

> Note:  
Key refers to a **Scan Code** not a Virtual Key.

The struct you are passing is copied, so you can just throw it at the function.

### **`KEYBINDS_UNREGISTER UnregisterKeybind;`**
`typedef void (*KEYBINDS_UNREGISTER)(const char* aIdentifier);`
	
Since each keybind can only have one handler, meaning if you have a keybind such as `KB_BOONTABLE` you probably wanna make it more distinct to your addon, such as `KB_MYFIRSTADDON_BOONTABLE` to avoid any conflict with other addons. Otherwise it can happen due to load order that they steal your keybind.

The focus here is on the end-user setting whichever they prefer.
If there are two boon tables, they should be able to pick which one opens, or if they want they can even bind both.

## DataLink
DataLinks function basically the same as MumbleLink.

### **`DATALINK_GETRESOURCE GetResource;`**
`typedef void* (*DATALINK_GETRESOURCE)(const char* aIdentifier);`

Returns the resource of the given identifier.  
Example:
> `GetResource("DL_MUMBLE_LINK");`  
Returns a pointer to a MumbleLink struct, you will have to cast accordingly.  
E.g. `MumbleLink* ml = (MumbleLink*)GetResource("DL_MUMBLE_LINK");`

This is the easiest way to get MumbleLink, without any work on your end, as this is going to process dependant and handles multiboxing.

### **`DATALINK_SHARERESOURCE ShareResource;`**
`typedef void* (*DATALINK_SHARERESOURCE)(const char* aIdentifier, size_t aResourceSize);`

If you want to share your own custom data you once again have to declare an identifier, so other addons can find it and its size.

Nexus will allocate a shared memory space for you, to which you can write.

> Note:  
If you are sharing any **functions** via DataLink, make sure that you're nulling those when your addon gets unloaded, otherwise any depending addon might call freed memory.
	
## Textures
In Nexus Textures have the following struct:
```cpp
typedef struct Texture
{
	unsigned Width;
	unsigned Height;
	ID3D11ShaderResourceView* Resource;
} Texture;
```

### **`TEXTURES_GET GetTexture;`**
`typedef Texture* (*TEXTURES_GET)(const char* aIdentifier);`

Get Texture will return either a pointer to the Texture if it exists, or a nullptr if it doesn't. You need to handle this appropriately.

### **`TEXTURES_LOADFROMFILE LoadTextureFromFile;`**
`typedef void (*TEXTURES_LOADFROMFILE)(const char* aIdentifier, const char* aFilename, TEXTURES_RECEIVECALLBACK aCallback);`

If you want to load textures from disk, you can do so with this function.
Pass its identifier, so other addons can access it as well, pass the path to the file and lastly define your callback function to receive the Texture* (or nullptr on fail).

You can declare your receive callback as follows:  
`typedef void (*TEXTURES_RECEIVECALLBACK)(const char* aIdentifier, Texture* aTexture);`
```cpp
void ReceiveTexture(const char* aIdentifier, Texture* aTexture)
{
	std::string str = aIdentifier;

	if (str == "TEX_MYCOOLIMAGE")
	{
		myCoolImage = aTexture;
	}
}
```

### **`TEXTURES_LOADFROMRESOURCE LoadTextureFromResource;`**
`typedef void (*TEXTURES_LOADFROMRESOURCE)(const char* aIdentifier, unsigned aResourceID, HMODULE aModule, TEXTURES_RECEIVECALLBACK aCallback);`

Loading from resource refers to an embedded image inside of your .DLL.
For this you once again have to pass the identifier, but this time instead of the path you will need to pass the ResourceID as taken from `resource.h` as well as a handle to your loaded .DLL.

You can usually get the handle in your DllMain function on PROCESS_ATTACH.

### **`TEXTURES_LOADFROMURL LoadTextureFromURL;`**
`typedef void (*TEXTURES_LOADFROMURL)(const char* aIdentifier, const char* aRemote, const char* aEndpoint, TEXTURES_RECEIVECALLBACK aCallback);`

Again, very similar to the other functions but you will have to pass aRemote, referring to the baseURL of your remote location and aEndpoint, referring to the file or well, endpoint.

Example using the GW2 API:
`LoadTextureFromURL("TEX_DUNGEON_ICON", "https://render.guildwars2.com", "/file/943538394A94A491C8632FBEF6203C2013443555/102478.png", ReceiveTexture)`

The full URL would be "https://render.guildwars2.com/file/943538394A94A491C8632FBEF6203C2013443555/102478.png".

![](https://render.guildwars2.com/file/943538394A94A491C8632FBEF6203C2013443555/102478.png)

## Quick Access / Shortcuts / Menu bar
This is the menu bar at the top left of your screen, by default Nexus is set to extend it, you can change its position however you like though.

Nexus distinguishes between Shortcuts and SimpleShortcuts.
Shortcuts being its own icon which will invoke a *Keybind*.
Whereas SimpleShortcuts will append to the Nexus icon when you right-click it.

### **`QUICKACCESS_ADDSHORTCUT AddShortcut;`**
`typedef void (*QUICKACCESS_ADDSHORTCUT) (const char* aIdentifier, const char* aTextureIdentifier, const char* aTextureHoverIdentifier, const char* aKeybindIdentifier, const char* aTooltipText);`

Here's a bit more going on.
- `aIdentifier` as always, the identifier/name of your shortcut.
- `aTextureIdentifier` this is the name of the icon of your shortcut.
- `aTextureHoverIdentifier` this is the name of the icon which should be displayed while hovering.
- `aKeybindIdentifier` this should map to whichever Keybind you want to invoke when the icon is clicked. Make sure you registered it beforehand.
- `aTooltipText` you can show a simple text when hovering over the icon.

> Note:  
If you need help creating icons that look native to Guild Wars 2, you can reach out to us at Raidcore, we can get you started.

### **`QUICKACCESS_REMOVE RemoveShortcut;`**
`typedef void (*QUICKACCESS_REMOVE) (const char* aIdentifier);`

Removes the shortcut icon, simply pass its identifier.

### **`QUICKACCESS_ADDSIMPLE AddSimpleShortcut;`**
`typedef void (*QUICKACCESS_ADDSIMPLE) (const char* aIdentifier, GUI_RENDER aShortcutRenderCallback);`

Since a simple shortcut merely appends to the Nexus icon's context menu, you will have to pass a Render callback here.

In this callback you can render any ImGui you like, checkboxes, text, whatever.

### **`QUICKACCESS_REMOVE RemoveSimpleShortcut;`** (same as RemoveShortcut)
`typedef void (*QUICKACCESS_REMOVE) (const char* aIdentifier);`

Same as RemoveShortcut.
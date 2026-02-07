\page keybinds Keybinding Tutorial

Nexus provides an API allowing addon developers to register keybinds without implementing a custom `WndProc` handler.  
To implement a keybind, create a callback that handles routing your keybind events to your own custom handlers and register it with Nexus.

### Registering Keybinds
There are two methods of registering a keybind in nexus, which differ by how they represent the default binding.  
`AddonAPI_t::InputBinds_RegisterWithString` accepts a string to define the default binding.  
`AddonAPI_t::InputBinds_RegisterWithStruct` accepts a `Keybind_t` to define the default binding.  
```cpp
#define KEYBIND_STRING "KB_MYKEYBIND_STRING"
#define KEYBIND_STRUCT "KB_MYKEYBIND_STRUCT" 
void RegisterKeybinds() {
	Addon_API->InputBinds_RegisterWithString(
		KEYBIND_STRING,
		*ProcessKeybind,
		"ALT+SHIFT+T"
	);

	Keybind_t kbDef = {
		0x55, //Keybind_t.Key, 0x55 = U
		true, //Keybind_t.Alt
		false, //Keybind_t.Ctrl
		true, //Keybind_t.Shift
	};
	Addon_API->InputBinds_RegisterWithStruct(
		KEYBIND_STRUCT,
		*ProcessKeybind,
		kbDef
	);
}
```

### Deregistering Keybinds
To deregister a keybind, simply pass its `aIdentifier` to `AddonAPI_t::InputBinds_Deregister`. 

### Handling Keybinds
```cpp
void ProcessKeybind(const char* aIdentifier, bool aIsRelease) {
	if (strcmp(aIdentifier, "KB_MYKEYBIND_STRING") == 0 && !isRelease) {
		//Handle that key event
	}
	else if (strcmp(aIdentifier, "KB_MYKEYBIND_STRUCT") == 0 && !isRelease) {
		//Handle another key event
	}
}
```

## Putting it all together
Below is an example of how you would combine the previous three steps into a working keybind handler.

### `Keybinds.hpp`
```cpp
#pragma once
namespace Keybinds {
	const char* MyKeybind = "KB_EXAMPLEADDON_DOSOMETHING";
	void Register();
	void Cleanup();
	void Handle(const char*, bool);
}
```
### `Keybinds.cpp`
```cpp
#include "Keybinds.hpp"
void Keybinds::Register() {
	Addon_API->InputBinds_RegisterWithString(
		Keybinds::MyKeybind,
		Keybinds::Handle,
		"CTRL+SHIFT+S"
	)
}
void Keybinds::Handle(const char* aIdentifier, bool aIsRelease) {
	if (strcmp(aIdentifier, Keybinds::MyKeybind) == 0 && !aIsRelease) {
		Addon_API->Log(LOGL_INFO, "example_addon", "MyKeybind was pressed");
	}
}
void Keybinds::Cleanup() {
	Addon_API->InputBinds_Deregister(Keybinds::MyKeybind);
}
```

### `ModuleMain.cpp`
```cpp
#include "Keybinds.hpp"
// ...
void AddonLoad(AddonAPI_t* API) {
	// ... see the quickstart for more info
	Keybinds::Register();
}
void AddonUnload() {
	Keybinds::Cleanup();
}
// ...
```

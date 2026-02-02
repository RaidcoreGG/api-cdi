\page Events Event Handling Tutorial

Nexus implements an event system, allowing you to subscribe to both Nexus's internal events, as well as user-defined events from both your own addon and other loaded addons.

## Subscribing to an event
Subscribing to an event is as simple as defining a callback and passing it to the AddonAPI.
### `EventHandlers.hpp`
```cpp
#pragma once
namespace EventHandlers {
	void MumbleIdentityUpdate(void*);
	void Register();
	void Cleanup();
}
```
### `EventHandlers.cpp`
```cpp
#include "shared.h"
void EventHandlers::MumbleIdentityUpdate(void* eventArgs) {
	// eventArgs for EV_MUMBLE_IDENTITY_UPDATED is Mumble::Identity
	Mumble::Identity* id = static_cast<Mumble::Identity*>(eventArgs);
	char buffer[50];
	sprintf(buffer, (size_t)50, "Changed character to %s\n", id->Name);
	Addon_API->Log(LOGL_INFO, "example-addon", buffer);
}

void EventHandlers::Register() {
	// EV_MUMBLE_IDENTITY_UPDATED is called every time your identity changes, e.g. commanding a squad or changing characters.
	Addon_API->Events_Subscribe("EV_MUMBLE_IDENTITY_UPDATED", *EventHandlers::MumbleIdentityUpdate);
}
void EventHandlers::Cleanup() {
	Addon_API->Events_Unsubscribe("EV_MUMBLE_IDENTITY_UPDATED", *EventHandlers::MumbleIdentityUpdate);
}
```
### `ModuleMain.cpp`
```cpp
#include "EventHandlers.hpp"
void AddonLoad(AddonAPI_t* API) {
	// ... see the quickstart for details.
	EventHandlers::Register();
}
void AddonUnload() {
	// ... See the quickstart for details.
	EventHandlers::Cleanup();
}
// ...
```

## Raising Events
Nexus also provides an API for raising events to be consumed either by your own code, or by other developers.
An example of raising an event is provided below.
```cpp
// ...
#define EXAMPLE_EVENT_IDENTIFIER "EV_EXAMPLEADDON_EXAMPLE"
void RaiseExampleEvent() {
	const wchar_t* message = "This is an example event";
	Addon_API->Events_Raise(EXAMPLE_EVENT_IDENTIFIER, (void*)message);
	Addon_API->Events_RaiseNotification(EXAMPLE_EVENT_IDENTIFIER);
}
```
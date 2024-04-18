# Events
This document explains each of the available events in the Raidcore Nexus API.

## Nexus
### `EV_WINDOW_RESIZED`
No payload. (Payload is `nullptr`).  
Check the `DL_NEXUS_LINK` shared resource for the width &amp; height of the window.

### `EV_MUMBLE_IDENTITY_UPDATED`
Payload is `MumbleIdentity*`.  
This pointer stays valid and does not change, so it can be set once and subsequent calls can be used as a notification/trigger to update some derived values or similar.  
Definitions can be found below or in the public [repository](https://github.com/RaidcoreGG/RCGG-lib-mumble-api).
```cpp
struct Identity
{
	char		Name[20];
	unsigned	Profession;
	unsigned	Specialization;
	unsigned	Race;
	unsigned	MapID;
	unsigned	WorldID;
	unsigned	TeamColorID;
	bool		IsCommander;
	float		FOV;
	unsigned	UISize;
};
```

### `EV_ADDON_LOADED`
Payload is `int*` (signed) of the addon that was loaded.

### `EV_ADDON_UNLOADED`
Payload is `int*` (signed) of the addon that was unloaded.

### `EV_VOLATILE_ADDON_DISABLED`
Payload is `int*` (signed) of the volatile addon that was disabled because of a game update.

## ArcDPS
Nexus relays ArcDPS local &amp; squad combat callbacks as events.  
All types are defined by ArcDPS, refer to the definitions in the [evtc](https://www.deltaconnected.com/arcdps/evtc/README.txt) and [api](https://www.deltaconnected.com/arcdps/api/README.txt) documentation.

### `EV_ARCDPS_COMBATEVENT_LOCAL_RAW`
Payload is `EvCombatData*`.
```cpp
struct EvCombatData
{
	cbtevent* ev;
	ag* src;
	ag* dst;
	char* skillname;
	uint64_t id;
	uint64_t revision;
};
```

### `EV_ARCDPS_COMBATEVENT_SQUAD_RAW`
Payload is `EvCombatData*`. Refer to definition above.

## Unofficial Extras
Nexus relays Unofficial Extras callbacks as events.  
All types are defined by Unofficial Extras, refer to the definitions in [their repository](https://github.com/Krappa322/arcdps_unofficial_extras_releases).  
The only exception is `SquadUpdate`, which just wraps two parameters into a struct. The type is defined below.

### `EV_UNOFFICIAL_EXTRAS_SQUAD_UPDATE`
Payload is `SquadUpdate*`.
```cpp
struct SquadUpdate
{
	UserInfo* UserInfo;
	uint64_t UsersCount;
};
```

### `EV_UNOFFICIAL_EXTRAS_LANGUAGE_CHANGED`
Payload is `Language*`.

### `EV_UNOFFICIAL_EXTRAS_KEYBIND_CHANGED`
Payload is `KeyBindChanged*`.

### `EV_UNOFFICIAL_EXTRAS_CHAT_MESSAGE`
Payload is `ChatMessageInfo*`.

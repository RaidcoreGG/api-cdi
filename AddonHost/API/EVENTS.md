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
Payload is `EvCombatData*`. Refer to definition above. These events have a 2 second delay.

### `EV_ARCDPS_SELF_JOIN`
Payload is `EvAgentUpdate*` of the self player agent. Events of this type are triggered upon map load. The last event can be retriggered on demand by addons sending an `EV_REPLAY_ARCDPS_SELF_JOIN` event with an empty payload.

```cpp
struct EvAgentUpdate				// when ev is null
{
	char account[64];		// dst->name	= account name
	char character[64];		// src->name	= character name
	uintptr_t id;			// src->id		= agent id
	uintptr_t instanceId;	// dst->id		= instance id (per map)
	uint32_t added;			// src->prof	= is new agent
	uint32_t target;		// src->elite	= is new targeted agent
	uint32_t Self;			// dst->Self	= is Self
	uint32_t prof;			// dst->prof	= profession / core spec
	uint32_t elite;			// dst->elite	= elite spec
	uint16_t team;			// src->team	= team
	uint16_t subgroup;		// dst->team	= subgroup
};
```

### `EV_ARCDPS_SELF_LEAVE`
Payload is `EvAgentUpdate*` of the self player agent. Refer to definition above. Events of this type are triggered when changing instance or leaving a party / squad.

### `EV_ARCDPS_SQUAD_JOIN`
Payload is `EvAgentUpdate*` of an allied player agent. Refer to definition above. Events of this type are triggered when allied players in your instance join your party / squad or when allied players in your party / squad join your instance. These events have a 2 second delay.

Nexus tracks all players in your squad and can retrigger these events on demand by addons sending an `EV_REPLAY_ARCDPS_SQUAD_JOIN` event with an empty payload. This is intended to be used during addon load, you should be careful to handle duplicates since this can be triggered by other addons. 

### `EV_ARCDPS_SQUAD_LEAVE`
Payload is `EvAgentUpdate*` of an allied player agent. Refer to definition above. Events of this type are triggered when allied players in your instance and party / squad either leave your instance or leave your party / squad. You will not recieve these events if you are the one to change instance or leave the party / squad. These events have a 2 second delay. 

### `EV_ARCDPS_TARGET_CHANGED`
Payload is `EvAgentUpdate*` of the new target. Refer to definition above. Events of this type are triggered when you target an agent. The last event can be retriggered on demand by addons sending an `EV_REPLAY_ARCDPS_TARGET_CHANGED` event with an empty payload.

### `EV_ACCOUNT_NAME`
Payload is `const char*` of the self player account name. Events of this type are triggered upon first map load. This event can also be triggered on demand by addons sending an `EV_REQUEST_ACCOUNT_NAME` event with an empty payload.

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

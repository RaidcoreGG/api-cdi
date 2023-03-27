# Addon Host - Nexus
[![](https://img.shields.io/discord/410828272679518241.svg?logo=discord&logoColor=ffffff&style=for-the-badge&color=blue)](https://discord.gg/Mvk7W7gjE4)

**Nexus is an Addon Host to simplify installing, loading and updating addons. No fidgeting with DLL files and removing after the game updates and the addons break.**

**For developers:** Easy to implement, easy to use. No need to reinvent the wheel.\
*More information in the API section.*

# Features
- **Loading & unloading** addons **at runtime**.
- **Automatic updates**.
- **Addon Library** for quick discovery & installation of new addons.
- **Globally handled keybinds**.
- Globally handled **localisation**.
- **Multiboxing** Compatibility.
- **100% ToS friendly**

---

# Installation
1. Download `d3d11.dll` found in the [latest release](https://github.com/RaidcoreGG/GW2-Nexus-Releases).
2. Place the file in your game installation directory (e.g. `C:\Program Files\Guild Wars 2`). In that same folder you should see `Gw2-64.exe`.
3. Start the game, you should see the menu pop up.

---

# API

## Feature Overview
- **Hot-Loading** in combination with **automatic updates**.
- **Event Publishing / Subscribing**.
- **Keybinds handled** by the Addon Host. No need to fiddle with WndProc. *Unless you want to*.
- **Logging**, to file, to window, or custom implementation.
- Resource Registry, to **share resources & functions** between addons.
- **Texture/Icon Loader**, lifting all the heavy work for you. Just embed your resource or load from disk.
- Easily access [Mumble](https://github.com/RaidcoreGG/RCGG-lib-mumble-api), **combat log**, world / **map completion** progress & **character stats**.
- **Shared GW2 API & cache**, easily request data from the [official Guild Wars 2 API](https://api.guildwars2.com/v2).

## API Definitions
You can find the definitions for the API [here](./API/Definitions) or add them as a submodule from its [repository](https://github.com/RaidcoreGG/RCGG-lib-nexus-api).

## Example code
You can find an example addon [here](https://github.com/RaidcoreGG/GW2-Compass). Which adds a simple compass to Guild Wars 2.

---

# Open Source
The Addon Host is not open source for a few reasons. Mainly because my code has been stolen and then even sold before, I don't want that to happen again.

If you're concerned about anything malicious going on, feel free to decompile and disassemble it, I don't obfuscate in any way and use the standard compiler of Visual Studio.

I will however give a select number of trustworthy community developers as well as any ArenaNet employee, access upon request.

*I may open-source the Addon Host in the future, but that heavily depends on it getting popular enough to ~~give me bragging rights~~ allow me to defend my creation in case it gets stolen again.*

---

# Credits
<!--- [GGDM](https://nkga.github.io/post/ggdm---combat-analysis-mod-for-guild-wars-2/): For the idea of a proxy dll, hot-loading & invaluable help with hooking related questions. -->
- [ArcDPS](https://www.deltaconnected.com/arcdps/): For the idea of an addon loading system.
- Crafty.1372: For enduring the absolute lunacy that my questions to him are.
- Greaka.6905: For giving me a comprehensible list of why the alternative addon host solution is terrible, as I never tried it and never will.
- [Dear ImGui](): For the UI Framework and undescribable pain & suffering.
- [nlohmann::json](): For the JSON Framework and undescribable pain & suffering.
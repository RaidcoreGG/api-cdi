# Addon Host - Nexus
[![](https://img.shields.io/discord/410828272679518241.svg?logo=discord&logoColor=ffffff&style=for-the-badge&color=blue)](https://discord.gg/Mvk7W7gjE4)

**Nexus is an Addon Host to simplify installing, loading and updating addons. No fidgeting with DLL files, crashing games after updates or inconsistent behaviour across addons.**

**For developers:** Easy to implement, easy to use. No need to reinvent the wheel.\
*More information in the API section.*

# Features
- **Loading & unloading** addons **at runtime, without restarting**.
- **Automatic updates**.
- **Addon Library** for quick discovery & installation of new addons.
- **Globally handled keybinds**.
- **Multiboxing** compatibile.
- **100% ToS compliant.**

---

# Installation
1. Download `d3d11.dll` found in the [latest release](https://github.com/RaidcoreGG/GW2-Nexus-Releases).
2. Place the file in your game installation directory (e.g. `C:\Program Files\Guild Wars 2`). In that same folder you should see `Gw2-64.exe`.
3. Start the game, you should have a new icon in the top left, or try opening the menu using "CTRL+M" by default.

---

# API

## Feature Overview
- **Hot-Loading**
- **Automatic Updates** (For Addons & Nexus)
- **Event Publishing / Subscribing**
- **Keybinds handled** by the Addon Host. No need to fiddle with WndProc. *Unless you want to*.
- **Logging** (To File/Window or custom implementation.)
- **Resource Registry** to share resources & functions between Addons.
- **Texture Loader**, lifting all the heavy work for you. Just embed your resource or load from disk.
- Easily access [Mumble](https://github.com/RaidcoreGG/RCGG-lib-mumble-api), **combat log**, world / **map completion** progress & **character stats**.
- **Shared GW2 API & cache**, easily request data from the [official Guild Wars 2 API](https://api.guildwars2.com/v2).

## API Definitions
You can find the definitions for the API [here](./API/Definitions) or add them as a submodule from its [repository](https://github.com/RaidcoreGG/RCGG-lib-nexus-api).

## Example code
You can find an example addon [here](https://github.com/RaidcoreGG/GW2-Compass). Which adds a simple compass to Guild Wars 2.

---

# Source Code
The **Addon Host** also known as **Raidcore Nexus**, will remain closed source for now. There might be a partial release of non critical internals in the future.

As for any concerns, feel free to decompile and disassemble the .DLL. It is not obfuscated in any way and the standard Visual Studio compiler is used.

> **If you are an ArenaNet employee and would like to get access to the source code, you can contact us [here](mailto:contact@raidcore.gg?subject=Nexus%20Source%20Code%20Request).**

---

# Credits
<!-- [GGDM](https://nkga.github.io/post/ggdm---combat-analysis-mod-for-guild-wars-2/): For the idea of a proxy dll & hot-loading. -->
- [Deltaconnected / ArcDPS](https://www.deltaconnected.com/arcdps/): For the idea of an addon loading system.
- [Thomas McBoyle](https://github.com/TMcBoyle): For general guidance & programming help.
- [Jakub Vitek](https://github.com/Sognus): Invaluable help with debugging & general guidance.
<!-- [Dear ImGui](): For the UI Framework and undescribable pain & suffering. (Though I learned to love ImGui.)
- [nlohmann::json](): For the JSON Framework and undescribable pain & suffering. -->
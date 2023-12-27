# Game Engines Integration

Thanks to the fact that the GOG GALAXY SDK is available as C++ libraries, C# wrapper and a Unity package (based on the C# wrapper), GOG GALAXY features may be easily implemented in projects created with the majority of popular game engines.

In the table below you will find some additional information on popular game engines — it may be helpful during the planning process of GOG GALAXY features implementation:

| Engine              | Scripting Languages                  | Comment                                                      |
| ------------------- | ------------------------------------ | ------------------------------------------------------------ |
| Unity3D             | C#                                   | Native support: the GOG GALAXY SDK available as a script package asset for importing to a Unity project |
| Unreal Engine 4     | C++, Blueprints                      | Native support: a plugin for the GOG GALAXY SDK available on our [GitHub repository](https://github.com/gogcom/galaxy-ue4-oss-plugin) |
| CRYENGINE           | C++, C#                              | Native support                                               |
| Game Maker Studio 2 | GameMaker Language (GML)             | An open source wrapper for the GOG GALAXY SDK v1.113.3 created by Vadim Dyachenko (YellowAfterlife) available on his [GitHub repository](https://github.com/GameMakerDiscord/GOG.gml) |
| RPG Maker           | Javascript                           | No support                                                   |
| Amazon Lumberyard   | Lua                                  | C++ components can be added using [“gems”](https://docs.aws.amazon.com/lumberyard/latest/userguide/component-entity-system-pg-gems-code.html) |
| Godot               | GDScript, NativeScript,Visual Script | C++ components can be added using [“modules”](http://docs.godotengine.org/en/latest/development/cpp/custom_modules_in_cpp.html) |
| Visionaire Studio   | Lua                                  | Official support: see the [`getProperty`](https://wiki.visionaire-tracker.net/wiki/GetProperty) command and commands containing *GameClient* in the [Visionaire Studio documentation](https://wiki.visionaire-tracker.net/wiki/Player_Commands) |
| Defold              | Lua                                  | An open source GOG GALAXY SDK extension created by Marius Petcu (dapetcu21) available on his [GitHub repository](https://github.com/dapetcu21/defold-gog-galaxy). This extension allows developers to implement achievements into Defold-based games |
| Ren'Py              | Python                               | A small ctypes library and a python helper class created by Nur Endah Safitri is available on [VifthFloor Public Repository](https://gitlab.com/vifthfloor-public/renpy-gog/-/tree/master?ref_type=heads) |

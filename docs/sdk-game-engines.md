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
| Amazon Lumberyard   | Lua                                  | C++ components can be added using [“gems”](https://docs.aws.amazon.com/lumberyard/latest/userguide/component-entity-system-pg-gems-code.htmlGodotGDScript) |
| Godot               | GDScript, NativeScript,Visual Script | C++ components can be added using [“modules”](http://docs.godotengine.org/en/latest/development/cpp/custom_modules_in_cpp.html) |
| Visionaire Studio   | Lua                                  | Official GOG GALAXY SDK support: see commands containing *GameClient* |


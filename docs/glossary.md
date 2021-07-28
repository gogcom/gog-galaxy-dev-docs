# Glossary

## CDN (Content Delivery Network)

A content delivery network or content distribution network is a geographically distributed network of proxy servers and their data centers. The goal is to provide high availability and high performance by distributing the service spatially relative to end-users.

## Crossplay

The possibility for players to interact with each other despite using different gaming or hardware platforms, e.g.: GOG users playing with Steam users on Windows PCs, or with PlayStation 4 or Xbox One users on consoles. For more information, please see [*Crossplay*](sdk-crossplay.md).

## Depot

A folder that contains the core game data and assets required to run a game, such as executables or textures. In fact, a depot is just a directory/folder on a hard drive. For more information, please see [*Depots*](bc-depots.md).

## DLC

Downloadable content (DLC) is a set of extensions to the base game such as new maps, levels, etc. For more information, please see [*DLCs and Extras*](dlc-and-extras.md).

## Extras

Additional materials to a game such as audio tracks, videos, manuals and so on. On GOG, they can be sold independently from the game and don’t require its presence. Lastly, they don’t need to be added to the game build in the Developer Portal. For more information, please see [*DLCs and Extras*](dlc-and-extras.md).

## Overlay

The GOG GALAXY Overlay is an on-screen menu that can be displayed in a game after pressing **Shift+Tab**. It allows players to use multiplayer features while playing a game (e.g.: chatting or joining a multiplayer game, taking screenshots, displaying the FPS counter and more). For more information, please see [*GOG GALAXY Overlay*](gc-overlay.md).

## Package

Packages define which depots and tasks should be put together to create a working game build. Each package should contain all of the depots and tasks required to launch a game in the specified configuration (a language chosen by a user and the architecture of their OS). For more information, please see [*Packages*](bc-packages.md).

## Release

The process of making a Release Candidate build publicly available.

## Release Candidate

A final version of a game that is fully functional, but not quite ready to be released to the consumer market. It must be verified by the GOG QA team for regressions and any potential issues before it can be released to the public.

## Rollback

The rollback feature is unique to the GOG GALAXY client, not available in other game platforms, which allows a user to select and play a previous version of a game in case the latest update causes problems. For more information, see [*Rollback*](gc-rollback.md).

## SDK (Software Development Kit)

A set of tools used for developing applications.

## Staging Branch

A branch for developers to test the stability of an uploaded build in the GOG GALAXY client without publishing it to the Master branch. For more information, please see [*Build Branches*](build-branches.md).

## Task

A task points to an executable (a launcher or a core game *.exe* file) or to an URL address. Tasks are used to launch and interact with the game in the GOG GALAXY client as well as through Start Menu / Desktop shortcuts in Windows. For more information, please see [*File Tasks*](bc-file-tasks.md) and [*URL Task*](bc-url-task.md).
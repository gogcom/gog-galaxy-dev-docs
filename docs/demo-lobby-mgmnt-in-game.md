# Lobby Management In Game

## Description

The “lobby management in game” methods handle lobby operations while the player is in an online multiplayer match. These methods are chiefly listeners for events such as leaving the lobby, changing lobby data and changing the state of lobby members. Separating the lobby management in game from the one in menu allows implementing different behaviors in response to the same actions.

## Initialization and Termination

The **LobbyManagementInGame** listeners are initialized by the `LobbyDataListenerMainMenu.OnLobbyDataUpdated()` method described in the [previous implementation example](demo-lobby-mgmnt-main-menu-implementation.md#lobbydatalistenermainmenuonlobbydataupdated-callback). This method included in the **Matchmaking** script (available at *Assets/Scripts/GalaxyManager/Features/Matchmaking.cs*) calls the `LobbyManagmentInGameListenersInit` method, which in turn initializes the `LobbyDataListenerInGame()`, `LobbyLeftListenerInGame()` and `LobbyMemberStateListenerInGame()` listeners.

When the player exits the match (either by quitting to the main menu or when the host leaves the lobby), all listeners are disposed of.

## Definitions of Listeners

### LobbyLeftListenerInGame

This listener receives callbacks when the user succeeds or fails to leave a lobby. It provides the GalaxyID of the lobby the player left. Regardless of the result, this listener provides both the `lobbyID` it was trying to leave and the result of calling the `OnLobbyLeft()` method.

In this demo we use lobby topology of the FCM type without ownership transition (*LOBBY_TOPOLOGY_TYPE_FCM*). Because of this, **LobbyLeftListenerInGame()** receives a callback in two situations:

- when the player (*client*) leaves the lobby,
- when the lobby host (*owner*) leaves.

### LobbyDataListenerInGame

This listener receives callbacks when a given lobby or specific lobby member data is retrieved. It provides the GalaxyID of the lobby and/or the lobby member whose data was changed. If only the lobby data was changed, the member’s GalaxyID will be empty and we can check this using the `GalaxyID.IsValid` method.

### LobbyMemberStateListenerInGame

It listens for the event of changing the state of a lobby member. It provides GalaxyID of the lobby and GalaxyID of the lobby member whose state was changed, along with a specific **LobbyMemberStateChange**.
# Lobby Management in Main Menu

## Description

The “lobby management in main menu” methods handle lobby operations when the player is in the lobby menu. These methods are chiefly listeners for leaving the lobby and changing lobby data events. Separating the lobby management in the menu from the one in the game allows implementing different behaviors in response to the same actions.

## Initialization and Termination

Listeners for **LobbyManagementMainMenu** are initialized when the user successfully joins the lobby (on the **LobbyEnteredListenerBrowsing.OnLobbyEntered** callback), or creates a lobby (on the **LobbyCreatedListener.OnLobbyCreated** callback).

The listeners are disposed of when a game is started (on the **LobbyDataListenerMainMenu.OnLobbyDataUpdated()** callback) or when the player leaves the lobby (on the **LobbyLeftListenerMainMenu.OnLobbyLeft()** callback).

## Definitions of Listeners

### LobbyLeftListenerMainMenu

This listener receives callbacks when the user succeeds or fails to leave a lobby. It provides the GalaxyID of the lobby the player left. Regardless of the result, this listener provides both the `lobbyID` it was trying to leave and the result of calling the `Matchmaking.LeaveLobby()` method.

In this demo we use lobby topology of the FCM type without ownership transition (*LOBBY_TOPOLOGY_TYPE_FCM*). Because of this, **LobbyLeftListenerMainMenu()** receives a callback in two situations:

- when the player (*client*) leaves the lobby,
- when the lobby host (*owner*) leaves.

Inside this listener we implemented three additional methods:

- `HostClosedLobby()` for notifying the player-client that the lobby host left (and the lobby was closed), and for getting the player-client back to the online menu,
- `UserLeftLobby()` for getting the player-client back to the online menu,
- `LobbyLeft()` to restore default (null) values of the following variables: `CurrentLobbyID`, `LobbyOwnerID` and `lobbyName`, and to dispose of the **LobbyManagementMainMenu** listeners.

### LobbyDataListenerMainMenu

This listener receives callbacks when a given lobby or specific member data is retrieved. It provides the GalaxyID of the lobby and the member whose data was changed. If only the lobby data was changed, the member’s GalaxyID will be empty (`new GalaxyID(0)`).

This listener is used very often: first, we use it in the **OnlineWait** screen of the main menu. When a member joins a lobby, their data `state` is set to `notReady`. When this lobby member clicks the *Ready* button, their `state` is set to `ready`. When both members’ states are set to `ready`, the lobby state is set to `ready`, which in turn activates the *Start game* button for the lobby owner.

When the lobby owner starts the game, they set the lobby `state` to `steady`. When the players receive information about this state change, they begin loading the new scene. After the loading is completed, each member sets their own `state` to `go`. When both players’ `state` is set to `go`, the game starts.

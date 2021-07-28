# Lobby Browsing

## Description

The demo game provides a set of methods responsible for displaying a list of available multiplayer lobbies in the *Server Browser* menu (found in the *Online* section of the main menu of the game). This menu also allows the player to join one of the existing lobbies (provided it is not full).

## Initialization and Termination

Lobby browsing methods are contained within the **Matchmaking** script available at *Assets/Scripts/GalaxyManager/Features/Matchmaking.cs*. They are enabled by **OnlineBrowserController** (available at *Assets/Scripts/UI/MainMenu/OnlineBrowserController.cs*), when a player opens **OnlineBrowserScreen** by entering *Online→Server Browser* menu. When enabled, **LobbyListListenerBrowsing()**, **LobbyDataRetrieveListenerBrowsing()** and **LobbyEnteredListenerBrowsing()** are initialized.

The controller is disabled when the player closes *Server Browser* menu. All listeners listed above are disposed of when **Matchmaking** is closed.

## Definitions of Listeners

### LobbyListListenerBrowsing

We use this listener to retrieve data of all available lobbies, so that we can display the actual names of those lobbies.

This listener receives callbacks when a list of all lobbies for the **clientID** of the game is retrieved with `Matchmaking.RequestLobbyList()`. After the lobby list is successfully retrieved, the listener provides the total count of available lobbies (excluding private lobbies).

### LobbyEnteredListenerBrowsing

We use this listener to set the user’s default data and to retrieve the lobby and lobby owner’s GalaxyID when a lobby is entered.

This listener receives callbacks when a user succeeds or fails to enter a lobby. Regardless of the result, it provides the GalaxyID of the lobby the user was trying to join, as well as the result itself (if the lobby was joined or not).

### LobbyDataRetrieveListenerBrowsing

This listener receives callbacks when a specific lobby data is retrieved. It provides the GalaxyID of the lobby.
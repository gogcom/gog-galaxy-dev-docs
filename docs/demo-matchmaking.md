# Matchmaking

## Description

**Matchmaking** handles lobbies and lobby members. It is initialized when a player enters the *Online* section in the main menu of the game, and it remains present for the entire duration of the online game session. It is responsible for lobby-related operations during a multiplayer game, such as lobby creation, chat and handling the lobby view in UI.

The **Matchmaking** script is available at *Assets/Scripts/GalaxyManager/Features/Matchmaking.cs*.

The variables of the Matchmaking class store the `lobbyID` of the current lobby, the `lobbyName` and the `userID` of the lobby owner.

The GOG GALAXY SDK [`Matchmaking()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html) methods enable operations on multiplayer lobbies, like creating, listing, joining and leaving a lobby, as well as changing and getting lobby data.

!!! Info
    A lobby has to be set as joinable for the players to be able to enter it. In this demo, the maximum number of players per lobby is 2, and a multiplayer game can only be started with a full lobby.

## Initialization

This script is added as a component to the **GalaxyManager** GameObject, when the player opens the menu for an online multiplayer game. This happens in the `OnlineController.OnEnable()` method from the **OnlineController** script available at *Assets/Scripts/UI/MainMenu/OnlineController.cs*:

```c#
    void OnEnable()
    {
        GalaxyManager.Instance.StartMatchmaking();
    }
```

## Termination

The **Matchmaking** script is disabled when the player returns to the main menu. This is done by the `MainController.OnEnable()` method from the **MainController** script available at *Assets/Scripts/UI/MainMenu/MainController.cs*:

```c#
	void OnEnable () 
	{
		GalaxyManager.Instance.ShutdownMatchmaking();
	}
```

When this class is terminated, it ensures that all of its related classes are closed by calling the `ShutdownAllMatchmakingClasses()` method in the `OnDisable()` method:

```c#
    void OnDisable()
    {
        // Make sure that all listeners are properly disposed when Matchmaking is closed.
        LobbyBrowsingListenersDispose();
        LobbyCreationListenersDispose();
        LobbyChatListenersDispose();
        LobbyManagmentMainMenuListenersDispose();
        LobbyManagmentInGameDispose();
    }
```


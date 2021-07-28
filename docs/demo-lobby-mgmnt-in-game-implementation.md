# Lobby Management In Game: Examples of Implementation

## Managing Lobby Members in an Online Game

### User Experience

Once the online game starts, the player awaits the confirmation that both players are ready to start the game. As soon as they are, the game starts automatically.

If any player leaves the game during gameplay, the game stops. The other player is informed that the opponent has left the game, and is given an option to return to the main menu.

### Solution

Based on callbacks from `Matchmaking.LobbyLeftListenerInGame()` and `Matchmaking.LobbyMemberStateListenerInGame()`, we display a message informing that the other player has left the lobby.

### Methods and Usage

#### LobbyDataListenerInGame.OnLobbyDataUpdated Callback

```c#
    public override void OnLobbyDataUpdated(GalaxyID lobbyID, GalaxyID memberID)
    {
        Debug.Log("LobbyID: " + lobbyID + "\nMemberID: " + memberID);
        if (memberID.IsValid())
        {
            if (AllMembersGo(lobbyID) && matchmaking.IsCurrentUserLobbyOwner())
            {
                matchmaking.SetLobbyData(lobbyID, "state", "go");
            }
            return;
        }
        if (matchmaking.GetLobbyData(lobbyID, "state") == "go")
        {
            GameObject.Find("GameManager").GetComponent<Online2PlayerGameManager>().enabled = true;
            GameObject.Find("PopUps").GetComponent<PopUps>().ClosePopUps();
        }
    }
```

This callback is triggered whenever lobby data or lobby member data was changed. If the lobby data was changed, then the `memberID` will not be a valid GalaxyID. We use this to check if the callback was triggered by lobby data or lobby member data change by calling the `GalaxyID.IsValid` method on the `memberID`. We set the lobby data and the lobby member data to synchronize both players and make sure that they are ready to start the match. In more detail, the algorithm from here onwards is as follows:

1. At this point, the lobby data and current user’s lobby member data under the `state` key should be set to `steady`.
2. Once the **Online2PlayerGame** scene is loaded and all preparations are done, then the users’ `state` will be set to `go` by the `Online2PlayerGameManager.Awake` method.
3. When the above change is detected and the **LobbyDataListenerInGame.OnLobbyDataUpdated** callback is fired, we will:

    - Check if the memberID is a valid GalaxyID. Because the change from the step #2 above was made to the lobby member, not the lobby itself, the memberID will be a valid GalaxyID.
    - Check if both users’ `state` has been set to `go`, using the `AllMembersGo()` method.
    - Use the `return` keyword to stop further execution of this callback.

4. If both lobby members’ data under the `state` key was set to `go`, then we will set the lobby data under the `state` key to `go` as well.
5. This will again fire the **LobbyDataListenerInGame.OnLobbyDataUpdated** callback.
6. Because the change from the previous step was made to the lobby data, the `memberID` will not be a valid GalaxyID, and the code within the first `if` block will not be executed. Thanks to that, we will go straight to the code that follows the skipped block, where we’ll check if the `state` of the lobby was set to `go`.
7. Since the lobby `state` was set to `go`, we’ll start the game by enabling the **Online2PlayerGame** class and close the pop-ups.

#### LobbyDataListenerInGame.AllMembersGo

```c#
    bool AllMembersGo(GalaxyID lobbyID)
    {
        uint lobbyMembersCount = matchmaking.GetNumLobbyMembers(lobbyID);
        if (lobbyMembersCount < 2)
            return false;
        for (uint i = 0; i < lobbyMembersCount; i++)
        {
            if (matchmaking.GetLobbyMemberData(matchmaking.GetLobbyMemberByIndex(lobbyID, i), "state") != "go")
                return false;
        }
        return true;
    }
```

The `AllMembersGo()` method is used to check whether a lobby has at least two players, and both are ready to start the match. It will return `false` if there are less than 2 players in the lobby, or at least one of the lobby member’s data under the `state` key is not equal to `go`. Otherwise, it will return `true`.

#### LobbyLeftListenerInGame.OnLobbyLeft Callback

```c#
        public override void OnLobbyLeft(GalaxyID lobbyID, LobbyLeaveReason leaveReason)
        {
            if (leaveReason != LobbyLeaveReason.LOBBY_LEAVE_REASON_USER_LEFT)
                GameObject.Find("PopUps").GetComponent<PopUps>().PopUpWithLoadSceneButton("Connection to lobby lost\nReason: " +
                    leaveReason, "Back");
            matchmaking.CurrentLobbyID = null;
            matchmaking.LobbyOwnerID = null;
            GameManager.Instance.GameFinished = true;
            GalaxyManager.Instance.ShutdownNetworking();
            matchmaking.LobbyManagmentInGameListenersDispose();
            matchmaking.LobbyChatListenersDispose();
        }
```

This callback will be triggered whenever the current user leaves a lobby. This can happen in three cases:

- The current user leaves the lobby on their own. This will happen when we call the `Matchmaking.LeaveLobby()` method.
- The current user is not the lobby owner, and the lobby owner leaves the lobby. Although this might seem unusual, it happens, because when the lobby owner leaves a lobby, then the said lobby will be closed, causing the current user to leave it as well.
- Connection to the lobby is lost.

In the last two cases, a proper notification about this event is displayed on the screen along with an option to quit to the main menu.

Regardless of the reason of leaving the lobby, the `currentLobbyID` and `lobbyOwnerID` variables storing information about the lobby are reset to their defaults (null), the `GameFinished` flag is set to `true`, the **Networking** script is shut down and, finally, all **LobbyManagmentInGame** and **LobbyChat** listeners are disposed of.

#### LobbyMemberStateListenerInGame.OnLobbyMemberStateChanged Callback

```c#
        public override void OnLobbyMemberStateChanged(GalaxyID lobbyID, GalaxyID memberID, LobbyMemberStateChange memberStateChange)
        {
            Debug.Log("OnLobbyMemberStateChanged lobbyID: " + lobbyID + " memberID: " + memberID + " change: " + memberStateChange);
            if (memberStateChange != LobbyMemberStateChange.LOBBY_MEMBER_STATE_CHANGED_ENTERED &&
                GameObject.Find("Online2PlayerGameEnd") == null)
            {
                GameObject.Find("PopUps").GetComponent<PopUps>().PopUpWithLeaveLobbyButton("Other player left lobby", "Back");
                GameManager.Instance.GameFinished = true;
        }
```

This callback is fired every time a lobby member state changes. We use it to detect if other lobby members left the lobby. We check if the state of the lobby member has changed to anything else other than `LobbyMemberStateChange.LOBBY_MEMBER_STATE_CHANGED_ENTERED`. If that’s the case and the **Online2PlayerGameEnd** GameObject is not active, we display an appropriate message on the host’s screen along with an option to return to the main menu.
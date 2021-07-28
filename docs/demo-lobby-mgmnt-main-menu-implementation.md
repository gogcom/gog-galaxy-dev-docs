# Lobby Management in Main Menu: Examples of Implementation

## Starting an Online Game

### User Experience

After creating or joining the lobby, the player clicks the *Ready* button. Once both players are ready, the lobby owner can start the game. When the game starts, a new multiplayer match level starts loading for both players. After the level is loaded, the game begins.

### Solution

This is accomplished in two scenes: **MainMenu** and **Online2PlayerGame**. In the **MainMenu** scene, we check the `state` of lobby members’ data every time any change is made. If any of the lobby members sets their state to `ready`, we check if both players are ready. If they are, the lobby data state is set to `ready`, which activates the *Start* game button for the lobby owner. After the owner clicks the *Start game* button, the lobby data state is set to `steady`, which triggers the players to load the **Online2PlayerGame** level. Once the level is loaded, each player sets their own lobby member data state to `go`. When both players set their lobby member data state to `go`, the game starts.

This whole process is handled by the **Matchmaking** and **OnlineWaitController** scripts.

### Variables

The following variables are declared within the **OnlineWaitController** class:

| Variable           | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `entriesContainer` | GameObject, in which all entries with player info will be instantiated |
| `entryPrefab`      | Prefab used to display every player’s entry                  |
| `startGameButton`  | Button used to start the game                                |
| `entriesList`      | List of all GameObjects instantiated in the `entriesContainer` |
| `lobbyID`          | GalaxyID of the currently joined lobby                       |
|`displayPlayerListCoroutine` | Stores a reference to a Unity coroutine used to periodically refresh the player list in the **OnlineWait** screen |

### Methods and Usage

#### OnlineWaitController.OnEnable

```c#
    void OnEnable()
    {
        lobbyID = GalaxyManager.Instance.Matchmaking.CurrentLobbyID;
        displayPlayerListCoroutine = DisplayPlayerListCoroutine();
        StartCoroutine(displayPlayerListCoroutine);
    }
```

Once the **OnlineWaitController** script is enabled, we:

1. Assign the GalaxyID of the entered lobby to the `lobbyID` variable with the `Matchmaking.CurrentLobbyID` method.

2. Call the `DisplayPlayerListCoroutine()` method to display and update the displayed list of players. We want this to happen every frame so that the latest data is always shown.

#### OnlineWaitController.DisplayPlayerListCoroutine

```c#
    IEnumerator DisplayPlayerListCoroutine()
    {
        uint lobbyMembersCount;
        GameObject currentEntry;
        GalaxyID currentMember;

        for(;;)
        {
            lobbyMembersCount = GalaxyManager.Instance.Matchmaking.GetNumLobbyMembers(lobbyID);
            DisposeEntries();
            for (uint i = 0; i < lobbyMembersCount; i++)
            {
                currentMember = GalaxyManager.Instance.Matchmaking.GetLobbyMemberByIndex(lobbyID, i);
                currentEntry = Instantiate(entryPrefab, entriesContainer.transform);
                currentEntry.name = currentMember.ToString();
                currentEntry.transform.GetChild(0).GetComponent<Text>().text = GalaxyManager.Instance.Friends.GetFriendPersonaName(currentMember);
                currentEntry.transform.GetChild(1).GetComponent<Text>().text = GalaxyManager.Instance.Matchmaking.GetPingWith(currentMember).ToString();
                currentEntry.transform.GetChild(2).GetComponent<Text>().text = GalaxyManager.Instance.Matchmaking.GetLobbyMemberData(currentMember, "state");
                entriesList.Add(currentEntry);
            }
            yield return new WaitForSecondsRealtime(0.5f);
        }

    }
```

First, we define three additional variables:

| Variable            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `lobbyMembersCount` | Stores the current number of lobby members; we assign its value by calling `Matchmaking.GetNumLobbyMembers(lobbyID)` |
| `currentEntry`      | Stores the GameObject that is being currently instantiated (for easier access) |
| `currentMember`     | Stores the GalaxyID of the currently displayed user          |

Then, we make sure that the displayed list is empty by calling `DisposeEntries()`. We iterate over lobby members and for each member we:

1. Assign the player’s GalaxyID to the `currentMember` variable by calling `Matchmaking.GetLobbyMemberByIndex(lobbyID, index)`.
2. Instantiate the prefab and assign a reference to the newly created GameObject to `currentEntry`.
3. Set the `currentEntry.name` property (the name of the currently instantiated GameObject) to its GalaxyID (just to make sure that each entry has a unique name in the Unity hierarchy).
4. Set the values of text objects representing the player’s name, ping and state inside the `currentEntry` GameObject using the Unity Transform inheritance property.
5. Add the `currentEntry` (the just instantiated GameObject) to our `entriesList` for easier disposal.

#### OnlineWaitController.DisposeEntries

```c#
    void DisposeEntries()
    {
        foreach (GameObject entry in entriesList)
        {
            Destroy(entry);
        }
        entriesList.Clear();
        entriesList.TrimExcess();
    }
```

This method destroys all displayed entries using a `foreach` loop by calling the `Destroy(Object obj)` method for each entry inside the `entriesList`. Finally, it clears and trims `entriesList`.

#### OnlineWaitController.Ready

```c#
    public void Ready()
    {
        GalaxyManager.Instance.Matchmaking.SetLobbyMemberData("state", "ready");
    }
```

This method calls `Matchmaking.SetLobbyMemberData("state", "ready")` when *Ready* button is pressed.

#### OnlineWaitController.StartGame

```c#
    public void StartGame()
    {
        GalaxyManager.Instance.Matchmaking.SetLobbyData(lobbyID, "state", "steady");
        GalaxyManager.Instance.Friends.SetRichPresence("status", "In online 2 player match");
    }
```

This method sets the lobby data `state` to `steady` when the lobby owner clicks the *Start game* button. When users receive the information that the lobby data `state` is set to `steady`, they automatically start loading the new level. It also sets the lobby owner’s rich presence `status` to `In online 2 player match`.

#### LobbyDataListenerMainMenu.OnLobbyDataUpdated Callback

```c#
        public override void OnLobbyDataUpdated(GalaxyID lobbyID, GalaxyID memberID)
        {
            if (memberID.IsValid())
            {
                if (matchmaking.IsCurrentUserLobbyOwner())
                {
                    matchmaking.SetLobbyData(lobbyID, "state", AllMembersReady(lobbyID) ? "ready" : "notReady");
                }
                return;
            }

            GameObject.Find("OnlineWaitScreen").GetComponent<OnlineWaitController>().startGameButton.GetComponent<Button>().
                interactable = (matchmaking.GetLobbyData(lobbyID, "state") == "ready" &&
                matchmaking.IsCurrentUserLobbyOwner());

            if (matchmaking.GetLobbyData(lobbyID, "state") == "steady")
            {
                Debug.Assert(matchmaking.GetLobbyMemberData(GalaxyManager.Instance.MyGalaxyID, "state") == "ready");
                matchmaking.SetLobbyMemberData("state", "steady");
                friends.SetRichPresence("connect", null);
                matchmaking.LobbyManagmentMainMenuListenersDispose();
                matchmaking.LobbyManagmentInGameListenersInit();
                SceneController.Instance.LoadScene(SceneController.SceneName.Online2PlayerGame, true);
            }
        }
```

This callback is triggered whenever a lobby data or lobby member data was changed. If the lobby data was changed, then the `memberID` will not be a valid GalaxyID. We use this to check whether the callback was triggered by a lobby data or by a lobby member data change by calling the `GalaxyID.IsValid` method on the `memberID` variable. We set the lobby data and the lobby member data to synchronize both players and make sure that both are ready to start the match. In more detail, the algorithm is as follows:

1. At this point, the lobby data and the current user’s lobby member data under the `state` key should be set to `notReady`.
2. Once the current user clicks the *Ready* button, we will set their `state` to `ready`.
3. When the above change is detected and the **LobbyDataListenerInGame.OnLobbyDataUpdated()** callback is fired, we will:

    - Check if the memberID is a valid GalaxyID. Because the change from the step #2 above was done to the lobby member, not the lobby itself, the memberID will be a valid GalaxyID.
    - Call the `Matchmaking.SetLobbyData()` method to set the lobby data under the `state` key to either `ready` or `notReady`, depending on the result of the `AllMembersReady()` method.
    - Use the `return` keyword to stop further execution of this callback.

4. If both lobby members’ data under the `state` key were set to `ready`, then we will set the lobby data under the `state` key to `ready` as well.
5. This will again fire the **LobbyDataListenerInGame.OnLobbyDataUpdated()** callback.

    - Because the change from the step #4 was made to the lobby data, `memberID` will not be a valid GalaxyID and the code within the `if (memberID.IsValid())` block will not be executed.
    - The lobby `state` was set to `ready`, so we'll set the *Start game* button to be interactable for the lobby owner.

6. Once the lobby owner clicks the *Start game* button, the lobby `state` will be changed to `steady`.
7. Once again, the **LobbyDataListenerInGame.OnLobbyDataUpdated()** will be triggered.

    - Because the change from the step #6 was made to the lobby data, `memberID` will not be a valid GalaxyID, and the code within the `if (memberID.IsValid())` block will not be executed.
    - The lobby data `state` is now set to `steady`, so the `if (matchmaking.GetLobbyData(lobbyID, "state") == "steady")` block will be executed:

        - Set the lobby member `state` to `steady`.
        - Remove the `connect` string from the users’ rich presence, so other GOG users can’t join via their rich presence.
        - Dispose of the **LobbyManagementListeners** used in the **MainMenu**, and
        - Initialize **LobbyManagementListeners** used **InGame**.
        - Lastly, we’ll start loading the **Online2PlayerGame** scene.

The rest of this algorithm will be discussed in [Lobby Management In Game: Example of Implementation](demo-lobby-mgmnt-in-game-implementation.md).

#### LobbyDataListener.AllMembersReady

```c#
    bool AllMembersReady(GalaxyID lobbyID)
    {
        uint lobbyMembersCount = matchmaking.GetNumLobbyMembers(lobbyID);
        if (lobbyMembersCount < 2)
            return false;
        for (uint i = 0; i < lobbyMembersCount; i++)
        {
            if (matchmaking.GetLobbyMemberData(matchmaking.GetLobbyMemberByIndex(lobbyID, i), "state") != "ready")
                return false;
        }
        return true;
    }
```

This method is used to check whether a lobby has at least two players and both are ready to start the game. It will return `false` if there are less than 2 players in the lobby, or at least one of the lobby members’ `state` is not equal to `ready`. Otherwise, it will return `true`.
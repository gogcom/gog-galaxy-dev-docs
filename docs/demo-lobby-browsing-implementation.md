# Lobby Browsing: Examples of Implementation

## Browsing Lobbies and Joining a Lobby

### User Experience

When the user enters *Online→Server Browser* menu, a list of all available lobbies is displayed along with their names and buttons for joining them. We show only lobbies that are not full and in this demo the maximum number of players in a lobby is 2.

When the user clicks *Join* button next to the lobby name, they will be taken to the lobby view.

### Solution

The main implementation of displaying multiplayer lobbies list is contained in the **OnlineBrowserController** script. Listeners to manage callbacks can be found in the **Matchmaking** script.

### Methods and Usage

#### OnlineBrowserController Class

```c#
    void OnEnable()
    {
        // Initialize the required listeners
        GalaxyManager.Instance.Matchmaking.LobbyBrowsingListenersInit();
        RequestLobbyList(false);
    }
```

When the user enters the Server Browser menu, the **OnlineBrowserController** script is enabled, since it is a component of **OnlineBrowserScreen**. It initializes the lobby browsing listeners in **Matchmaking** and calls the `RequestLobbyList(false)` method.

#### OnlineBrowserController.RequestLobbyList

```c#
    public void RequestLobbyList(bool refresh)
    {
        waitingCircularArrow.SetActive(true);
        if (refresh)
        {
            DisposeLobbiesList();
        }
        GalaxyManager.Instance.Matchmaking.RequestLobbyList();
    }
```

`RequestLobbyList(bool refresh)` activates the **waitingCircularArrow** GameObject, which is shown while the lobby list is being requested and prepared to be displayed.

If the `refresh` argument was `true`, the lobby list displayed previously would be flushed with the `DisposeLobbiesList()` method. In the end, the list of lobbies is requested with `Matchmaking.RequestLobbyList()`.

#### LobbyListListenerBrowsing

```c#
        public override void OnLobbyList(uint count, LobbyListResult result)
        {
            if (result != LobbyListResult.LOBBY_LIST_RESULT_SUCCESS)
            {
                Debug.LogWarning("OnLobbyList failure reason: " + result);
                return;
            }
            Debug.Log(count + " lobbies OnLobbyList");
            matchmaking.lobbyCount = count;
            if (count == 0)
            {
                GameObject.Find("OnlineBrowserScreen").GetComponent<OnlineBrowserController>().DisplayLobbyList(null);
                return;
            }
            for (uint i = 0; i < count; i++)
            {
                GalaxyID lobbyID = GalaxyInstance.Matchmaking().GetLobbyByIndex(i);
                Debug.Log("Requesting lobby data for lobby " + i + " with lobbyID " + lobbyID.ToString());
                matchmaking.RequestLobbyData(lobbyID);
            }
        }
```

The callback on retrieving a lobby list comes to the `LobbyListListenerBrowsing()` listener and then `LobbyListListenerBrowsing().OnLobbyList()` is called. If the request was successful, this method checks whether there is at least one lobby on the list. If this is the case, it requests lobby data for all lobbies using a `for` loop, which gets the GalaxyID of each lobby, stores it in the `lobbyID` variable, and then requests data for this particular lobby with the `Matchmaking.RequestLobbyData(lobbyID)` method.

#### LobbyDataRetrieveListenerBrowsing

Callback to `Matchmaking.RequestLobbyData(lobbyID)` comes to `LobbyDataRetrieveListenerBrowsing.OnLobbyDataRetrieveSuccess()`:

```c#
        public override void OnLobbyDataRetrieveSuccess(GalaxyID lobbyID)
        {
            Debug.Log("Data retrieved for " + lobbiesWithDataRetrievedCount + " lobbies out of " + matchmaking.lobbyCount);
            string name = GalaxyManager.Instance.Matchmaking.GetLobbyData(lobbyID, "name");
            string ping = GalaxyManager.Instance.Matchmaking.GetPingWith(lobbyID).ToString();
            object[] lobbyWithDetails = {name, ping, lobbyID};
            lobbyListWithDetails.Add(lobbyWithDetails);
            lobbiesWithDataRetrievedCount ++;
            if (lobbiesWithDataRetrievedCount >= matchmaking.lobbyCount)
            {
                GameObject.Find("OnlineBrowserScreen").GetComponent<OnlineBrowserController>().DisplayLobbyList(lobbyListWithDetails);
                lobbiesWithDataRetrievedCount = 0;
                matchmaking.lobbyCount = 0;
                lobbyListWithDetails.Clear();
                lobbyListWithDetails.TrimExcess();
            }
        }
```

Within this method we:

1. Retrieve the lobby data — its `name`, `ping` and `lobbyID` — using the `Matchmaking.GetLobbyData()` and `Matchmaking.GetPingWith()` methods (these methods are wrappers for [`GetLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a9f5a411d64cf35ad1ed6c87d7f5b465b) and the [`GetPingWith()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html#a8949164f11b2f956fd0e4e1f1f8d1eb5) methods, respectively, from the GOG GALAXY SDK) and store them in the `lobbyWithDetails` variable.
2. Add the `lobbyWithDetails` object, which stores the retrieved lobby data, to the lobby list (`lobbyListWithDetails`).
3. Increment the `lobbiesWithDataRetrievedCount` value every time the listener receives a callback.
4. Check if `lobbiesWithDataRetrievedCount` is bigger than or equal to `matchmaking.lobbyCount` (the total number of lobbies retrieved by the `LobbyListListenerBrowsing` listener). If it is, display the list using `OnlineBrowserController.DisplayLobbyList()`, set `lobbiesWithDataRetrievedCount` and `matchmaking.lobbyCount` values back to zero, and lastly clear and trim `lobbyListWithDetails`.

### Joining the Game

```c#
    public void DisplayLobbyList(List<object[]> lobbyList)
    {
        Debug.Log("Displaying lobby list");
        waitingCircularArrow.SetActive(false);
        GameObject currentEntry;

        if (lobbyList == null) return;

        foreach (object[] lobby in lobbyList)
        {
            Debug.Log("Current lobby ID " + lobby[2].ToString());
            currentEntry = Instantiate(entryPrefab, entriesContainer.transform);
            currentEntry.transform.GetChild(0).GetComponent<Text>().text = lobby[0].ToString();
            currentEntry.transform.GetChild(1).GetComponent<Text>().text = lobby[1].ToString();
            currentEntry.transform.GetChild(2).GetComponent<Button>().onClick.AddListener(() =>
            {
                JoinLobby((GalaxyID)lobby[2]);
            });
            entriesList.Add(currentEntry);
        }
    }
```

When the lobby list is displayed, we assign the `OnlineBrowserController.JoinLobby(lobbyID)` method to the event of clicking the *Join* button inside the Server Browser entry prefab.

The  `OnlineBrowserController.JoinLobby(lobbyID)` method allows the user to join the lobby using `Matchmaking.JoinLobby(lobbyID)` and displays **OnlineJoiningScreen** menu:

```c#
    public void JoinLobby(GalaxyID lobbyID)
    {
        Debug.Log("Joining lobby " + lobbyID);
        GalaxyManager.Instance.Matchmaking.JoinLobby(lobbyID);
        GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineJoining);
    }
```

#### LobbyEnteredListenerBrowsing

A callback to the `Matchmaking.JoinLobby(lobbyID)` method comes to **LobbyEnteredListenerBrowsing**, where the `OnLobbyEntered` method checks if the entry was successful or not:

```c#
        public override void OnLobbyEntered(GalaxyID lobbyID, LobbyEnterResult _result)
        {
            if (_result != LobbyEnterResult.LOBBY_ENTER_RESULT_SUCCESS)
            {
                Debug.Log("LobbyEnteredListenerBrowsing failed");
                GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineBrowser);
                GameObject.Find("PopUps").GetComponent<PopUps>().PopUpWithClosePopUpsButton("Could not join lobby\nReason: " +
                    _result.ToString(), "Back");
                return;
            }
            Debug.Log("LobbyEnteredListenerBrowsing succeded");
            matchmaking.CurrentLobbyID = lobbyID;
            matchmaking.LobbyOwnerID = matchmaking.GetLobbyOwner(lobbyID);
            matchmaking.SetLobbyMemberData("state", "notReady");
            GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineWait);
            friends.SetRichPresence("status", "In online lobby");
            friends.SetRichPresence("connect", "--JoinLobby=" + lobbyID);
            matchmaking.LobbyManagmentMainMenuListenersInit();
            matchmaking.LobbyChatListenersInit();
            matchmaking.LobbyBrowsingListenersDispose();
        }
```

First, we check if the result of entering a lobby was anything else but successful.

If this is the case, it means we did not succeed at entering the lobby, therefore we want to inform the user about that:

1. We go back to lobby browsing menu.
2. Display a pop up informing that we could not join the lobby and why.
3. Use the `return` keyword to prevent the rest of the code within the `OnLobbyEntered()` callback from executing.

Otherwise, it means we did succeed at entering the lobby, and we have to ensure that everything is ready:

1. Set the `CurrentLobbyID` variable within the **GalaxyManager.Instance.Matchmaking** instance to the lobby ID of the currently joined lobby. The `CurrentLobbyID` variable is later used to streamline the usage of methods in the **Matchmaking** class.
2. Set the `LobbyOwnerID` variable within the **GalaxyManager.Instance.Matchmaking** instance to the Galaxy ID of the owner of the currently joined lobby. The `LobbyOwnerID` variable is later used to check if the current user is the lobby owner.
3. Set the lobby member data of current users under the `state` key to `notReady`. This data is later used to verify whether the lobby members are ready to start a game or not.
4. Switch to the **OnlineWait** screen, which serves as a pre-game lobby where players can chat and set their state to `ready`, once they are ready to start the game. Additionally, the owner of the lobby can start the game, when both players are ready.
5. Set the rich presence `status` key of the users to `In online lobby` (this will be visible to GOG friends of the users in the GOG GALAXY friends list and chat) and the `connect` key to `"--JoinLobby=" + lobbyID` (this allows GOG friends of the users to join the lobby they are in via the GOG GALAXY friends list or chat).
6. Finally, we make sure that the required **LobbyManagmentMainMenu** and **LobbyChat** listeners are initialized, and the no longer needed **LobbyBrowsingListeners** are disposed of.

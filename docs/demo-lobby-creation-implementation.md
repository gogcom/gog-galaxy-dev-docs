# Lobby Creation: Examples of Implementation

## Creating an Online Lobby

### User Experience

The user opens the *Create lobby* menu, enters the game name, sets the privacy setting, and clicks the *Create* button. A waiting screen is displayed while the lobby is being created in the background. When the lobby is ready, the lobby menu is displayed.

### Solution

When **OnlineCreateScreen** is opened, the **OnlineCreateController** (available at *Assets/Scripts/UI/MainMenu/OnlineCreateController.cs*) will call the `Matchmaking.LobbyCreationListenersInit()` method that will in turn initialize the listeners required for creating a lobby. When *Create lobby* button is clicked, the `CreateLobby()` method is called. The `CreateLobby()` method will prepare the lobby data and then call the `Matchmaking.CreateLobby()` method in the **Matchmaking** script (available at *Assets/Scripts/GalaxyManager/Features/Matchmaking.cs*).

### Variables

Inside the script, you will find the following variables:

| Variable   | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `privacy`  | Reference to the **toggle** GameObject that stores lobby information; a lobby can be private (`privacy` set to `true`) or public (`privacy` set to `false`) |
| `gameName` | Reference to the **input field** GameObject that stores the lobby name typed in by the user |
| `message`  | Reference to the **text** GameObject used to display a warning if the user left the game name input field empty |

### Methods and Usage

#### OnlineCreateController.CreateLobby

```c#
    public void CreateLobby()
    {
        LobbyTopologyType lobbyTopologyTypeFCM = Galaxy.Api.LobbyTopologyType.LOBBY_TOPOLOGY_TYPE_STAR;
        uint maxMembers = 2;
        LobbyType lobbyPrivacy;
        if (gameName.text == "")
        {
            Debug.Log("Failure to create lobby: Game name can't be empty");
            message.text = "Game name can't be empty";
            message.gameObject.SetActive(true);
            return;
        }
        lobbyPrivacy = privacy.isOn?
            Galaxy.Api.LobbyType.LOBBY_TYPE_PRIVATE:
            Galaxy.Api.LobbyType.LOBBY_TYPE_PUBLIC;
            GalaxyManager.Instance.Matchmaking.CreateLobby(gameName.text, lobbyPrivacy, maxMembers, true, lobbyTopologyTypeFCM);
            message.gameObject.SetActive(false);
            GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineCreating);
        }
    }
```

1. First, we create three additional variables:

    | Variable               | Description                                                  |
    | ---------------------- | ------------------------------------------------------------ |
    | `lobbyTopologyTypeFCM` | Stores the information about the lobby topology that we want to use. In our case it is *Galaxy.Api.LobbyTopologyType.LOBBY_TOPOLOGY_TYPE_FCM* — a lobby type in which information is shared among players (P2P connection) |
    | `maxMembers`           | Stores information about how many players can join a lobby; in our case it is 2 |
    | `lobbyPrivacy`         | Stores information about whether the lobby is private or not |

2. We check if the game name input field is empty. If it is, and the user clicks *Create lobby* button, we display a message asking the user to enter the game name.

3. Check if the privacy toggle is selected (set to `true`). If that’s the case, `lobbyPrivacy` is set to `Galaxy.Api.LobbyType.LOBBY_TYPE_PRIVATE`; otherwise we set it to `Galaxy.Api.LobbyType.LOBBY_TYPE_PUBLIC`.

4. We call the `Matchmaking.CreateLobby()` method with the prepared lobby name (`gameName.text`), the lobby privacy setting (`lobbyPrivacy`), the maximum number of lobby members (`maxMembers`), information on whether the users should be able to join the lobby (by providing the value of `true` for the `joinable` parameter), and lastly – the information that the lobby should use the peer to peer topology (`lobbyTopologyTypeFCM`). This method is essentially a wrapper for the GOG GALAXY SDK [`CreateLobby`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a092de26b8b91e450552a12494dce4644) method.

5. `message` is disabled (in case it was displayed).

6. The displayed menu is switched to **OnlineCreatingScreen**, which consists of a spinning arrow and a message to inform the user that the lobby is being created.

#### LobbyCreatedListener.OnLobbyCreated Callback

This listener is contained in the **Matchmaking** script:

```c#
        public override void OnLobbyCreated(GalaxyID lobbyID, LobbyCreateResult _result)
        {
            if (_result != LobbyCreateResult.LOBBY_CREATE_RESULT_SUCCESS)
            {
                Debug.Log("LobbyCreatedListenerCreation failed");
                GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineCreate);
                GameObject.Find("PopUps").GetComponent<PopUps>().PopUpWithClosePopUpsButton("Could not create lobby\nReason: " +
                    _result.ToString(), "back");
                return;
            }
            Debug.Log("LobbyCreatedListenerCreation succeded");
            matchmaking.SetLobbyData(lobbyID, "name", matchmaking.lobbyName);
            matchmaking.SetLobbyData(lobbyID, "state", "notReady");
            matchmaking.CurrentLobbyID = lobbyID;
            matchmaking.LobbyOwnerID = matchmaking.GetLobbyOwner(lobbyID);
            matchmaking.SetLobbyMemberData("state", "notReady");
            GameObject.Find("MainMenu").GetComponent<MainMenuController>().SwitchMenu(MainMenuController.MenuEnum.OnlineWait);
            friends.SetRichPresence("status", "In online lobby");
            friends.SetRichPresence("connect", "--JoinLobby=" + lobbyID);
            matchmaking.LobbyManagmentMainMenuListenersInit();
            matchmaking.LobbyChatListenersInit();
            matchmaking.LobbyCreationListenersDispose();
        }
```

First, we check if the result of entering a lobby was anything else but successful.

If that is the case, it means we did not succeed at entering the lobby, therefore we want to inform the user about that:

1. We go back to the lobby creating menu.
2. Display a pop up informing that we could not create the lobby and why.
3. Use the `return` keyword to prevent the rest of the code within the `OnLobbyCreated` callback from executing.

Otherwise, it means we did succeed at entering the lobby, and we have to make sure everything is ready:

1. Set the current lobby data under the `name` key to the lobby name so that the proper name is displayed for users browsing lobbies.
2. Set the current lobby data under the `ready` key to `notReady`. This is later used to verify whether the game can be started.
3. Set the `CurrentLobbyID` within the `GalaxyManager.Instance.Matchmaking` instance to the lobby ID of the currently joined lobby. The `CurrentLobbyID` variable is later used to streamline the usage of methods in the **Matchmaking** class.
4. Set the `LobbyOwnerID` within the `GalaxyManager.Instance.Matchmaking` instance to the Galaxy ID of the owner of the currently joined lobby. The `LobbyOwnerID` is later used to check if the current user is the lobby owner.
5. Set the current users’ lobby member data under the `state` key to `notReady`. This data is later used to verify whether the lobby members are ready to start a game or not.
6. Switch to the **OnlineWait** screen, which serves as a pre-game lobby where players can chat and set their state to `ready`, once they are ready to start the game. Additionally, the owner of the lobby can start the game, once both players are ready.
7. Set the rich presence `status` key of the users to `In online lobby` (this will be visible to GOG friends of the users in the GOG GALAXY friends list and chat) and the `connect` key to `"--JoinLobby=" + lobbyID` (this allows GOG friends of the users to join the lobby they are in via the GOG GALAXY friends list or chat)..
8. Finally, we make sure that the required **LobbyManagmentMainMenu** and **LobbyChat** listeners are initialized, and the no longer needed **LobbyCreatingListeners** are disposed of.

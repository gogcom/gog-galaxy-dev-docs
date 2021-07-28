# Invitations

## Description

This class is responsible for accepting game invitations, for which there can be two scenarios:

- When a user accepts an invite while already playing the game (the `SetUpInviteInGame` method).
- When a user accepts an invite from the GOG GALAXY client (the `CheckForInviteFromClient` and the `SetUpInviteFromClient` methods).

In both situations, the received **connectionString** that has been sent along with the invite is parsed in order to retrieve a **lobbyID** and consequently to make the user join the lobby they were invited to.

## Class Initialization and Termination

Like with other classes, this class initializes its own listeners when it is enabled and disposes of them when it is disabled.

Additionally, in the **OnEnable** MonoBehaviour class, we call the `CheckForInviteFromClient` method. This method will go through all command line arguments that the game was launched with and detect if the `--JoinLobby=ConnectionString` argument was used. More about this [in the example of implementation](demo-invitations-implementation.md#invitationscheckforinvitefromclient).

```c#
    void OnEnable()
    {
        ListenersInit();
        CheckForInviteFromClient();
    }
```

## Definitions of Listeners

### GameJoinRequestedListener

```c#
    private class GameJoinRequestedListener : GlobalGameJoinRequestedListener
    {
        public override void OnGameJoinRequested(GalaxyID userID, string connectionString)
        {
            Debug.Log("OnGameJoinRequested userID: \"" + userID + "\" connectionString \"" + connectionString + "\"");
            GalaxyManager.Instance.Invitations.SetUpInviteInGame(connectionString);
        }
    }
```

This listener callback `OnGameJoinRequested` is triggered when a user is in game and accepts an incoming invitation. We use it to launch the `SetUpInviteInGame(string connectionString)` method.

### GameInvitationReceivedListener

```c#
    private class GameInvitationReceivedListener : GlobalGameInvitationReceivedListener
    {
        public override void OnGameInvitationReceived(GalaxyID userID, string connectionString)
        {
            Debug.Log("OnGameInvitationReceived userID: \"" + userID + "\" connectionString \"" + connectionString + "\"");
        }
    }

```

This listener callback `OnGameInvitationReceived` is triggered when a user receives a game invitation. We don’t use this feature in our project, but you can use this callback to display an in-game pop-up for the user, when a game invitation is received. We opted for handling the game invitations via the [GOG GALAXY Overlay](gc-overlay.md), which is accessible at all times with **Shift+Tab**.
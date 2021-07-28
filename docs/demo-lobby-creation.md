# Lobby Creation

## Description

There are two main methods responsible for lobby creation process — `CreateLobby()` in **OnlineCreateController** and a method of the same name in **Matchmaking** — as well as one listener (**LobbyCreatedListener**).

## Initialization and Termination

The listener is enabled by the **OnlineCreateController** script and disposed of when the users either creates a lobby or leaves the **OnlineCreateScreen**.

## Definitions of Listeners

### LobbyCreatedListener

This listener receives callbacks for the event of creating a lobby. The `OnLobbyCreated()` method informs whether a lobby was successfully created: if it was, the user joins it and it becomes ready for use.

!!! Note
    We don’t use the **LobbyEnteredListener** here, but if we were using it, a notification for the **LobbyEnteredListener.OnLobbyEntered** would follow immediately after **LobbyCreated.OnLobbyCreated**, since the owner automatically enters the lobby they created.

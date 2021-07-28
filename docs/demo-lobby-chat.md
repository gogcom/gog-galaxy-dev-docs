# Lobby Chat

## Description

Chat in a lobby is handled by the **ChatController** class (available at *Assets/Scripts/UI/GameMenu/MultiScene/ChatController.cs*), and by the **Matchmaking** class, which is responsible for sending and receiving lobby chat messages. **LobbyMessageListenerChat**, which is required for handling lobby chat messages, is initialized when user joins or creates a lobby.

## Initialization and Termination

When the user joins (`LobbyEnteredListenerBrowsing.OnLobbyEntered()`) or creates (`LobbyCreatedListener.OnLobbyCreated()`) a lobby, the `Matchmaking.LobbyChatListenersInit()` method is called and it initializes the **LobbyMessageListenerChat** listener. The listener is disposed of when the user leaves a lobby (`LobbyLeftListenerInGame.OnLobbyLeft()` or `LobbyLeftListenerMainMenu.OnLobbyLeft()`).

## Definitions of Listeners

### LobbyMessageListenerChat

This is a listener for the event of receiving a lobby message. When a message comes to the lobby, `OnLobbyMessageReceived()` is called, which provides the information on the message ID and the length of the message.
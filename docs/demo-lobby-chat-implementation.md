# Lobby Chat: Examples of Implementation

## Sending and Receiving Lobby Chat Messages

### User Experience

We want the players to be able to communicate with each other while in the same lobby — both in the lobby view and during a match.

### Solution

GOG GALAXY SDK methods for sending and reading lobby messages are gathered in the **Matchmaking** class. However, the **LobbyMessageListenerChat** listener is initialized separately in **Matchmaking.LobbyEnteredListenerBrowsing.OnLobbyEntered** in the event of successfully entering a lobby, or in **Matchmaking.LobbyCreatedListener.OnLobbyCreated** in the event of successfully creating a lobby (both listeners are defined in the **Matchmaking** class available at *Assets/Scripts/GalaxyManager/Features/Matchmaking.cs*).

Lobby messages can be sent from the input box in the lobby view and during an online multiplayer match (pressing the ++t++ key displays the input box). All messages are shown for both players in the lobby. Actions of reading and sending message from the chat input field, as well as displaying them, are executed by the **ChatController** script (available at *Assets/Scripts/UI/MultiScene/ChatController.cs*) attached to the **OnlineChat** GameObject in the game.

### Methods and Usage

The **Matchmaking** script is attached to the **GalaxyManager** GameObject when a player enters the *Online* section from the game main menu, and the **ChatController** script is added when a player joins the lobby (by creating it or joining an existing one).

#### ChatController.SendLobbyMessage

```c#
    public void SendLobbyMessage()
    {
        string message = messagePrompt.text;
        if (message.Length != 0)
        {
            GalaxyManager.Instance.Matchmaking.SendLobbyMessage(lobbyID, message);
            messagePrompt.text = null;
        }
    }
```

This method is called when the player clicks the *Send message* button in the pre-game lobby, or opens the chat prompt by pressing the ++t++ key and hits ++enter++ while in a match. The text from the `messagePrompt` InputField is assigned to the `message` variable. If the message is not empty (which can happen, when the user presses ++enter++ when there is no text in the `messagePrompt`), it is sent by `Matchmaking.SendLobbyMessage(lobbyID, message)` and the text in the input box is cleared.

A callback then comes to `Matchmaking.LobbyMessageListenerChat()`, and the `OnLobbyMessageReceived()` method is called.

#### LobbyMessageListenerChat.OnLobbyMessageReceived

```c#
        public override void OnLobbyMessageReceived(GalaxyID lobbyID, GalaxyID senderID, uint messageID, uint messageLength)
        {
            Dictionary<string, string> messageAndSenderDict = new Dictionary<string, string>();
            ChatController chatMenuController = GameObject.Find("OnlineChat").GetComponent<ChatController>();
            Debug.Log("Message from lobby: \'" + lobbyID + "\', sender: \'" + senderID + "\', with value: \'" + message +
                "\' received.");
            message = matchmaking.GetLobbyMessage(matchmaking.CurrentLobbyID, ref senderID, messageID);
            messageAndSenderDict.Add("sender", friends.GetFriendPersonaName(senderID));
            messageAndSenderDict.Add("message", message);
            chatLobbyMessageHistory.Add(messageAndSenderDict);
            if (chatMenuController != null) chatMenuController.DisplayChatMessage(messageAndSenderDict);
            if (GameManager.Instance != null) ((Online2PlayerGameManager)GameManager.Instance).PopChatPrompt();
        }
```

Inside this method, we:

1. Initialize the `messageAndSenderDict` dictionary, which contains the received chat message along with the username of the sender.
2. Call `Matchmaking.GetLobbyMessage(GalaxyID lobbyID, ref senderID, messageID)` when a new message comes to the chat in the lobby, which makes it possible to read the message content by its ID and assign it to the `message` string variable.
3. Add the sender’s username (retrieved with `Friends.GetFriendPersonaName(senderID)`) and the message content as values to the `sender` and `message` keys, respectively, to the `messageAndSenderDict` variable.
4. Add the `messageAndSenderDict` variable (that is, the message with the sender’s username) to `chatLobbyMessageHistory` (the list of all received messages).
5. If `chatMenuController` is not equal to null – that is, it exists in the current Unity scene – we use its `DisplayChatMessage` method to display the received chat message.
6. If `GameManager.Instance` is not equal to null – that is, the player is in a match – we cast it to the **Online2PlayerGameManager** class, so that we would have access to its `PopChatPrompt` method. Then, we use this method to open a chat window for a couple of seconds in order to let the player know that a new chat message was received.

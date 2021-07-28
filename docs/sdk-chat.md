# Chat

This feature of the GOG GALAXY SDK allows GOG users to chat with each other independently from the in-game messaging. GOG chat is available on the [website](https://www.gog.com/account/chat) (after a user logs in), in the [GOG GALAXY client application](gc-chat.md) (when a user clicks a friend on the friends list) and in the [overlay](gc-overlay.md), and can be accessed directly by a game using the GOG GALAXY SDK.

## Getting Chat Room Info

1. First, call [`IChat::RequestChatRoomWithUser()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a1c71546d2f33533cb9f24a9d29764b60), providing the ID of the user you want to start a chat with. (The ID is this user’s GalaxyID).
2. The response comes to [`IChatRoomWithUserRetrieveListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomWithUserRetrieveListener.html). If the request was successful, the [`OnChatRoomWithUserRetrieveSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomWithUserRetrieveListener.html#a1a3812efce6f7e1edf0bcd2193be5f75) method will return the ID of the entered chat room (`chatRoomID`), which is essential for the `IChat` methods.
3. Having this `chatRoomID`, you can:

    - get the total number of users in this chat room with the [`IChat::GetChatRoomMemberCount()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a251ddea16a64be7461e6d5de78a4ad63) method,
    - get the ID of a particular user in this chat room with the [`IChat::GetChatRoomMemberUserIDByIndex()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a6a8716d44283234878d810d075b2bc09) method,
    - get the number of unread messages with the [`IChat::GetChatRoomUnreadMessageCount()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#ae29c115c1f262e6386ee76c46371b94b) method.

## Sending Messages

1. [Request the chat room info](#getting-room-info) as described in the steps #1 and #2.
2. Call the [`IChat::SendChatRoomMessage()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a11cff02f62369b026fd7802bdeb9aca1) method, giving the chat room ID and the message text as its input parameters.
3. If the message was sent successfully, the [`IChatRoomMessageSendListener::OnChatRoomMessageSendSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomMessageSendListener.html#a36c02be9c4ab7ad034ffaec9a5b0aa2d) callback method will be called, providing the ID of the chat room along with the internal index, the ID and the timestamp of the sent message, so you can track the sent messages (or indicate that sending is in progress).
4. The recipients (members of the chat room) will receive a notification about an incoming message using the [`IChatRoomMessagesListener::OnChatRoomMessagesReceived()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomMessagesListener.html#ab440a4f2f41c573c8debdf71de088c2a) callback method, which provides the ID of the chat room, the number of messages received in this chat room and the length of the longest message, so you can allocate a buffer of an appropriate size.

## Receiving Messages

You don’t have to call the `IChat::RequestChatRoomWithUser()` explicitly in order to receive new incoming messages. The notification about a new message comes to [`IChatRoomMessagesListener::OnChatRoomMessagesReceived()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomMessagesListener.html#ab440a4f2f41c573c8debdf71de088c2a) callback method, providing the ID of the chat room, the number of messages received in this chat room and the length of the longest message, so you can allocate a buffer of an appropriate size.

Within the `OnChatRoomMessagesReceived()` call, you can retrieve the subsequent messages with the [`IChat::GetChatRoomMessageByIndex()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#ae8d32143755d50f5a0738287bce697f9) method, providing their index number and the size of the output buffer as input parameters. As a result, this method returns:

- the global ID of the message,
- the type of the message,
- the message sender’s ID,
- the time when the message was sent,
- the contents of the message.

When the retrieval is completed, you can mark this chat room as read with the [`IChat::MarkChatRoomAsRead()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a1b8ef66f124412d9fef6e8c4b1ee86c3) method.

## Fetching Historical Messages

1. [Request the chat room info](#getting-room-info) as described in the steps #1 and #2.
2. Call the [`IChat::RequestChatRoomMessages()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#acad827245bab28f694508420c6363a1f) method in order to retrieve:
    - messages older than the one defined by its ID in the `referenceMessageID` parameter, or
    - a specific number of latest messages defined by the `limit` parameter (when the `referenceMessageID` parameter is left empty).
3. The response comes to [`IChatRoomMessagesRetrieveListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomMessagesRetrieveListener.html). If the operation was successful, the [`OnChatRoomMessagesRetrieveSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChatRoomMessagesRetrieveListener.html#af640da6293b7d509ae0539db4c7aa95a) callback method will provide the number of messages received along with the length of the longest message, so you can allocate a buffer of an appropriate size.
4. Now you can retrieve the subsequent messages with the [`IChat::GetChatRoomMessageByIndex()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#ae8d32143755d50f5a0738287bce697f9) method, providing their index number and the size of the output buffer as input parameters. As a result, this method returns:
    - the global ID of the message,
    - the type of the message,
    - the message sender’s ID,
    - the time when the message was sent,
    - the contents of the message.
5. When the retrieval is completed, you can mark this chat room as read with the [`IChat::MarkChatRoomAsRead()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html#a1b8ef66f124412d9fef6e8c4b1ee86c3) method.
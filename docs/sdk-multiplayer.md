# Multiplayer

The GOG GALAXY SDK allows you to use its features to implement **multiplayer**, **matchmaking** and **P2P communication** in your game.

## Hosting a Multiplayer Lobby

*As a user trying to host a lobby*

Creating a lobby is pretty straightforward: you need to call the [`IMatchmaking::CreateLobby()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a092de26b8b91e450552a12494dce4644) functions, which takes in the following parameters:

| Parameter           |                             Type                             | Description                                                  |
| :------------------ | :----------------------------------------------------------: | ------------------------------------------------------------ |
| `lobbyType`         | [LobbyType](https://docs.gog.com/galaxyapi/group__api.html#ga311c1e3b455f317f13d825bb5d775705) | Specifies the type of a lobby: private, public, visible only to friends, or even invisible only to friends |
| `maxMembers`        |                           integer                            | Maximum number of lobby members                              |
| `joinable`          |                             bool                             | Determines whether the lobby is joinable immediately after its creation |
| `lobbyTopologyType` | [LobbyTopologyType](https://docs.gog.com/galaxyapi/group__api.html#ga966760a0bc04579410315346c09f5297) | Specifies the way users connect to each other                |

The GOG GALAXY SDK allows you to create lobbies of several topologies:

| Lobby Topology                               | Description                                                  |
| :------------------------------------------- | ------------------------------------------------------------ |
| LOBBY_TOPOLOGY_TYPE_FCM                      | Every user connects with every other user (and the multiplayer server); no ownership migration |
| LOBBY_TOPOLOGY_TYPE_STAR                     | Every user connects to the lobby owner (and the multiplayer server); no ownership migration |
| LOBBY_TOPOLOGY_TYPE_CONNECTIONLESS           | Every user connects only to the server with lobby owner transition, direct communication between users is not possible, yet lobby/member data is propagated as usual; sending lobby messages works |
| LOBBY_TOPOLOGY_TYPE_FCM_OWNERSHIP_TRANSITION | Every user connects with every other user with lobby owner transition |

After the `IMatchmaking::CreateLobby()` call is made and a lobby creation request has been completed, you will receive two responses: one to [`ILobbyCreatedListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyCreatedListener.html) with the ID of your lobby, and the other one to [`ILobbyEnteredListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyEnteredListener.html) — provided that the lobby was created successfully, in which case the owning user enters it automatically. Then lobby data and user information is automatically requested for every member of the lobby.

## Setting Lobby Data

*As a user hosting a lobby, after performing the steps in the [Hosting a Multiplayer Lobby](#hosting-a-multiplayer-lobby) scenario above*

The next thing you might want to do after hosting a lobby is specifying Lobby Data — that is data that can hold lobby properties and can make the lobby distinguishable from others. In order to do that, you need to call [`IMatchmaking::SetLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a665683ee5377f48f6a10e57be41d35bd) that takes 3 parameters: the ID of the lobby for which you want to set the data for, the key under which the data will be held, and the value of that key. For example:

```
SetLobbyData(1234, "LobbyName", "TestLobby")
```

## Setting Other Lobby Properties

*As a user hosting a lobby, after performing the steps in the [Hosting a Multiplayer Lobby](#hosting-a-multiplayer-lobby) scenario*

Creating a multiplayer lobby is not the only time you can specify lobby properties: states like “joinable” etc. can also be modified after a lobby has been created. In order to do that, you can use functions like [`IMatchmaking::SetLobbyJoinable()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a1b2e04c42a169da9deb5969704b67a85), [`IMatchmaking::SetLobbyType()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#ab7e0c34b0df5814515506dcbfcb894c0) or [`IMatchmaking::SetMaxNumLobbyMembers()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#acbb790062f8185019e409918a8029a7c). For a full list of functions please refer to the [SDK documentation](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html).

## Requesting a List of Lobbies

*As a user trying to join an already hosted lobby*

First thing you will need to do is call the [`IMatchmaking()::RequestLobbyList()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a3968e89f7989f0271c795bde7f894cd5) function. This function can take a Boolean parameter specifying whether to include lobbies which are full. After the call processes you should receive a response to [`ILobbyListListener::OnLobbyList()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyListListener.html#a95e2555359b52d867ca6e2338cbcafcf) listener function.

That response will contain the number of lobbies found and whether there was an error during retrieving them. Inside that listener function you can use the [`IMatchmaking::GetLobbyByIndex()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#aec7feb30e61e3ad6b606b23649c56cc2) method to retrieve a lobby ID (please note that this is the only place that this method can be called in). Once you have the ID of a lobby, you will want to call the [`IMatchmaking::RequestLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a26d24d194a26e7596ee00b7866c8f244) function, passing the ID as a parameter. This allows you to retrieve information about a specified lobby; the call is asynchronous, so you need to wait for a response to an appropriate listener ([`ILobbyDataListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyDataListener.html) or [`ILobbyDataRetrieveListener)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyDataRetrieveListener.html).

## Using Lobby Data

*As a user trying to join an already hosted lobby and after performing the steps in the [Requesting a List of Lobbies](#requesting -a-list-of-lobbies) scenario above*

Once a response to the [`IMatchmaking::RequestLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a26d24d194a26e7596ee00b7866c8f244) function is received, you should call [`IMatchmaking::GetLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a9f5a411d64cf35ad1ed6c87d7f5b465b), using the ID of the lobby and the key of the property, which you want to retrieve. For example, in order to retrieve the *“LobbyName”* property, which was set in the **Setting Lobby Data** scenario, you would call `GetLobbyData(1234, "LobbyName")` and receive *“TestLobby”* as a return value.

## Retrieving Information Other Than Lobby Data

*As a user trying to join an already hosted lobby and after performing the steps in the [Requesting a List of Lobbies](#requesting -a-list-of-lobbies) scenario*

Once a response to the [`IMatchmaking::RequestLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a26d24d194a26e7596ee00b7866c8f244) function is received, you can call various functions in order to retrieve different information about the lobby, for example: [`IMatchmaking::GetNumLobbyMembers()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a8007909a1e97a3f026546481f0e06c12) or [`IMatchmaking::GetLobbyOwner()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#ab70f27b6307243a8d3054cb39efc0789). For a list of all functions that can be called on a lobby before and after joining it, please refer to the [SDK documentation](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#details).

## Joining a Lobby

*As a user trying to join an already hosted lobby and after performing the steps in the [Requesting a List of Lobbies](#requesting -a-list-of-lobbies) scenario*

After you select the ID of the lobby to which you want to connect to, you can call [`IMatchmaking::JoinLobby()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a809b3d5508a63bd183bcd72682ce7113). This call is asynchronous and after calling it, a response will come to the [`ILobbyEnteredListener::OnLobbyEntered()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyEnteredListener.html#a6031d49f0d56094761deac38ca7a8460) listener function, indicating whether you successfully joined the lobby and providing the ID of the lobby you have just joined.

## Communicating Inside a Lobby

*As a user hosting a lobby, or as a user that has joined a lobby*

There is a couple of ways to communicate with other users in a lobby:

1. **Sending lobby messages** — you can call the [`IMatchmaking::SendLobbyMessage()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a61228fc0b2683c365e3993b251814ac5) function to send a message to all members in the lobby (including yourself). Each recipient should be registered for notifications about incoming messages with a dedicated listener, and when such notification is received, call [`IMatchmaking::GetLobbyMessage()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a372290fa1405296bae96afe7476e1147).
2. **Sending P2P Packets** — you can use the [`INetworking::SendP2PPacket()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html#a1fa6f25d0cbd7337ffacea9ffc9032ad) function to send packets to other lobby members or to the lobby owner (depending on the topology of the lobby).
3. **Setting lobby data** — the lobby owner can set various lobby data, using the [`IMatchmaking::SetLobbyData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a665683ee5377f48f6a10e57be41d35bd) function, which the lobby members can then read.

A lobby topology specifies, to whom you should send p2p packets:

- **FCM** — packets should be sent to every other lobby member.
- **Star** — packets should be sent to the lobby owner and the lobby owner should then redistribute packets to other members.
- **Connectionless** — connectionless type does not support packet sending at this moment; it’s ideal for making a chatroom and other features, which can rely only on lobby messages and lobby data.

## Leaving a Lobby

*As a user hosting a lobby, or as a user that has joined a lobby*

If you are a user that has joined a lobby — and are not the lobby owner — you can simply call [`IMatchmaking::LeaveLobby()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a96dfce0b1e35fdc46f61eadaa9639f49) to initiate leaving a lobby. You will receive a callback to [`ILobbyLeftListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyLeftListener.html) once leaving is finished.  Other lobby members will receive a callback to their [`ILobbyMemberStateListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyMemberStateListener.html), indicating that someone has left the lobby. If you are the lobby owner, you also use the `IMatchmaking::LeaveLobby()` method, but what happens to the lobby after you leave, depends on the topology:

- **FCM** — disconnection of the owner from the lobby results in closing the lobby.
- **Star** — disconnection of the owner from the lobby results in closing the lobby.
- **Connectionless** — disconnection of the owner from the lobby results in choosing a new lobby owner.
- **FCM Ownership Transition** — disconnection of the owner from the lobby results in choosing a new lobby owner.

## GameServer

The implementation of GameServers (dedicated servers) in the GOG GALAXY SDK is not much different from the implementation of a user-hosted game. The only differences between GameServers and user-hosted multiplayer games are as follows:

- a server shall be logged in with the [`SignInAnonymous()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a9d4af026704c4c221d9202e2d555e2f1) method
- not all interfaces and methods are available for GameServer, e.g. the [`IChat`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IChat.html), [`IFriends`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html) interfaces or the [`IUser::SetUserData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a2bee861638ff3887ace51f36827e2af0), [`IMatchmaking::RequestLobbyList()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a3968e89f7989f0271c795bde7f894cd5) methods
- GameServers are using another set of global listeners prefixed with “GameServer”, e.g.:
    - [`IUser::GameServerGlobalAuthListener()`](https://docs.gog.com/galaxyapi/group__api.html#gaf49ff046f73b164653f2f22fb3048472)
    - [`IMatchmaking::GameServerGlobalLobbyCreatedListener()`](https://docs.gog.com/galaxyapi/group__api.html#ga0f6bf49d1748a7791b5a87519fa7b7c8)
    - [`INetworking::GameServerGlobalServerNetworkingListener()`](https://docs.gog.com/galaxyapi/group__api.html#ga273ad4080b9e5857779cd1be0a2d61e8).
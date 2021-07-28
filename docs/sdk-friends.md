# Friends

The GOG GALAXY SDK allows you to download the list of your friends and get detailed information about them.

!!! Important
    After the user is logged in, the GOG GALAXY SDK automatically requests the friends list using the `IFriends::RequestFriendList()` method, and then retrieves user information for each member on the list using the  `IFriends::RequestUserInformation()` method, so there is no need to request it explicitly with the steps outlined below. Moreover, the SDK receives broker notifications about any changes (addition, removal) to the friends list, keeping it up to date.
    

    That said, the only thing developers should care about is iterating over the list with the help of the `IFriends::GetFriendCount()` and `IFriends::GetFriendByIndex()` methods. The following list of actions is here for your reference only to show how the things work.

1. First, call [`IFriends::RequestFriendList()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aa648d323f3f798bbadd745bea13fc3b5).
2. After the friends list is retrieved, you can browse it by calling:
    - [`IFriends::GetFriendCount()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a8c7db81fe693c4fb8ed9bf1420393cbb), which returns the number of friends on the list (or `0` if the retrieval failed),
    - [`IFriends::GetFriendByIndex()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a07746daaec828d1d9f1e67e4ff00a02d), which returns the GalaxyID of a friend placed at a given index on the list.
3. The request for a friends list automatically calls [`IFriends::RequestUserInformation()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a4692fc9d422740da258c351a2b709526).

    !!! Note
        This call is performed automatically for friends (after requesting the list of friends) and fellow lobby members (after entering a lobby or getting a notification about some other user joining it), therefore in many cases you don’t need to call it manually. You should just wait for the appropriate callback to come to the [`IPersonaDataChangedListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IPersonaDataChangedListener.html).
    
4. Responses to `IFriends::RequestUserInformation()` come both to the [`IPersonaDataChangedListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IPersonaDataChangedListener.html) and to the [`IUserInformationRetrieveListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUserInformationRetrieveListener.html). Also, user’s avatars are downloaded, taking into account both the `avatarCriteria` argument given in `RequestUserInformation()` and the `defaultAvatarCriteria` value set by calling [`SetDefaultAvatarCriteria()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#ae9172d7880741f6dc87f9f02ff018574) (available options of avatar types are listed in the [API documentation](https://docs.gog.com/galaxyapi/group__api.html#ga51ba11764e2176ad2cc364635fac294e)).
5. After a user information is retrieved, you can read it by calling [`IFriends::GetFriendPersonaName()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aae6d3e6af5bde578b04379cf324b30c5) (not thread-safe) or [`IFriends::GetFriendPersonaNameCopy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a136fe72f661d2dff7e01708f53e3bed6). A user’s avatar can be obtained with [`IFriends::GetFriendAvatarUrl()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a4fe15f4be55cf030e12018134a281591) (not thread-safe) or [`IFriends::GetFriendAvatarUrlCopy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aac64a5c1bc9789f18763d4a29eeb172f).

## Inviting Friends

It is possible to invite your friends to play together by calling [`IFriends::ShowOverlayInviteDialog()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#ae589d534ed8846319e3a4d2f72d3c51a). This method requires the `connectionString` argument, which will allow other users to join your game. Typically, the `connectionString` looks like this:

```
-connect-lobby-<id>
```

As a result of this call, a game invitation dialog is displayed and you can invite users to your game.

If an invited user accepts the invitation, the `connectionString` is added to the command-line parameters for launching the game. In case the game is already running, the `connectionString` comes to the [`IGameInvitationReceivedListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IGameInvitationReceivedListener.html) or to the [`IGameJoinRequestedListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IGameJoinRequestedListener.html), if the invite was accepted by the user on the overlay.

!!! Important
    Before you can use this call, the GOG GALAXY Overlay must be enabled in the [GOG GALAXY client](gc-overlay.md) (it is on by default, but can be changed by the user), and then successfully injected into the game. To check whether the overlay is initialized, either call [`IUtils::GetOverlayState()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUtils.html#ace3179674f31b02b4dcd704f59c3e466) or wait for the [`IOverlayInitializationStateChangeListener::OnOverlayStateChanged()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IOverlayInitializationStateChangeListener.html#af55f5b8e0b10dde7869226c0fbd63b35) callback.

## RichPresence

The GOG GALAXY SDK allows you to set the RichPresence `status`, `metadata` and `connect` keys:

- `status` key will be visible to your friends in the chat and friends list of the GOG GALAXY client,
- `connect` key will be visible to your friends as *JOIN* button, which works exactly the same way as if a player received the game invitation sent with the [`IFriends::SendInvitation()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aefdc41edcd24ebbf88bfb25520a47397) method,
- `metadata` key will not be publicly visible, but accessible to other components of a game; for example, this key could contain player’s coordinates on a map, which then could be used to display player’s position on a mini-map on-screen.

RichPresence information can be requested with the [`RequestRichPresence`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a5a3458c1a79eb77463a60278ef7a0b5d) method. Responses come both to the [`IRichPresenceListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IRichPresenceListener.html) and the [`IRichPresenceRetrieveListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IRichPresenceRetrieveListener.html). In order to retrieve this information, use the [`GetRichPresence`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#af7d644d840aaeff4eacef1ea74838433) (not thread-safe) or the [`GetRichPresence`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a661d5d43361d708d1ed3e2e57c75315f)methods.
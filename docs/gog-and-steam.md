# Porting Your Games from Steam to GOG

Chances are you already have your game prepared for Valve’s Steam platform. The best option would be to simply push this Steam-ready build to GOG and have all implemented features working out of the box on our platform. Although we strive to make this transition as smooth as possible, the differences between the two platforms are too significant to overcome them with just a few clicks.

Here, you will find a list of items that need your attention when trying to adapt an existing Steam build to the GOG environment.

## No DRM, No Problem

GOG’s “DRM-Free” philosophy means that games released on GOG can’t have any DRM/copy protection and should work in all possible scenarios, regardless of the result of GOG GALAXY authentication, GOG GALAXY client presence or Internet connection availability. Of course, online-only features such as multiplayer, retrieving friends or leaderboards cannot be available when the user is offline, yet all other aspects of the game should not be affected, e.g. achievements functionality should be available no matter if the user is online or offline (as long as the GOG GALAXY client is installed and the user is logged in to it).

Therefore, all Steam methods related to DRM (user authentication in particular) are of no use on GOG. For more, please see the articles on [authorization in GOG GALAXY](sdk-galaxy-feats-and-states.md) and [authenticating using Steam credentials](sdk-authentication-with-other-services.md#authenticating-using-steam-credentials).

## Authentication

Speaking of user authentication: for online features to be available, a user has to be authenticated against GOG backend services. Unlike Steamworks, which relies on the Steam client to authenticate a user, the GOG GALAXY SDK requires a user to be authenticated explicitly. For this, the [`IUser::SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939) method is provided in the GOG GALAXY SDK.

There are two ways to authenticate a user:

- using the GOG GALAXY client to log in (this is the default way),

- using “Steam Encrypted App Ticket” (request acquiring ticket using Steamworks first).

As mentioned previously, authentication failure should not limit the ability of a player to play offline.

## Listeners

Like in Steamworks, to handle the results of async operations, Galaxy uses a concept of *callbacks* and *listeners*. To process the input/output data and call appropriate listeners, the [`galaxy::api::ProcessData()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7) method should be called once in a while, similarly to `SteamAPI_RunCallbacks()`.

Listeners can be of one out of the two types:

- automatically registered **global listeners**: such listeners automatically register themselves upon the creation and react on every related event until unregistered (or deleted),
- **specific listeners**: listeners for a particular method call. Unlike global listeners, they are not registered automatically. Instead, you should pass them as a parameter to a function.

Both global and specific listeners share the same interfaces, so it’s easier to use/re-use them.

### Global Listeners

Unlike Steamworks, we do not provide a macro similar to `STEAM_CALLBACK` to declare a listener. Instead, a developer has to inherit from a specific abstract class representing a listener and implement appropriate callback methods, e.g.:

```c++
// Global listener automatically registers itself for all related events upon its construction
class AuthListener : public galaxy::api::GlobalAuthListener
{
	void OnAuthSuccess() override
	{
		// Enable online features
	}

	void OnAuthFailure(galaxy::api::IAuthListener::FailureReason failureReason) override
	{
		std::cerr << "Failed to authenticate: reason=%u" << failureReason << std:endl;
	}

	void OnAuthLost() override
	{
		std::cerr << "Authentication lost" << std:endl;
	}
}

AuthListener globalAuthListener;
```

### Specific Listeners

Instead of returning a handle for an asynchronous operation, specific listeners are passed by pointer to a specific method. For example:

```c++
class AuthListener : public galaxy::api::IAuthListener
{
	void OnAuthSuccess() override
	{
		// Enable online features
	}

	void OnAuthFailure(galaxy::api::IAuthListener::FailureReason failureReason) override
	{
		std::cerr << "Failed to authenticate: reason=%u" << failureReason << std:endl;
	}

	void OnAuthLost() override
	{
		std::cerr << "Authentication lost" << std:endl;
	}
}

AuthListener specificListener;
galaxy::api::User()->SignInGalaxy(true, &specificListener);
```

It’s safe to unregister such a listener manually (by calling the [`galaxy::api::IListenerRegistrar::Unregister()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IListenerRegistrar.html#a12efe4237c46626606669fc2f22113bb) method) before it’s called by the SDK in case its result is not important anymore.

## Achievements, Leaderboards and Statistics

Most of the operations regarding achievements that are available on Steam are also accessible using the GOG GALAXY SDK with no or minimal changes. For example,

```
bool GetAchievementAndUnlockTime( const char \*pchName, bool \*pbAchieved, uint32 \*punUnlockTime )
```

Steam method is equivalent to

```
*GetAchievement ( const char \* name, bool & unlocked, uint32_t & unlockTime, GalaxyID userID = GalaxyID() )*
```

method in GOG GALAXY.

The major difference you may notice is that although there is no direct correlation between statistics and achievements both in Steam and the GOG GALAXY SDK, the former allows to set achievements progress, while the latter relies solely on stats (achievements have only a locked/unlocked state). In order to unlock an achievement, which is supposed to be based on a statistic, you need to retrieve and check the desired statistic using the [`IStats::GetStatInt()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a5a40b3ae1cd2d0cb0b8cf1f54bb8b759) or [`IStats::GetStatFloat()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a4949fc4f78cf0292dca401f6411fab5b) user statistics methods, and unlock the achievement using the [`SetAchievement()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#aa5f8d8f187ae0870b3a6cb7dd5ab60e5) method, when the desired value is reached. Then, you can set a user statistics value with the [`IStats::SetStatInt()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#adefd43488e071c40dc508d38284a1074) or [`IStats::SetStatFloat()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#ab6e6c0a170b7ffcab82f1718df355814) methods.

Because of this discrepancy, as well as no support for global achievements information and global statistics in the GOG GALAXY SDK, functionalities realized by Steam methods such as `IndicateAchievementProgress` or `GetAchievementAchievedPercent` will have to be replaced by some workarounds or removed altogether from the version to be released on GOG.

!!! Info
    Please note that “trusted” statistics (that may be set only by a game server) and server-related achievements are not yet supported by the GOG GALAXY SDK.

In case of leaderboards, the implementation is similar to the one in Steamworks: the methods for creating, finding and setting leaderboards scores are corresponding in both platforms. And, like in Steam, before working with the leaderboards, you have to find and retrieve them from the backend.

Trusted leaderboards are supported in the GOG GALAXY SDK, but there are no methods provided for resetting leaderboard entries.

For more, please see [*Statistics and Achievements*](sdk-stats-and-achievements.md) and [*Leaderboards*](sdk-leaderboards.md).

## Multiplayer

Unlike Steam, we **do not separate game sessions into pre-game lobbies and game servers**. Instead, a unified term “Lobby” exist in the GOG GALAXY SDK. Lobbies are different by the topology types used when connecting players:

- LOBBY_TOPOLOGY_TYPE_FCM
- LOBBY_TOPOLOGY_TYPE_STAR
- LOBBY_TOPOLOGY_TYPE_CONNECTIONLESS
- LOBBY_TOPOLOGY_TYPE_FCM_OWNERSHIP_TRANSITION

For most games, it is convenient to use the STAR topology, however CONNECTIONLESS lobby might be a better choice when implementing a pre-game chat-like lobby, since it doesn’t require a direct connection between users.

There is no limitation to the number of lobbies that the user is in at the same time, yet one lobby cannot host more than 250 members.

Also, **we do not expose IP addresses of users and a game host** (both user-created and dedicated servers) and use GalaxyIDs of an appropriate type (User or Lobby) instead. As a result, all methods related to retrieving GameServer information (such as `ISteamMatchmaking::GetLobbyGameServer()`) are not provided.

There is also no need to establish a connection between the peers as long as they are in the same lobby, so methods/callbacks for requesting/accepting incoming connections are not provided. Packets from unwanted sources may be simply ignored.

For more, please see [*Multiplayer*](sdk-multiplayer.md).

## Crossplay

[Crossplay](sdk-crossplay.md) is a Galaxy-specific feature that does not exist in Steamworks and allows play between Steam and Galaxy users.

## Friends

Most of the operations regarding friends interactions can be implemented in a very similar way on both platforms with analogous naming convention. Functions like [`GetFriendPersonaName`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aae6d3e6af5bde578b04379cf324b30c5) have the same meaning and result both on Steam and GOG GALAXY. The most notable differences will be that the GOG GALAXY SDK lacks the methods regarding Clan, Groups, Co-play (although some part of it might be done based on rich-presence) and Followers functionalities.

## Storage

Both GOG GALAXY and Steam provide Storage interfaces, which allow writing, uploading and sharing files among users, as well as syncing between cloud and PC.

For more, please see [*Storage*](sdk-storage.md).

## Networking

This interface is almost identical on both platforms and allows sending, checking and reading P2P packets. The main difference is that GOG GALAXY doesn’t offer Sockets and public interfaces in general as all methods are based on GalaxyID (public interfaces might be needed for custom dedicated servers).

Moreover, the GOG GALAXY SDK has two ways of checking whether any packets are available for the lobby member:

- [`INetworkingListener::OnP2PPacketAvailable()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworkingListener.html#afa7df957063fe159e005566be01b0886) can only be triggered after the callback is processed by the [`api::ProcessData()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7) method
- [`INetworking::IsP2PPacketAvailable()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html#aebff94c689ad6ad987b24f430a456677) allows to check for packets immediately after they arrive on a target machine. This may be crucial in latency-sensitive games.

## GameServer

As mentioned in Multiplayer section, game hosts in GOG GALAXY are not publicly available: their IP addresses are not exposed and the communication is based on GalaxyID. Therefore, all Steamworks methods related to GameServers, e.g. for setting map names (`ISteamGameServer::SetMapName`, `ISteamGameServer::SetServerName`), favorite game server, a friend’s game server etc., have no counterparts in the GOG GALAXY SDK.

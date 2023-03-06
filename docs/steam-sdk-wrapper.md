
# Steam SDK Wrapper (Beta)
**Keep in mind this project is a Work In Progress, with many features and improvements to come.<br>
Please leave your [feedback](https://forms.gle/3h2oULcDGaDsZKMdA).**
## Introduction

As you know from [the previous article](https://docs.gog.com/gog-and-steam/), there are some essential differences between GOG and Steam. With this tool we aim to decrease the time needed to implement additional features on GOG.  
If you already have a Steam version of your product, you can get it up and running on our platform within minutes, not hours. No API methods need to be replaced in the code — all you need is to use our Steam SDK Wrapper (Beta).

**Galaxy SDK API is still available alongside Steam SDK Wrapper functionality** so it's possible to create custom Galaxy SDK init but leave other functionality like stats or leaderboards to Steam Wrapper.

Steam SDK Wrapper (Beta) is a middle layer that intercepts Steam API calls and translates them into calls that can be understood by the GOG backends.  
Not all Steam features are supported yet — and, obviously, some will never be — but basic functionality is preserved. Currently, Steam SDK Wrapper (Beta) allows to use the following features out of the box:

- achievements,
- leaderboards,
- stats,
- friends,
- matchmaking (lobby and invites w/o lobby/user data yet)

## Offline mode

Games on GOG being DRM-free require that the game is playable even without installed Galaxy Client.
Therefore it should be possible to play the game even when `SteamAPI_Init` returns `false` which isn't the case for Steam.  
It means that in order to ensure that your game and its online features work, **you need to either**:
 
- keep Galaxy Client running 

or 

- allow your game to be able to continue running when `SteamAPI_Init` returns `false` (which will result in disabling its online features).

It's a special case that could imply game's code changes e.g only replace `SteamAPI_Init` section with `galaxy::api::Init`.
This feature is still being discussed and worked on.

DLC Discovery and Storage interface's methods using local filesystem (FileWrite/FileRead etc.) are still available even without Galaxy Client running and authorization.
## Implementation

1. Make sure you’re using a [supported Steam API version](#supported-steam-api-versions).
2. Add achievements to DevPortal, ideally using the [VDF file from Steam](https://docs.gog.com/sdk-steam-import/?h=vdf).
3. Download [Steam SDK Wrapper (Beta)](https://devportal.gog.com/galaxy/components/steam_sdk_wrapper)
4. Add Steam SDK Wrapper (Beta) to your game. There are two ways to achieve that. Do one of the following:
    1. Rename *GalaxySteamWrapper/Libraries/GalaxySteamWrapper[64].dll* to *steam_api[64].dll* and use it to replace the original *steam_api[64].dll* file in your already built game 
    2. Recompile your game linking against *GalaxySteamWrapper/Libraries/GalaxySteamWrapper[64].lib*
5. Copy *GalaxySteamWrapper/Libraries/Galaxy[64].dll* to the same directory as *steam_api[64].dll* or executable, depending on how working directory and links are handled
6. Create a [*GalaxyConfig.json*](#configuration-file) file where you specify `client_id` and either `client_secret` or `client_code` and place it in the same directory as *steam_api[64].dll* 
7. Your build (ideally no need for rebuilding if you chose **4.1**) is now ready to be uploaded to DevPortal

### Demo game

You can check our Steam Wrapper demo game and its source code for the exact implementation.
Ask our support for its license (1931358602 Steam Wrapper Demo Game).
[Check its source code](https://github.com/gogcom/steam-wrapper-demo-game) and build it yourself.

## Supported Steam API Versions

Steam SDK Wrapper (Beta) can support only specific versions of the Steam API headers and your game must be built with one of them. Currently, the supported versions are:

- **1.31** *to* **1.55**

If your game already uses one of the above, no action is necessary. Otherwise, support for other versions of the Steam API headers can be added to Steam SDK Wrapper (Beta) or you may update your current pipeline (your existing build) or set up a new one with a different Steam API version.  
Whichever you choose, you can find the list of all SteamWorks releases [here](https://partner.steamgames.com/downloads/).

## Configuration File

In order for Steam SDK Wrapper (Beta) to know the [SDK credentials](https://docs.gog.com/bc-project-properties/#sdk-credentials) of your game, they must be specified in `GalaxyConfig.json` (you can obtain them in [Devportal](https://docs.gog.com/developer-portal/#games-screen-product-buttons)). It is a flat JSON file used for configurations which should be distributed along with the game and located in its working directory — usually where the EXE file is. The only required properties are `client_id` and either `client_secret` or  `client_code`. The remaining [options](#available-options) are read and parsed during `SteamAPI_Init`.

### client_code

Plaintext `client_secret` can be used for testing purposes, but it is recommended to use `client_code` instead when releasing. `client_code` is an encrypted version of `client_secret`. You can obtain it by accessing SDK credentials in the [games menu](https://devportal.gog.com/panel/games).

### Minimal Config Example
```json
{
    "client_id": "XXXXXXXXXXXXXXXXX",
    "client_code": "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
}
```

### Available Options
| Option             | Type   | Default  | Description |
| :----------------- | :----- | :------- | :---------- |
| `client_id`        | string | required | ID of the client |
| `client_secret`    | string | `""`     | Secret of the client (**required** unless `client_code` is specified) |
| `client_code`      | string | `""`     | Code of the client (**required** unless `client_secret` is specified) |
| `config_file_path` | string | `"."`    | Path to the folder containing GOG GALAXY configuration files |
| `storage_path`     | string | `""`     | Path to the folder for storing internal SDK data |
| `host`             | string | `""`     | Local IP address this peer would bind to |
| `port`             | number | `0`      | Local port used for communication with servers and peers |
| `auth_on_init`     | bool   | `true`   | `SteamAPI_Init` will attempt blocking `SignInGalaxy` |
| `require_online`   | bool   | `false`  | Indicates if sign in with GOG GALAXY backend is required |
| `is_unity`         | bool   | `false`  | See Unity section under Game Engines |
| `dlcs`             | array  | `[]`     | Array of DlcInfo struct  `{steam_id:number, name:string, galaxy_id:number}`|

## Bindings to other programming languages

As of 1.32 flat API is supported so projects like [Steamworks.NET](https://steamworks.github.io/) or [Facepunch.Steamworks](https://wiki.facepunch.com/steamworks/) **should** work without and additional tinkering.
Same goes with other projects that utilizes `steam_api.dll`'s calls to flat API.

CSteamworks is not currently supported.

### Manual Callback Dispatch

As of version 1.1.2 of SteamWrapper, Steam's manual callback dispatch (instead of running all callbacks at once with `SteamAPI_RunCallbacks` you can dispatch them manually `SteamAPI_ManualDispatch`) is fully supported.  
Facepunch.Steamworks moved to using manual dispatch as of 2.3.0 so it should be possible to use this and later versions with SteamWrapper.

### Game engines

| Game Engine          |                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `Unreal Engine` | both native Steam implementation and EOS seems to work but a little tinkering might be needed. |
| `Unity` | appears to work without an issue |
| `Game Maker` | appears to work without an issue |
| `RenPy` | some issues regarding working directory/dll placemnt has been reported |

Most of the issues with bindings and game engines seems to be related to working directory/dll or exe placement.
We are working on a better approach regarding theses issues.

## Unity

Depending on your Steamworks implementation (Custom/SteamWorks.NET/Facepunch) you might need to set `is_unity` flag in `GalaxyConfig.json` to `true`.

## Unreal Engine

Depending on your Steamworks implementation some changes to Unreal Engine itself might be needed.

### Native C++

If your Steam implementation is in native C++, it should work like any other game so the default drag & drop method mentioned above should suffice.  

### EOS

When using EOS Online Subsystem Steam and blueprints some changes to the engine itself need to be made.  
First you need to check which Steamworks version does your version of UE support here: `/YourUnrealEnginePath/Engine/Source/ThirdParty/Steamworks/` and `Steamworks.build.cs`. For UE 4.27 it's Steamv151 so please stick to this version.
Online Subsystem is a private dependency so by default it is statically linked into your project using `steam_api[64].dll` shipped with the engine thus recompilation is necessary.  
This implies drag-and-dropping `GalaxySteamWrapper[64].dll` and `Galaxy[64].dll` e.g here `/YourUnrealEnginePath/Engine/Binaries/ThirdParty/Steamworks/Steam[Current Version]/Win64` replacing the original `steam_api[64].dll` and then recompiling your project.  
After recompilation, please make sure to ship your package with `GalaxyConfig.json` placed in `YourPacakge/YourProject/Binaries/Win[64]` beside your game's executable file and `Galaxy[64].dll` in `YourPackage/Engine/Binaries/ThirdParty/Steamworks/Steam[Current Version]`.

## Methods Implemented

**Note:** Unlisted methods are not implemented.

### SteamClient Interface

| Method                    | Supported | Remarks                                                      |
| ------------------------- | --------- | ------------------------------------------------------------ |
| `SetWarningMessageHook` | Yes       | Does the same as `SteamUtils::SetWarningMessageHook`: outputs GOG GALAXY error messages |

### SteamMatchmaking Interface

| Method                    | Supported | Remarks                                                      |
| -------------------- | --------- | --------------------------------------------------- |
| `JoinLobby`        | Yes       | |
| `RequestLobbyList` | Partially | Does not request data for each lobby automatically |
| `GetLobbyByIndex`  | Yes       |                                                     |
| `CreateLobby`      | Yes       |                                                     |

### SteamRemoteStorage Interface

| Method         | Supported | Remarks |
| -------------- | --------- | ------- |
| `FileExists` | Yes       |         |
| `FileRead`     | Yes       |                                                              |
| `FileWrite`    | Partially | No checking for reaching file limits or invalid paths/filenames |
| `GetFileCount` | Yes       |                                                              |
| `GetFileSize`  | Yes       |                                                              |
| `GetQuota`     | No        | |

### SteamUser Interface

| Method                        | Supported | Remarks                                  |
| ---------------------------- | -------- | --------------------------------------- |
| `BLoggedOn`                 | Yes       |  |
| `RequestEncryptedAppTicket` | Yes       |                                          |

### SteamUserStats Interface

| Method                                 | Supported | Remarks                                                      |
| -------------------------------------- | --------- | ------------------------------------------------------------ |
| `RequestCurrentStats`                | Yes       |                                                              |
| `GetStat (integer)`                  | Yes       |                                                              |
| `GetStat (float)`                    | Yes       |                                                              |
| `SetStat (integer)`                  | Yes       |                                                              |
| `SetStat (float)`                    | Yes       |                                                              |
| `UpdateAvgRateStat`                  | Yes       |                                                              |
| `GetAchievement`                     | Yes       |                                                              |
| `SetAchievement`                     | Yes       |                                                              |
| `ClearAchievements`                  | Yes       |                                                              |
| `GetAchievementAndUnlockTime`        | Yes       |                                                              |
| `StoreStats`                         | Yes       |                                                              |
| `GetAchievementDisplayAttribute`     | Yes       |                                                              |
| `FindOrCreateLeaderboard`            | Yes       |                                                              |
| `FindLeaderboard`                    | Yes       |                                                              |
| `GetLeaderboardName`                 | Yes       |                                                              |
| `GetLeaderboardEntryCount`           | Yes       |                                                              |
| `GetLeaderboardSortMethod`           | Yes       |                                                              |
| `GetLeaderboardDisplayType`          | Yes       |                                                              |
| `DownloadLeaderboardEntries`         | Yes       | Cannot be called with `k_ELeaderboardDataRequestUsers`, as in Steam |
| `DownloadLeaderboardEntriesForUsers` | Yes       |                                                              |
| `GetDownloadedLeaderboardEntry`      | Yes       |                                                              |
| `UploadLeaderboardScore`             | Yes       |                                                              |

### SteamUtils Interface

| Method                  | Supported | Remarks                                                      |
| :---------------------- | :-------- | :----------------------------------------------------------- |
| `SetWarningMessageHook` | Yes       | Does the same as `SteamClient::SetWarningMessageHook`: outputs GOG GALAXY error messages |

### SteamFriends Interface

| Method                    | Supported | Remarks                                                      |
| ------------------------- | --------- | ------------------------------------------------------------ |
| `GetClanActivityCounts`           | No         | Always sets arguments to 0                                  |
| `GetFriendsGroupName`             | Simplified | Assumes everybody is in a single group, always returns `Friends`. |
| `GetFriendPersonaNameHistory`     | Simplified | Assumes no history exists                                    |
| `GetPlayerNickname`               | Simplified | Assumes no nickname is set                                   |
| `GetFriendsGroupCount`            | Simplified | Groups are not supported; assumes everybody is in a single group |
| `GetFriendCount`                  | Partially  | In the friend flags passed to this method, only `k_EFriendFlagImmediate` is supported |
| `GetFriendByIndex`                | Partially  | In the friend flags passed to this method, only `k_EFriendFlagImmediate` is supported |
| `HasFriend`                       | Partially  | In the friend flags passed to this method, only `k_EFriendFlagImmediate` is supported |
| `SetRichPresence`                 | Partially  | No custom keys, only `status`, `metadata` and `connect` allowed |
| `GetFriendRichPresence`           | Partially  | No custom keys, only `status`, `metadata` and `connect` allowed |
| `GetFriendSteamLevel`             | No         | Returns `0`                                                   |
| `GetClanCount`                    | Simplified | Returns `0`                                                   |
| `GetFriendsGroupMembersCount`     | Partially  | Returns friend count regardless of a group                   |
| `GetFriendsGroupMembersList`      | Partially  | Returns friend list regardless of a group                    |
| `GetFriendRelationship`           | Partially  | Returns only `k_EFriendRelationshipFriend` or `k_EFriendRelationshipNone` |
| `GetPersonaState`                 | Partially  | Returns only `k_EPersonaStateOnline` or `k_EPersonaStateOffline` |
| `GetFriendPersonaState`           | Partially  | Returns only `k_EPersonaStateOnline` or `k_EPersonaStateOffline` |
| `GetUserRestrictions`             | Partially  | Returns only `k_nUserRestrictionNone` or `k_nUserRestrictionUnknown` |
| `GetPersonaName`                  | Yes        |                                                              |
| `SetPersonaName`                  | No         |                                                              |
| `GetFriendPersonaName`            | Yes        |                                                              |
| `GetFriendGamePlayed`             | No         |                                                              |
| `GetFriendsGroupIDByIndex`        | No         |                                                              |
| `GetClanByIndex`                  | No         |                                                              |
| `GetClanName`                     | No         |                                                              |
| `GetClanTag`                      | No         |                                                              |
| `DownloadClanActivityCounts`      | No         |                                                              |
| `SetInGameVoiceSpeaking`          | No         |                                                              |
| `SetPlayedWith`                   | No         |                                                              |
| `GetSmallFriendAvatar`            | Yes        |                                                              |
| `GetMediumFriendAvatar`           | Yes        |                                                              |
| `GetLargeFriendAvatar`            | Yes        |                                                              |
| `RequestUserInformation`          | Yes        |                                                              |
| `RequestClanOfficerList`          | No         |                                                              |
| `GetClanOwner`                    | No         |                                                              |
| `GetClanOfficerCount`             | No         |                                                              |
| `GetClanOfficerByIndex`           | No         |                                                              |
| `ClearRichPresence`               | Yes        |                                                              |
| `GetFriendRichPresenceKeyCount`   | Yes        |                                                              |
| `GetFriendRichPresenceKeyByIndex` | Yes        |                                                              |
| `RequestFriendRichPresence`       | Yes        |                                                              |
| `GetCoplayFriendCount`            | No         |                                                              |
| `GetCoplayFriend`                 | No         |                                                              |
| `GetFriendCoplayTime`             | No         |                                                              |
| `GetFriendCoplayGame`             | No         |                                                              |
| `GetFollowerCount`                | No         |                                                              |
| `IsFollowing`                     | No         |                                                              |
| `EnumerateFollowingList`          | No         |                                                              |
| `IsClanPublic`                    | No         |                                                              |
| `IsClanOfficialGameGroup`         | No         |                                                              |

### Flat API

From version 1.32 the flat API functions shipped with Steamworks are supported with some exceptions:

| Method                    |
| ------------------------- |
| `DestructISteamHTMLSurface` |
| `RequestPublisherOwnedAppData` |
| `GetPublisherOwnedAppData` |
| `Set_SteamAPI_CPostAPIResultInProcess` |
| `Remove_SteamAPI_CPostAPIResultInProcess` |
| `RunFrame` |
| `Set_SteamAPI_CCheckCallbackRegisteredInProcess` |
| `SetXboxPairwiseID` |
| `GetXboxPairwiseID` |
| `SetPSNID` |
| `GetPSNID` |
| `SetStadiaID` |
| `GetStadiaID` |
| `ShowModalGamepadTextInput` |
| `GetConfigValueInfo` |
| `IterateGenericEditableConfigValues` |
| `ResetIdentity` |
| `SetIPv4Addr` |
| `GetFakeIPType` |
| `GetIPv4` |
| `IsFakeIP` |

Also SteamTV and SteamVR interfaces are not supported

**Keep in mind this project is a Work In Progress, with many features and improvements to come.<br>
Please leave your [feedback](https://forms.gle/3h2oULcDGaDsZKMdA).**

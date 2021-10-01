# GOG GALAXY FAQ

**Frequently asked questions about the GOG GALAXY SDK integration**

When you run into any problem, please first update to the [newest version of the GOG GALAXY SDK](https://devportal.gog.com/panel/sdk). Also, please make sure that you are using the latest production version of our [GOG GALAXY client](https://www.gog.com/galaxy) (after installing open it, let it update itself and then sign in). We rely on our users keeping the GOG GALAXY client up to date, as it provides the most important components of the platform.

For any questions that are not covered by this FAQ, please contact [our support team](mailto:support@developer.gog.com), or read the [SDK documentation](sdk.md).

## Builds

1. **I want to send a preview build to GOG, how should I do this?**

    Please refer to the [*Games in Development*](games-in-development.md) article for more information.

2. **Do you add your own installer?**

    Yes, we add our own installer. It will be created after you upload your game build with Build Creator or Pipeline Builder and let us know it awaits our QA process. Read more in [*Offline Installers*](offline-installers.md).

3. **How do build updates get delivered and published? How can I add a changelog?**

    You upload your build, using Build Creator or Pipeline Builder, and then publish the updated build on the Master branch. After that, you can create and add a changelog to the build. See [*Updates*](updates.md).

4. **How do I set up downloadable content (DLC) for my game?**

    DLCs are either bundled with your base game during preparing game build in Build Creator or Pipeline Builder, or uploaded and released independently. See [*DLCs and Extras*](dlc-and-extras.md) and contact your GOG Product Manager.

## General SDK Questions

1. **How can I invite my team members to the Devportal?**

    In the Devportal, click *Users→Invite*. More in [*User Management*](user-management.md).

2. **Do you have an example (a source code) of implementing the GOG GALAXY SDK in a game?**

    Yes, it’s written in C# and you will find it in [our GitHub repository](https://github.com/gogcom/galaxy-csharp-demo-game/). See [*C# Demo Game*](demo-desc-and-prerequisities.md).

3. **I keep getting authorization and/or [`galaxy::api::SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939) errors**

    Make sure you have the necessary privileges (license) for your game in the [User Management](user-management.md) section of the Developer Portal – the GOG GALAXY SDK will not initialize if your account does not have a valid license for the game. If your account is ran by your publisher, please contact its administrator. Please check whether your GOG GALAXY client is running and you are logged in. If your problems persist, please contact [our support team](mailto:support@developer.gog.com).

4. **What are the `client_id` and `client_secret` parameters and where can I get them for my game?**

    `Client_id` is a unique identifier of your game in GOG GALAXY. `Client_id` and `client_secret` are used for initializing GOG GALAXY during the [`galaxy::api::Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) call. You can find them in the *SDK Credentials* button for a given game in the *Games* section of the [Developer Portal](developer-portal.md).

5. **How often should I call `ProcessData()`?**

    [`ProcessData`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7) should be called frequently. It is designed to impose a minimal overhead (a couple of microseconds at most). Calling it once a few seconds should be enough, although you may call it every frame, if needed.

6. **Should I obfuscate the `client_id` and `client_secret` in the game code?**

    It is up to you. However, we believe that no matter how hard we obfuscate the credentials, if someone is really determined to get them, they will find a way.

7. **Can the game run multiple processes with multiple active connections to the GOG GALAXY network at the same time? With each process having its own `Init()` and [`SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939)?**

    In general it is OK but the separate client applications that were signed in to the same account will not be able to see each other’s multiplayer lobbies, messages, etc.

8. **I couldn’t find any server-side APIs for authenticating a user. Other platforms let you authenticate a user via encrypted tickets by making API calls to verify that a user has signed in and owns the game. Is there an equivalent for GOG GALAXY?**

    Sure! We do have [encrypted app tickets](sdk-encrypted-tickets.md) allowing authentication in a third-party backend using GOG GALAXY service.

9. **What data does the encrypted app ticket contain?**

    We encrypt following data inside the ticket:

    - user_id

    - client_id

    - timestamp of ticket generation

    - some additional data that will be passed to the ticket request.

10. **Technically, you could prevent the game from starting, if the user is not signed in to GOG GALAXY. Is this considered a bad practice and does it violate GOG’s “DRM free” principle?**

    The single player mode should work regardless of whether the user is using GOG GALAXY or not (or whether they are currently online or not). The current user status in our GOG GALAXY SDK can be used for example to enable/disable the multiplayer option in the main menu.

11. **What should happen, if the user is not signed in to GOG GALAXY when the game starts?**

     We don’t have any mandatory GOG GALAXY messaging in-game. For example, in *The Witcher Adventure Game*, if you’re not using GOG GALAXY, the multiplayer option in the main menu is just disabled. Of course, if you’d like to add some more info in this case — that’s great, we’d just like to get a preview of this screen.

12. **The game considers me logged in even after I close the GOG GALAXY client.**

     This is by design.

13. **Is it necessary to check for the presence of an installed GOG GALAXY client or is it safe to call the API functions regardless?**

     If the GOG GALAXY client is not present (or the user is not logged in), the methods related to achievements will throw exceptions that you should catch. You can also disable all exceptions in our SDK when making the `Init()` call. Be sure to check that your game does not crash when making a call to any of the GOG GALAXY components when GOG GALAXY was not initialized.

14. **Does [`GetError()`](https://docs.gog.com/galaxyapi/group__api.html#ga11169dd939f560d09704770a1ba4612b) depend on the  `throwExceptions` flag of the `Init()` function or does this always work? And does it return the same error as when it’s thrown as an exception?**

     This flag of the `Init()` function is not supported anymore. If there is any error, `GetError()` will return it (or `NULL` if no error occurred).

     If you want to throw exceptions, use the `ThrowIfGalaxyError()` function from [GalaxyExceptionHelper.h](https://docs.gog.com/galaxyapi/GalaxyExceptionHelper_8h_source.html).

15. **Do I need to put the *GalaxyPeer.dll* file into the game?**

     No. In the present version of the SDK, the *GalaxyPeer.dll* file is loaded from the GOG GALAXY client redistributables. You have to place this file manually only in case of very old SDK versions: 1.114.12 and older. Please refer to the [API documentation](https://docs.gog.com/galaxyapi/index.html#getting-started).

16. **How to get the player display name? The GOG GALAXY [encrypted app ticket](sdk-encrypted-tickets.md) does not contain it.**

     [`galaxy::api::Friends()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga27e073630c24a0ed20def15bdaf1aa52)`->`[`GetPersonaName()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#a3341601932e0f6e14874bb9312c09c1a) once you are logged in.

17. **Can I use `IsLoggedOn()` to determine the current status of the connection with the backends?**

     If [`IsLoggedOn`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a3e373012e77fd2baf915062d9e0c05b3) returns `true`, it means that the online authentication was successful, but it does not mean that you are still connected to the backends.

     Fortunately, there’s a special method for checking the connection status: [`GogServicesConnectionState`](https://docs.gog.com/galaxyapi/group__api.html#ga1e44fb253de3879ddbfae2977049396e) with its [dedicated listener](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IGogServicesConnectionStateListener.html).

18. **What is the difference between OPERATIONAL_STATE_SIGNED_IN and OPERATIONAL_STATE_LOGGED_ON?**

     We have something that is called OfflineMode. If the user was signed in previously and now they do not have an internet connection, then you can still call [`OnAuthSuccess`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#aa824226d05ff7e738df8e8bef62dbf69), but you will get a different operational state.

     OPERATIONAL_STATE_SIGNED_IN, but not OPERATIONAL_STATE_LOGGED_ON means that we know who the user is (SIGNED_IN), but is not authenticated with our backend (not LOGGED_ON). Learn more on this in the article on [authorization in GOG GALAXY](sdk-galaxy-feats-and-states.md).

     If the user has downloaded statistics, achievements and leaderboards before, while being online, and now they don’t have an internet connection, they can still:

     - unlock achievements,

     - update a statistics value,

     - set a leaderboard score.

     The data will be synchronized once you run the game again with the internet connection re-established.

19. **Do you provide a NAT punchthrough and proxy functionality?**

     Yes, they are automatically handled in the SDK. If we can’t pass the NAT, then we connect the player through one of our proxy servers.

20. **How does the `galaxy::api::IsDlcInstalled()` method work?**

     The GOG GALAXY SDK allows you to check if a DLC has been installed. You can use the [`galaxy::api::IsDlcInstalled()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a46fbdec6ec2e1b6d1a1625ba157d3aa2) method, supplying the DLC Product ID as a parameter. This call is not dependent on the Operational State of the Galaxy Peer, provided the [`galaxy::api::Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) call has been made. This is regardless of if the `galaxy::api::Init()` call was successful or resulted in an error.

    !!! Tip
        `IsDlcInstalled()` only requires the `galaxy::api::Init()` call to be made, regardless of its result. The method determines if the DLC has been installed by checking a metadata file created when the game is installed through the GOG GALAXY client.

        To successfully make this call:
        
        1. Install the GOG GALAXY client.
        2. Make the game available on a branch in the GOG GALAXY client (either publish it through the developer pipeline on a beta branch or contact your Product Manager).
        3. Install the game with the GOG GALAXY client.


## Achievements, Stats, Leaderboards

1. **Are there recommendations or restrictions to consider for the achievement icons?**

    You should upload the best quality achievement icons in the Developer Portal: go to *Games* page, click the *Galaxy Features* button for a particular game, select *Achievements* in the list. In the resulting *Achievements* screen, select files for Unlocked/Locked icons for a given achievement, if you have already defined achievements, or click the green *Add new* button, then define achievements and select their icons. Maximum single icon size is 2 MB, allowed image types: JPEG, PNG. We handle any resizing/formatting internally.
   
    In the [*Achievements*](achievements.md) article you will find all details on achievements in the Developer Portal.
   
2. **How should I implement offline achievements?**

    If you implemented achievements, offline achievements will work automatically as long as you don’t disable them. Two things to take into consideration:

    - It is important not to disable GOG GALAXY Features if [`AuthListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html) returns `OnAuthSuccess`
    - `IsLoggedOn` (which gives you info about backend connection) returns `true` for online mode and `false` for offline mode, so you shouldn’t disable GOG GALAXY Features based on that.

3. **Setting Achievements does not work — I call the appropriate methods and nothing happens.**

    - Have you called [`RequestUserStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a38f5c146772f06dfd58c21ca599d7c25) before calling any other achievement methods?
    - Have you called [`StoreStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a0e7f8f26b1825f6ccfb4dc26482b97ee) after calling [`SetAchievement()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#aa5f8d8f187ae0870b3a6cb7dd5ab60e5) to commit the changes?
    - If you did all of the above, then you can try (if you haven’t already) to set up listeners and use [`GetError()`](https://docs.gog.com/galaxyapi/group__api.html#ga11169dd939f560d09704770a1ba4612b) to see if some exception occurred. Please consult with the SDK documentation for details.

4. **Trying to set Achievements or Statistics throws an unhandled exception and crashes the game!**

    - Do you use exceptions? They are available with the `ThrowIfGalaxyError()` method from  [`GalaxyExceptionHelper.h`](https://docs.gog.com/galaxyapi/GalaxyExceptionHelper_8h_source.html).
       - If so, do you catch exceptions after every GOG GALAXY method call?
       - If not, do you check [`galaxy::api::GetError()`](https://docs.gog.com/galaxyapi/group__api.html#ga11169dd939f560d09704770a1ba4612b) after every GOG GALAXY method?
    - Did you use the [`RequestUserStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a38f5c146772f06dfd58c21ca599d7c25) method and waited for an appropriate callback? The documentation clearly states that you cannot use [`SetAchievement()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#aa5f8d8f187ae0870b3a6cb7dd5ab60e5) and [`SetStatInt()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#adefd43488e071c40dc508d38284a1074) before retrieving the achievements and statistics for the game from our backend.

5. **Can you reset the leaderboards using a GOG GALAXY SDK call?**

    No. This feature would possibly allow any user to reset the entire leaderboard.

6. **The leaderboards do not work. Why?**

    Please check if you implemented a proper flow for leaderboards:

      1. [`RequestLeaderboards()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a7290943ee81882d006239f103d523a1d) — to download leaderboard definitions.
      2. [`RequestLeaderboardEntriesAroundUser()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a1215ca0927c038ed5380d23d1825d92d) or [`RequestLeaderboardEntriesForUsers()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#ac06a171edf21bb4b2b1b73db0d3ba994) or  [`RequestLeaderboardEntriesGlobal()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a8001acf6133977206aae970b04e82433) — these will get you entries in an existing leaderboard, and will work only if the call from the first step was successful (check this with a callback listener).

      3. [`GetRequestedLeaderboardEntry()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a2a83a60778dafd8375aa7c477d9f7bff) — this will get you a specific entry in a leaderboard and will work only if the previous steps were successful.

      Also, are you using listeners properly to check for errors and call [`ProcessData`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7)?

7. **Does the GOG GALAXY SDK support creating leaderboards during runtime?**

    Yes! Read about the [`FindOrCreateLeaderboard()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a1854172caa8de815218a0c44e2d04d8c) call in the documentation.

## Multiplayer

1. **I have my own back-end servers for multiplayer. Can I use them with GOG GALAXY?**

    Yes! We offer an [encrypted app ticket](sdk-encrypted-tickets.md) feature so that you can use the GOG GALAXY SDK login and authenticate users on your own backend server.

2. **Is the [`galaxy::api::Networking()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html) the only method to send and retrieve custom packets?**

    Yes, you have to implement the GOG GALAXY SDK `Networking()` method to use our NAT punchthrough/UDP proxy servers.

3. **Is there a way to get the remote server IP and port via GOG GALAXY, like you can in Steam?**

    No, there is no way to retrieve the IP/port pair internally using the SDK method calls.

4. **What port does the GOG GALAXY remote server use?**

    It’s a random port from the range of 1024-65535.

5. **Does the GOG GALAXY SDK support anonymous login?**

    Yes, we support anonymous login for dedicated [`GameServers`](https://docs.gog.com/galaxyapi/group__GameServer.html) only with the [`SignInAnonymous()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a9d4af026704c4c221d9202e2d555e2f1) method.

6. **In case of Steam, we can use the `SteamAPI_IsSteamRunning` call to check whether Steam is running. Is there a similar call for Galaxy?**

    No, we only check if you’re signed in to GOG GALAXY at game startup. After that, logging out does not affect the SDK while the game is still running. We only have the [`IsLoggedOn()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a3e373012e77fd2baf915062d9e0c05b3) call to check for the connection status, but this does not affect the GOG GALAXY client, only the connection of the GOG GALAXY libraries to our backends.

## Crossplay

1. **Could you explain to me how the crossplay feature works?**
   
    Each platform has its own multiplayer environment, and it is not possible to access one from the other. The way crossplay works is that you implement the networking stack from the GOG GALAXY SDK in your game and upload the builds to GOG and other platforms via their developer pipeline. Thanks to this, game builds on different platforms use the same (GOG) environment and are able to interact with each other — for example, users on different services are able to play with each other, because GOG GALAXY servers are handling everything. In other words, even if a game is published on Steam, for instance, it uses GOG GALAXY servers for all interactions. We are able to handle Steam app tickets for users logging in to GOG GALAXY through the Valve’s client. See [*Crossplay*](sdk-crossplay.md).
   
2. **What do you mean exactly by Steam user authentication? Do you mean GOG players being able to play with their friends on Steam?**

    ***GOG Authorization:***

    ```
    galaxy::api::Init(clientID, clientSecret)
    galaxy::api::User()->SignInGalaxy();
    ```

    It signs you in using credentials stored in the GOG GALAXY client (you need to have the client installed and be signed in).

    ***Steam Authorization:***

    ```
    galaxy::api::Init(clientID, clientSecret)
    galaxy::api::User()->SignInSteam(const char* steamAppTicket, uint32_t steamAppTicketLength, const char* personaName);
    ```

    where `steamAppTicket`, `steamAppTicketLength`, `personaName` are taken from the Steam SDK.

    You also need to have *GalaxyPeer.dll* placed along in the game folder.

    In both cases you should wait (and call [`galaxy::api::ProcessData`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7) ) until you get the [`AuthListener::OnAuthSuccess`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#aa824226d05ff7e738df8e8bef62dbf69) (or [`AuthListener::OnAuthFailure`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#ae6242dab20e1267e8bb4c5b4d9570300)) callback.

    ***GOG Multiplayer:***

    After successfully signing in (to GOG or Steam), you can use the [`galaxy::api::Matchmaking()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html) interface to create/list/join GOG lobbies and the [`galaxy::api::Networking()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html) interface to send packets of data inside a lobby.

3. **How do I sign in to GOG GALAXY when running the game in the Steam Client (for Crossplay purposes)?**

    You need to use Steam API to extract the user details and then pass them over to the GOG GALAXY SDK for creating a user account (if it doesn’t exist) or authenticating an existing user through [`Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) and [`SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939).

4. **Do you support Steam friend invites for games hosted using GOG GALAXY multiplayer?**

    Steam offers 2 types of invites:

     - Invitation to a lobby, which uses the Steam Overlay:

        ```
        SteamFriends()->ActivateGameOverlayInviteDialog( CSteamID steamIDLobby )
        ```

        we have no solution for this scenario yet.

     - Inviting users with a connection string:

        ```
        SteamFriends()->InviteUserToGame( CSteamID steamIDFriend, const char *pchConnectString )
        ```

        This will work by default, for example `connect-galaxy-lobby=1234`, but it requires the GUI for invitation to be done in-game.

1. **When using the GOG GALAXY matchmaking service without an associated GOG GALAXY account (logged in through Steam, using the GOG GALAXY Crossplay), what in-game name will be displayed for me?**

    You have to pass the Steam nickname as a parameter when calling [`galaxy::api::SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939) (as shown in the documentation).

## Unity/C# Wrapper SDK

1. **Which files should I use if I want to use the GOG GALAXY C# wrapper for SDK integration on Windows?**

    Place *Galaxy64.dll* or *Galaxy.dll* (from C++ SDK), *GalaxyCSharp.dll* and *GalaxyCSharpGlue.dll* in the game folder and it should work correctly. Just make sure all these files come from the same version of the SDK.

2. **I can’t find some of the classes and methods of the GOG GALAXY SDK using the C# wrapper.**

    Try using `Galaxy.Api.GalaxyInstance`.

3. **Unity on OSX allows me to only add .bundle files. How can I import the .dylibs?**

    Try renaming *libGalaxyCSharpGlue.dylib* to *GalaxyCSharpGlue.bundle*.

4. **GOG GALAXY does not work for me inside the Unity debugger or in Unity standalone builds.**

    GOG GALAXY SDK Unity Package should take care of placing the required libraries in their place both when using the editor and when building a standalone game. GOG GALAXY SDK Unity package can be found on our [Developer Portal](https://devportal.gog.com/galaxy/components/sdk).

    If you are using the GOG GALAXY SDK Unity package and the GOG GALAXY API is not available in the Unity editor, please make sure that the *Galaxy.dll* or *Galaxy64.dll* file is in the project’s root folder.

5. **GOG GALAXY throws an exception when calling the `Init()` method after stopping and starting the Unity3d debugger.**

    It is because `Shutdown()` is not called by Unity when the application is being stopped. You can fix this by adding calling `GalaxyInstance.Shutdown()` in `OnDestroy/Dispose` methods. Note that before calling `Shutdown()`, all the listeners should be disposed of.

## Overlay

1. **Can I implement the GOG GALAXY Overlay in our game?**
   
    The overlay is ready for integration. See [*GOG GALAXY Overlay*](gc-overlay.md).
   
2. **The overlay behaves oddly in some Unity games on macOS. What is the reason?**

    Recent changes in Unity make the overlay attach to the wrong process, and — as a result — appear in the wrong window. Unfortunately, we are unable to fix this at the moment.

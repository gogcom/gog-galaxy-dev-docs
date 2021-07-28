# GOG GALAXY SDK

This development kit is available on our [GOG GALAXY Developer Portal](https://devportal.gog.com/galaxy/components/sdk) and it allows your users to access online features such as achievements, leaderboards, multiplayer, storage and more.

The GOG GALAXY SDK is compatible with the [GOG GALAXY client](gc-client-overview.md) and the [GOG GALAXY overlay](gc-overlay.md). The GOG GALAXY client will display all of the unlocked achievements, and the overlay will display a notification whenever you unlock the achievement in a game. The overlay can also be used to send and receive invitations to online multiplayer games.

## DRM and SDK

One of the main features of games available on GOG.com is that they are provided DRM-free. This was taken into consideration when creating both the GOG GALAXY client and the SDK. The GOG GALAXY SDK provides many of its features even when the user is not signed in, or when the Internet connection is disabled. For example, when the Internet connection is not available, players are not able to host or join online games, but they still can unlock achievements, set their statistics or leaderboard scores. All changes that were made while offline are stored locally on users’ machines and are uploaded to the GOG backend once an Internet connection is reestablished.

There are three main concepts you should be aware of when implementing the GOG GALAXY SDK:

- The GOG GALAXY client should not be required to launch the game. A failed [`IGalaxy::Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) call, which tries to initialize the client, should not stop the game from loading, but rather just disable the game modules that require it — like achievements, leaderboards, statistics or multiplayer.
- If the GOG GALAXY client is installed and running, player authentication in our servers using the [`IUser::SignInGalaxy()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) call should be entirely optional, too. That is because we do not require active connection between the client and our servers. Once you have downloaded the game to your local drive, you should be able to play it even in the middle of the Amazon rainforest — provided you have an access to electricity, of course.
- Should a user lose their connection to the GOG GALAXY servers, the game should seamlessly switch to an offline mode. With achievements, however, we include a local cache that still allows the user to unlock them and synchronize with the servers when the connection is restored.

In the next chapters we show you what features our SDK offers, as well as how to implement and test them. We will also demonstrate the SDK in combat: you can check our sample game created in Unity with the GOG GALAXY SDK implemented in C#.
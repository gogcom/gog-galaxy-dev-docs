# Crossplay

The GOG GALAXY SDK provides a **crossplay** feature allowing users to create and join each other’s multiplayer games, regardless of what platform they use. Our SDK allows connecting users on Windows/MacOS/Linux PCs with users on consoles. It is required to implement the GOG GALAXY SDK multiplayer in builds for all the platforms of the game for a crossplay to work.

The general rule of thumb is to set up and receive proper credentials and certificates for your game in an external platform, then provide the necessary information to us, so that we can modify the ClientID properties of your product in GOG GALAXY environment.

## How Does It Work?

Each platform has its own multiplayer environment, and it is not possible to access one from the other. The way crossplay works is that you implement the networking stack from the GOG GALAXY SDK in your game and upload the builds to GOG and other platforms via their developer pipeline. Thanks to this, game builds on different platforms use the same (GOG) environment and are able to interact with each other — for example, users on different services are able to play with each other, because GOG GALAXY servers are handling everything.

## Implementing Crossplay

To allow multiplayer connections between GOG and Steam users, you need to add both [GOG](sdk-galaxy-feats-and-states.md) and [Steam](sdk-authentication-with-other-services.md#authenticating-using-steam-credentials) (or any platform you want to integrate) authorization methods to your build, as well as implement the GOG GALAXY [Multiplayer](sdk-multiplayer.md) feature. You may create separate builds of your game, one for each platform, then every build would perform authorization process according to the platform, but all builds would use the GOG GALAXY SDK for Multiplayer.

Also, remember to:

1. Put *Galaxy64.dll* (or *Galaxy.dll*) in the game folder.
2. Wait (and call [`galaxy::api::ProcessData`](https://docs.gog.com/galaxyapi/group__Peer.html#ga1e437567d7fb43c9845809b22c567ca7)) until you receive the [`AuthListener::OnAuthSuccess`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#aa824226d05ff7e738df8e8bef62dbf69) (or [`AuthListener::OnAuthFailure`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#ae6242dab20e1267e8bb4c5b4d9570300)) callback.

### GOG Multiplayer

After successfully signing in to GOG GALAXY, you can use the [`galaxy::api::Matchmaking()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga4d96db1436295a3104a435b3afd4eeb8) interface to create/list or join lobbies and the [`galaxy::api::Networking()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga269daf0c541ae9d76f6a27f293804677) interface to send packets of data inside a lobby. See [*Multiplayer*](sdk-multiplayer.md).

## Crossplay and Game Invites

Inviting GOG Friends to multiplayer lobbies is possible and fully supported, but available only when a user is logged in the GOG GALAXY client. The user is able to invite their GOG Friends only, either via the GOG GALAXY Overlay or directly from the game, using the GOG GALAXY SDK methods.

Inviting Steam friends to a multiplayer game is not supported by the GOG GALAXY SDK, but should be possible by utilizing both the Steam SDK (for handling invitations) and the GOG GALAXY SDK (for handling [matchmaking and networking](sdk-multiplayer.md)). Of course, this would only be used when a user is logged in to Steam, and it would allow a user to invite their Steam friends only.
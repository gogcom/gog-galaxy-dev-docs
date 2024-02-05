# General Information

The GOG GALAXY SDK requires users to be authenticated in order to use online features such as [Achievements](sdk-stats-and-achievements.md), [Multiplayer](sdk-multiplayer.md) or [Friends](sdk-friends.md). It allows to do so by providing several authorization methods relying either on the GOG GALAXY Backend and GOG GALAXY Client, or on other services, e.g. Steam.

Additionally, you may use the GOG GALAXY SDK as an identity provider in your own services, relying on it to check user authentication and licenses for purchased games. More detailed information can be found in the next article, [*User Authentication Based On Other Services*](sdk-authentication-with-other-services.md).

Before you start implementing desired authorization methods, please read about GOG GALAXY operational states and how they affect the GOG GALAXY SDK features.

## Operational States

When implementing the GOG GALAXY SDK features, you need to pay special attention to these two methods:

| Method                                                       | Description                                                  |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [`galaxy::api::Init(initOptions)`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) | Initializes the GOG GALAXY Peer with specified client (game) credentials. It succeeds when the GOG GALAXY client is installed and running. You need to call `Init()` before calling any other GOG GALAXY SDK function |
| [`galaxy::api::IUser::SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939) | Allows to sign in the user based on the GOG GALAXY client authentication |

Based on their results, GOG GALAXY sets up two statuses:

| Status     | Method to Check the Status                                   | Description                                                  |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `SignedIn` | [`galaxy::api::User()::SignedIn()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#aa6c13795a19e7dae09674048394fc650) | Determines whether the user has a license for the game and is logged in to the GOG GALAXY client locally |
| `LoggedOn` | [`galaxy::api::User::IsLoggedOn()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a3e373012e77fd2baf915062d9e0c05b3) | Determines if the user has a license for the game, is logged in to the GOG GALAXY client and is connected to the GOG GALAXY Backend Services |

The table below shows how GOG GALAXY operational state depends on `SignedIn` and `LoggedOn` values:

![Operational States](_assets/sdk-operational-states.png)

## Features Availability vs. Operational States

This table shows how GOG GALAXY features depend on GOG GALAXY operational states. Please note that if the `SignedIn()` method returns `false`, achievements, leaderboards, networking and matchmaking services should not be called.

Interfaces that can be called — if used — regardless of GOG GALAXY operational states are the [`IApps`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html) (including the [`IsDlcInstalled(ProductID productID)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a46fbdec6ec2e1b6d1a1625ba157d3aa2) method) and the [`IStorage`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html) interface methods (without the [`FileShare()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a10cfbb334ff48fcb8c9f891adc45ca1d) method). These can be called only after calling the `Init()` method, but are not dependent on its result. This means that the `IApps` and the `IStorage` interfaces are available even if the `Init()` call fails.

| **Feature / Scenario**            | **No GOG GALAXY or No User** | **GOG GALAXY: Installed and running, game known in the client but the user doesn’t own it** | **GOG GALAXY: Installed and running, game launched externally and the user has a license** | **GOG GALAXY: Installed and running, game launched in the client and the user has a license** | **GOG GALAXY: Installed and running, game launched offline, the user has a license** | **Other platforms (crossplay)** |
| --------------------------------- | :--------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------------------------: |
| **SignedIn**                      |          **FALSE**           |                          **FALSE**                           |                           **TRUE**                           |                           **TRUE**                           |                           **TRUE**                           |            **TRUE**             |
| **LoggedOn**                      |          **FALSE**           |                          **FALSE**                           |                           **TRUE**                           |                           **TRUE**                           |                          **FALSE**                           |            **TRUE**             |
| **Multiplayer**                   |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✗                               |                ✔︎                |
| **Achievements**                  |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                ✗                |
| **Leaderboards**                  |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                ✗                |
| **Stats**                         |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                ✗                |
| **Rich presence**                 |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✗                               |                ✗                |
| **Overlay**                       |              ✗               |                              ✗                               |                              ✗                               |                              ✔︎                               |                            ✔︎[^1]                             |                ✗                |
| **Game time tracking**            |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✗                               |                ✗                |
| **DLC and language discovery** [^2] |            ✔︎               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                ✗                |
| **Cloud Storage — local storage** |              ✔︎               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                              ✔︎                               |                ✗                |
| **Cloud Storage — cloud sync**    |              ✗               |                              ✗                               |                              ✔︎                               |                              ✔︎                               |                              ✗                               |                ✗                |

[^1]: Overlay works in offline mode, but will not show any content requiring the internet connection.
[^2]: Except IsDlcOwned method which requires to be LoggedOn.

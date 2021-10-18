# General Remarks

## Project Structure

In our demo project, we use one main class called **GalaxyManager**, as well as 7 other classes, which we refer to as **feature classes**. Each feature class represents a separate functionality of the GOG GALAXY SDK.

The feature classes are:

- **Friends**
- **Invitations**
- **Leaderboards**
- **Matchmaking**
- **Networking**
- **StatsAndAchievements**
- **Storage**

**GalaxyManager** contains definitions for the most basic GOG GALAXY SDK methods, which can be used even if the SDK was not fully initialized, such as [`GetCurrentGameLanguage()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a3f9c65577ba3ce08f9addb81245fa305) and [`IsDLCInstalled()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a46fbdec6ec2e1b6d1a1625ba157d3aa2). It assigns its own instance to the `GalaxyManager.Instance` static variable, which is accessible throughout the entire project. For ease of access, it initializes, terminates, and holds references to all feature classes.

## Feature Class Structure

To improve the readability of the feature classes, we have structured them in accordance with a specific scheme outlined below — but this is completely optional.

In our scheme, every class is divided into 5 regions:

| **Region**           | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| **Variables**        | Contains all variables defined in the class                  |
| **Behaviors**        | Contains all MonoBehaviour methods used for script behaviour (e.g. `OnEnable()`, `OnDisable()`, or `Update()`) |
| **Listener methods** | Contains all methods related to listeners (in most cases, these are limited to methods, which initialize and terminate listeners) |
| **Methods**          | Contains all methods related to the particular feature class |
| **Listeners**        | Contains definitions of all listener classes related to the particular feature class |

## Debug Menu

![Debug Menu](_assets/demo-debug-menu.png)

At any point during the game, you can open *Debug Menu* by pressing ++f9++. *Debug Menu* allows you to call many of the Galaxy SDK methods from anywhere in the game. The result of the call will be shown in the **Console**, which is also available from *Debug Menu*. The menu was created for testing and debugging purposes, and contains several sections:

| **Section**      | Function                                                     |
| ---------------- | ------------------------------------------------------------ |
| **User**  | Allows you to monitor the current user’s status, which includes: ✔︎ checking if the SDK [`Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) method was successfully called, ✔︎ checking if the user is **Signed In** (the GOG GALAXY client is running and the game license is present on the user’s account), ✔︎ checking if the user is **Logged On** (the user is **Signed In** and the internet connection is available), ✔︎ displaying the user’s nickname. It also allows you to sign in or sign out a user, or check current user status |
| **Apps**         | Allows you to call the `GetCurrentGameLanguage()` which returns current game language (based on language set in the GOG GALAXY client), as well as `IsDlcInstalled` used to check whether a particular DLC is installed by passing its productID |
| **Utils**        | Allows you to show a website with a provided URL in the GOG GALAXY Overlay and set users rich presence |
| **Achievements** | Allows you to set (unlock) game achievements by passing their **apiKey**, get the unlock statuses of achievements, as well as to reset all user statistics and achievements in the game |
| **Statistics**   | Allows you to set values for all user statistics in game by calling the [`SetStatFloat()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#adefd43488e071c40dc508d38284a1074) and [`SetStatInt()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#ab6e6c0a170b7ffcab82f1718df355814) methods, to get these values, and to reset all user statistics and achievements in the game |
| **Leaderboards** | Allows you to request different scopes of entries (global and near the user) for a particular leaderboard, as well as to set the user’s score on a particular leaderboard |
| **Storage** | Allows you to upload and download files from the GOG GALAXY Cloud Storage. The `FileWrite` method moves a file specified by its absolute path to the GOG GALAXY Local Storage directory. Files located in the GOG GALAXY Local Storage will be synchronized with the GOG GALAXY Cloud Storage automatically. `FileRemove` removes a file specified by its path relative to the GOG GALAXY Local Storage, `FileShare` will share the file to the GOG GALAXY Cloud Storage, while `FileShareAll` will share all files located in the GOG GALAXY Local Storage. Shared files can be downloaded by any GOG user in the game by using `sharedFileID` of a file, and this is exactly what the `FileDownloadBySharedFileID` method does. `FileDownloadByNameAndUser` adds a level of abstraction to this process that allows you to download a file by specifying its name and the user ID of its owner |
| **Game**         | Allows you to remove specific **gameObjects** (pool balls) from the scene to make the game easier, or to finish the game (yep, it’s cheating, but no one needs to know) |
| **Arguments**    | Displays all arguments that were used to start the game      |
| **Console**      | Displays Unity debug logs on the screen                      |

## Glossary

- **GOG GALAXY Client** — our optional gaming client developed for easy access to games and goodies. Required for most of the GOG GALAXY SDK features to work.
- **GOG GALAXY SDK** — a set of libraries that allow you to implement the GOG GALAXY features in your games.
- **GOG GALAXY Peer** — a working instance of the GOG GALAXY SDK.

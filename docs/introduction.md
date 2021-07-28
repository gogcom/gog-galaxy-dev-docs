> ***“With GOG GALAXY, being online will always be optional”***
>
> ***“Our client application will offer convenient game updating […], but we will never force it on you”.***
>
> — GOG GALAXY presentation on June 5, 2014

# Welcome to GOG GALAXY!

We are very happy to welcome you in our ever-growing family of game developers and publishers. We are sure you have many questions about the GOG GALAXY SDK and how it can be implemented to enhance your game. You’ll find the answers in the articles throughout our support center, which will provide you with information about releasing your games on GOG, implementing the GOG GALAXY SDK, and more. But first, we would like to take this moment to share our vision of GOG GALAXY.

## The Optional Client, Awesome SDK Features, DRM-free

The most obvious and visible part of GOG GALAXY is our desktop client application which lets you install, update, and play your games. Along with these features, you can also check your achievements, invite your friends to multiplayer matches, and access any other GOG GALAXY functionalities that you, as a developer, wish to implement into your game through our [Software Development Kit](sdk.md). It is our goal to provide you with the tools and features to enhance your game and offer a thrilling and engaging experience to our users.

That said, it is also important to us that our users have control over their games and play, which is why we conceived GOG GALAXY as a completely optional and DRM-free experience. That means that it’s the user who decides on the level of their involvement in accessing and using GOG GALAXY functionalities. We encourage users to download and play their games using the GOG GALAXY client, but we also host “offline installers” and bonus content/DLCs on the GOG.com website. Therefore, users can download, update, play, and enjoy their games and content through GOG without any interaction with the GOG GALAXY client.

Please keep this optional aspect in mind when integrating the GOG GALAXY SDK into your game, ensuring that the single player/offline functionalities of the game are available to the player even if the GOG GALAXY client is not present on their systems, or the player is logged out. Only the online functionalities (such as multiplayer or achievements) should require the GOG GALAXY client to be installed.

## Always In The Game

With the above philosophy in mind, we would like to show you our intended way of implementing GOG GALAXY into your game.

First of all, the **GOG GALAXY client should not be required to launch the game**. If our desktop application is not present, the game modules that require it should be disabled. The game itself will still run, but without features such as achievements, leaderboards, statistics or multiplayer.

Secondly, even if the GOG GALAXY client is installed and running, **player authentication in our servers should be entirely optional**, because we do not require an active connection between the client and our servers.

Last, but not least, **if a user loses their connection to the GOG GALAXY servers, the game should seamlessly switch to an offline mode**.

All of this will be covered in greater detail in our dedicated documentation and we’re here to help should you have any questions or run into any problems.
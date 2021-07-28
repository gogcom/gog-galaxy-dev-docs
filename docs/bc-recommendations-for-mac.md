# Our Recommendations for macOS Game Structure

We strongly recommend releasing the game as a single Application Bundle: all the game content, including game assets, DLCs, audio, etc. contained inside a single *GameName.app* folder. However, this requires you to prepare a [valid macOS Application Bundle](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW1) beforehand.

Our recommendations:

- Prepare application bundle according to the [Apple documentation](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW1).
  All the product content (main game binaries, assets, as well as DLCs) should be installed inside the app bundle on the end-user machine.
- Follow Apple’s recommendation and don’t write game saves and any temporary files inside the application bundle while it is running. For that you should use system preferred paths, for example *~/Library/Application Support/<gamename\>*. Otherwise, they could be lost during game files sync by GOG GALAXY. Read about [GOG GALAXY Cloud Saves](gc-cloud-saves.md).
- Select *App bundle* option in the *Installation Directory* section of *Project Properties* window.
- Select your *.app* directory as an only one **depot** for a simple game, or — in case of having multiple depots across the game and its DLCs — make sure they have the same folder structure. [Read more about adding depots](bc-adding-depots.md).
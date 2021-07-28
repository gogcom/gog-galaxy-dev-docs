# Cloud Saves

Using GOG GALAXY features, you have two options to implement synchronization of user’s save files with the Cloud:

- Automatic saves syncing through the GOG GALAXY client
- The [IStorage interface](sdk-storage.md) from the GOG GALAXY SDK.

## Automatic Cloud Saves Syncing

We provide automatic saves syncing in the GOG GALAXY client. This is the easiest way of implementing Cloud Saves without changing the source code of a game and we are integrating such Cloud Saves internally, so there is no need for you to implement any extra features in your code! Where possible, cloud saves support will be added during the final stages of our QA process.

When automatic cloud saves are enabled, the GOG GALAXY client will synchronize files in a specified directory on the user’s local machine with the cloud every time the game is launched or closed.

Unfortunately, sometimes this feature can’t be implemented by us and when this happens, you have to use the alternative method to enable cloud saves in your game.

## IStorage Interface from the GOG GALAXY SDK

If your game requires more complex managing of local and remote files or you don’t want to interact with the local file system directly, you can use the IStorage interface from the GOG GALAXY SDK. It provides the abstraction layer over the file system (methods for reading/writing/deleting files). You can also share the files created this way with other GOG users. Such files will also be synchronized automatically with the cloud, like mentioned in the previous section.

In this case, you need to implement the IStorage interface in your game code. More detailed information about this interface and its functions may be found in the [GOG GALAXY SDK API documentation](https://docs.gog.com/galaxyapi/index.html) (you can get the newest SDK libraries and documentation from the [Developer Portal](https://devportal.gog.com/galaxy/components/sdk)).
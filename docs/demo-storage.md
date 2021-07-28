# Storage

!!! Important
    Please note that implementing storage is not necessary if you only want to use cloud saves. Cloud saves can be handled by the GOG GALAXY client without the need of adding any special code to your game. The Storage interface of the GOG GALAXY SDK was created to allow users to share files with others.

## Description

This class allows you to upload and download files from the cloud storage in GOG GALAXY. Although it’s implemented with no particular purpose in the game, it can be used to experiment with the interface using the methods we created available in the [*Debug Menu*](demo-general-remarks.md#debug-menu).

## Class Initialization and Termination

The **Storage** class is initialized, when the user successfully logs in (in the [**AuthenticationListener.OnAuthSuccess()**](demo-galaxy-manager.md#galaxy-peer-initialization) callback of the **GalaxyManager** class). When enabled, the **Storage** class initializes three listeners:

- **FileShareListener**, which waits for the result of sharing a file, and adds the `fileName` key with the `sharedFileID` value to the current user data.
- **SharedFileDownloadListener** – a listener for the `DownloadSharedFileBySharedFileID` method for handling downloading of files from the local storage.
- **SpecificUserDataListener** is used to catch a callback for the [`RequestUserData`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a8e77956d509453d67c5febf8ff1d483d) method called inside of the `DownloadSharedFileByUserIdAndFileName` method in order to retrieve a `sharedFileID` of a file shared by other user.

All listeners are disposed of when the class is disabled.

## Definitions of Listeners

### FileShareListener

When a file was successfully shared, the callback comes to `FileShareListener.OnFileShareSuccess()`, where we can get the `sharedFileID` parameter provided with the callback (`sharedFileID` is necessary for sharing a file with other users), and then we store this value in the user data.

### SharedFileDownloadListener

This listener callback will run when a file was downloaded successfully or if it failed in the process. The downloaded file is held in memory and we use this listener to read the data and write it in the local storage. The file will be written to a folder inside of the local storage named either *0*, if the file was downloaded from an unspecified user using the `DownloadSharedFileBySharedFileID` method, or the owner’s Galaxy ID, if the file was downloaded from a specific user using the `DownloadSharedFileByUserIdAndFileName` method.

### SpecificUserDataListener

`SpecificUserDataListener` is a specific listener, which means it will only work for the methods which explicitly register to this listener. This listener is used in pair with the [`DownloadSharedFileByUserIdAndFileName`](demo-storage-implementation.md#downloadsharedfilebyuseridandfilename) method in order to let the user download a file from another user by using theirs GalaxyID and the name of the file.

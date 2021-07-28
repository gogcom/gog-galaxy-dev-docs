# Storage

The GOG GALAXY SDK **Storage** interface can be used in variety of cases. The most common one would be to create save files that are automatically synchronized with the cloud, but you can also use it to share files with members of an online lobby or with friends.

!!! Tip "Cloud Saves"
    Please note that the GOG GALAXY client has support for cloud saves that doesn’t require the use of the [IStorage](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#details) interface, or any other interference in your game code. **We highly recommend using the automatic cloud saves feature unless your game contains features that require the use of the IStorage interface.** For more details, please contact your Product Manager.

## Listeners

The Storage interface uses two listeners:

- [`SharedFileDownloadListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ISharedFileDownloadListener.html), which retrieves callbacks from the [`DownloadSharedFile()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#aabdcf590bfdd6365e82aba4f1f332005) method, and
- [`FileShareListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFileShareListener.html) used to retrieve callbacks for the [`FileShare`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a10cfbb334ff48fcb8c9f891adc45ca1d) method.

Please note that the `DownloadSharedFile` and `FileShare` methods will not function properly without their respective listeners.

## Cloud Saves

Implementing cloud saves using the Storage interface is pretty simple. In fact, all you need is the [`FileWrite()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a1c3179a4741b7e84fe2626a696e9b4df) method, and as it does not require any listeners, you don’t have to register any of them before proceeding.

!!! Important
    The `fileName` parameter of the `FileWrite()` method must be provided in the form of a path relative to local storage, and can use slashes to create subfolders as long as it is a valid UTF-8 string.

The `FileWrite()` method writes the data provided as a byte array to a specified path, including file name and extension, in local storage. All files located inside local storage are synchronized with the cloud every time the game is opened and closed. Local storage is located on the end user’s machine in:

- *%LocalAppData%\GOG.com\Galaxy\Applications\%ClientID%\Storage* on **Windows**, and
- *~/Library/Application Support/GOG.com/Galaxy/Applications/%ClientID%/Storage* on **macOS**.

!!! Warning
    It is advised not to manipulate files inside local storage outside of the GOG GALAXY SDK.

Therefore, to implement cloud saves in a game using the Storage interface, all you would need to do is to write data into save files using the `FileWrite()` method. Then the GOG GALAXY client would take care of synchronizing the saves for you.

## Sharing a File

This case requires few more steps. Usually, you would start with the same setup as in the [Cloud Saves](#cloud-saves) scenario above, since the file you want to share has to be in your local storage. When writing a file to local storage using the `FileWrite()` method, you will be able to give it a name using the `fileName` parameter. This `fileName` is important as it will be used as a reference to this particular file by other Storage methods.

Next, you will need to register the [`FileShareListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFileShareListener.html). This is a very important step, as the only way to get the shared file ID is from the [`FileShareListener::OnFileShareSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFileShareListener.html#a585daa98f29e962d1d5bf0a4c4bd703e) callback.

Once you’ve written the file to local storage and registered the listener, you can use the [`FileShare`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a10cfbb334ff48fcb8c9f891adc45ca1d) method in order to share the file. It is important to use the same `fileName` as in the `FileWrite()` method before.

`FileShare()` is an asynchronous method and therefore you will need to wait for the callback before proceeding. Once this call is finished and successful, you will receive the `FileShareListener::OnFileShareSuccess()` callback, which comes with two parameters: `fileName` and [`sharedFileID`](https://docs.gog.com/galaxyapi/group__api.html#ga34a0c4ac76186b86e6c596d38a0e2042). `sharedFileID` can be used by other users to download the file — in fact, if one user wants to download a file from another user, they will need to have the `sharedFileID` of the file.

## Downloading a Shared File

Once a file is shared and you have the `sharedFileID`, you can start downloading it.

First, you will need to register the [`SharedFileDownloadListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ISharedFileDownloadListener.html). This is quite useful, as you can use the [`SharedFileDownloadListener::OnSharedFileDownloadSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ISharedFileDownloadListener.html#a6ab1bf95a27bb9437f2eb3daff36f98d) to trigger the process of writing the downloaded file in the local storage.

Now you are ready to download the file. To start downloading the file, you should use the [`DownloadSharedFile()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#aabdcf590bfdd6365e82aba4f1f332005) method and provide the `sharedFileID` of the file you want to be downloaded. This call is asynchronous, and the completion of the download is signaled by the `SharedFileDownloadListener::OnSharedFileDownloadSuccess()` callback. As mentioned before, you can use this callback to write the byte array of the downloaded data to the file in local storage.

To write a downloaded file to your local storage, you should use three methods:

1. [`SharedFileRead()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#af4bbf0a1c95571c56b0eca0e5c3e9089) to read the downloaded file content into a buffer. Please note that you will need to provide a buffer for the content.
2. [`FileWrite()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a1c3179a4741b7e84fe2626a696e9b4df) to write the file to your local storage.
3. [`SharedFileClose()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a3f52b2af09c33a746891606b574d4526) to close the downloaded file and free up memory.

!!! Tip
    You can also use the [`GetSharedFileOwner()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a96afe47cd5ed635950f15f84b5cd51d3) method to determine from whom this file was downloaded. Moreover, you can use the [`GalaxyID`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1GalaxyID.html) returned by this method, and the `fileName` parameter of the `FileWrite()` method, to separate files downloaded from different users.

## Example: Sharing a File With Other Lobby Members

Let’s say your game has a multiplayer mode and it allows players to create their own custom emblems that will be visible to other players in said multiplayer mode.

1. The player creates their own emblem. Once the player has finished the emblem, you’ll need to write it to local storage using the `FileWrite()` method.
2. Now register the `FileShareListener` and share the file using the `FileShare()` method.
3. When the file is uploaded, the player will receive the `FileShareListener::OnFileShareSuccess()` callback containing the `SharedFileID` of the file with the player’s emblem. This `SharedFileID` is needed to share the file with other players later, so it has to be saved locally among other player data.
4. After the actions in step #3 are performed, you can safely unregister the `FileShareListener`.
5. As soon as a player joins the lobby, you should register the `SharedFileDownloadListener` and the [`LobbyDataListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyDataListener.html).

    !!! Info "About `LobbyDataListener`"
        `LobbyDataListener` informs about the event of a lobby or lobby member data update. [`LobbyDataListener.OnLobbyDataUpdated()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1ILobbyDataListener.html#a1dc57c5cd1fca408a3752cd0c2bbbebc) has two parameters: `lobbyID` and `lobbyMemberID`. If `lobbyMemberID` is empty, ie. `lobbyMemberID` is equal to `GalaxyID(0)`, it means that the **lobby data** was updated; otherwise it means that the **lobby member data** was updated.

6. Next, you may use the [`Matchmaking::SetLobbyMemberData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#a6e261274b1b12773851ca54be7a3ac04) method to write the `SharedFileID` of the player’s emblem in a place accessible for other players. Let’s use a string “emblem” as a key, and the `SharedFileID`of the emblem as a value parameter of the `Matchmaking::SetLobbyMemberData()` method.
7. Lastly, you call [`Matchmaking::GetLobbyMemberData()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IMatchmaking.html#acfc05eaa67dd8dbd90d3ad78af014aa2) to get the `SharedFileID` of other players’ emblems, and then use the said `SharedFileID` to download the file using the `DownloadSharedFile()` method. You may write down these other players’ emblems in local storage using the `SharedFileRead()`, `FileWrite()`, and `SharedFileClose()` methods. You may also use the `GetSharedFileOwner()` method to separate the files downloaded from different users.

!!! Important
    Please note that before using any of the following methods you need to call `DownloadSharedFile()` and wait for the callback on `SharedFileDownloadListener`:
    
    - [`GetSharedFileName()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#aa5c616ea1b64b7be860dc61d3d467dd3)
    - [`GetSharedFileNameCopy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#aa42aa0a57227db892066c3e948c16e38)
    - [`GetSharedFileOwner()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a96afe47cd5ed635950f15f84b5cd51d3)
    - [`GetSharedFileSize()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#af389717a0a383175e17b8821b6e38f44)
    - [`SharedFileClose()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#a3f52b2af09c33a746891606b574d4526)
    - [`SharedFileRead()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStorage.html#af4bbf0a1c95571c56b0eca0e5c3e9089)
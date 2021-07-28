# Friends

## Description

This class is responsible for handling the data of the currently logged in user and their friends. It is also used to set users’ rich presence.

!!! Note
    It is NOT obligatory to call the [`RequestFriendList`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriends.html#aa648d323f3f798bbadd745bea13fc3b5) method before calling other methods from the GOG GALAXY SDK Friends interface. `RequestFriendList` remains available for backwards compatibility. At the moment, friend list request is automatically performed by the GOG GALAXY SDK when the user is signed in.

## Class Initialization and Termination

This class follows the same routine as the other **GalaxyManager** feature classes when created and/or destroyed. When this class is enabled, we automatically initialize its listeners, and when it is disabled, we make sure that those listeners are closed.

Additionally, we subscribe to the **SceneManager.sceneLoaded** event with the **OnSceneLoaded** event handler with the intent to update users’ rich presence throughout the game. Please note that there is one instance when we update users’ rich presence manually: when the user creates or joins an online lobby.

```c#
    #region Behaviors

    void OnEnable()
    {
        ListenersInit();
        SceneManager.sceneLoaded += OnSceneLoaded;
    }

    void OnDisable()
    {
        ListenersDispose();
        SceneManager.sceneLoaded -= OnSceneLoaded;
    }

    #endregion

    #region Passive methods

    private void ListenersInit()
    {
        Listener.Create(ref friendListListener);
        Listener.Create(ref richPresenceChangeListener);
    }

    private void ListenersDispose()
    {
        Listener.Dispose(ref friendListListener);
        Listener.Dispose(ref richPresenceChangeListener);
    }

    // Sets the user rich presence based on the currently loaded scene name
    public void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        string value;
        sceneToRichPresenceDict.TryGetValue(scene.name, out value);
        SetRichPresence("status", value);
    }

    #endregion
```

## Definitions of Listeners

The Friends class requires two listeners:

- **FriendListListener**, which receives a callback when a friend list is retrieved
- **RichPresenceChangeListener**, which receives callbacks whenever the value of users’ rich presence is set.

### FriendListListener

```c#
    private class FriendListListener : GlobalFriendListListener
    {
        public bool retrieved = false;

        public override void OnFriendListRetrieveSuccess()
        {
            Debug.Log("Friend list retrieved");
            retrieved = true;
        }

        public override void OnFriendListRetrieveFailure(FailureReason failureReason)
        {
            Debug.Log("Friend list couldn't be retrieved, for reason " + failureReason);
        }

    }
```

We start by defining the `retrieved` public Boolean variable inside the **FriendListListener** class and then we use it to store information on whether the user friend list was retrieved or not. We decided to store this information as the Boolean, so we can easily check if the friend list was retrieved and if it safely uses the methods that require it.

!!! Important
    A friend list is automatically requested when the user is successfully signed in and this listener is still fired when the list is retrieved in this way. Because of this, it is not necessary to call `RequestFriendList()`.

We use [`OnFriendListRetrieveSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriendListListener.html#a8b9472f2304e62a9b419c779e8e6a2f6) and [`OnFriendListRetrieveFailure()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IFriendListListener.html#a9cbe96cfeea72a677589b645b4431b17) callbacks to this listener to set the value of the `retrieved` variable.

### RichPresenceChangeListener

We use this listener for debug purposes only.

```c#
    private class RichPresenceChangeListener : GlobalRichPresenceChangeListener
    {
        public override void OnRichPresenceChangeSuccess()
        {
            Debug.Log("Rich presence updated successfully");
        }

        public override void OnRichPresenceChangeFailure(FailureReason failureReason)
        {
            Debug.Log("Rich presence update failure for reason " + failureReason);
        }
    }
```
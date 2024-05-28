# Galaxy Manager

## Description

**GalaxyManager** is a central class managing all of the GOG GALAXY features. It inherits from the MonoBehaviour superclass. It should be initialized at the very start of the game and present for the entire duration of the game session. When the application is closed, **GalaxyManager** should make sure that all listeners are closed and **GalaxyInstance** is shut down.

## Class Initialization

To ensure that all requirements listed in the previous section are met, we take advantage of the MonoBehaviour Event Functions execution order.

First, we create a GameObject called **GalaxyManager** in Unity Editor, in the first game scene. Then we add a **GalaxyManager script component** to that GameObject.

When the GameObject is created, the `Awake()` Event function is called. We use the `Awake()` method to:

- add `DontDestroyOnLoad()` flag to the GameObject this script is attached to,
- create Instance, which is a static reference to **GalaxyManager** class, for easier access from outside of the class,
- make sure that there is only one instance of **GalaxyManager** running at all times: the Instance is only assigned if its current value is null; if the Instance is already assigned, we destroy the extra instance of GalaxyManager.

```c#
   void Awake()
   {
       if (Instance == null)
       {
           Instance = this;
           DontDestroyOnLoad(gameObject);
       }
       else
       {
           Destroy(this);
       }
   }
```

We take this approach (creating a static reference to a class that inherits from MonoBehaviour), because a static class cannot inherit from other classes, but we want **GalaxyManager** to inherit from MonoBehaviour and still have an easy access to the class.

## Galaxy Peer Initialization

When the **GalaxyManager** GameObject is enabled, the[`Init()`](https://docs.gog.com/galaxyapi/group__Peer.html#ga7d13610789657b6aebe0ba0aa542196f) method is called to initialize the Galaxy Peer. Init argument, `initParams`, requires Product credentials: **clientID** and **clientSecret**.

!!! Important
    Even if `Init()` returns an error, the Galaxy Peer will still be initialized, but only the [`Apps`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html) interface will be accessible. The only situation, in which `Init()` will fail and none of the GOG GALAXY SDK interfaces will be available is when *Galaxy.dll* is not present in the working directory of the game.

If `Init()` succeeds, we initialize [`AuthListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html) using the `ListenersInit()` method and finally sign in the current user to GOG GALAXY backends using the [`SignInGalaxy()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a65e86ddce496e67c3d7c1cc5ed4f3939) method.

```c#
   void OnEnable()
   {
       Init();
       ListenersInit();
       SignInGalaxy();
   }
```

The response to `SignInGalaxy()` comes to [`AuthListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html). If the user was signed in successfully, we can continue initializing the remaining GOG GALAXY features in the [`AuthListener.OnAuthSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAuthListener.html#aa824226d05ff7e738df8e8bef62dbf69) callback.

There, with the `StartStatsAndAchievements()` and the `StartFriends()` methods, we initialize helper objects allowing us to access these GOG GALAXY SDK interfaces using appropriate methods.

We also assign the current user’s GalaxyID to the `myGalaxyID` variable for easier access. Then, in case the user is online (`IsLoggedOn` is `true`), the `StartLeaderboards()` and `StartInvitations()` methods are initialized as well (Leaderboards and Invitations can’t be started when a user is offline).

!!! Note
    The `StartStorage()` method is also initialized, but we don’t use Storage in our game. It’s here for demo purposes only, as a “proof of concept”.

Lastly, two fallbacks are defined: when user authentication failed and when the user suddenly signs off from the Galaxy service.

```c#
    public class AuthenticationListener : GlobalAuthListener
    {
        public override void OnAuthSuccess()
        {
            Debug.Log("Successfully signed in");

            myGalaxyID = GalaxyInstance.User().GetGalaxyID();

            GalaxyManager.Instance.StartStatsAndAchievements();
            GalaxyManager.Instance.StartFriends();

            if (GalaxyManager.Instance.IsLoggedOn())
            {
                GalaxyManager.Instance.StartLeaderboards();
                GalaxyManager.Instance.StartInvitations();
                GalaxyManager.Instance.StartStorage();
            }

        }

        public override void OnAuthFailure(FailureReason failureReason)
        {
            Debug.LogWarning("Failed to sign in for reason " + failureReason);
            if (GameObject.Find("MainMenu")!= null) GameObject.Find("MainMenu").GetComponent<MainMenuController>().GalaxyCheck();
        }

        public override void OnAuthLost()
        {
            Debug.LogWarning("Authorization lost");
            if (GameObject.Find("MainMenu") != null) GameObject.Find("MainMenu").GetComponent<MainMenuController>().GalaxyCheck();
        }

    }
```

!!! Note
    To add other listeners you need to implement the corresponding listener's interface with its methods.

## Processing Data

Using the `Update()` method, we make sure that **GalaxyManager** processes data on every frame by calling `ProcessData()`. Without it, the listeners won’t work, and neither will any asynchronous functions.

```c#
   void Update()
   {
       /* Makes the GalaxyPeer process data.
       NOTE: This is required for any listener to work, and has little overhead. */
       GalaxyInstance.ProcessData();
   }
```

## GalaxyManager and GalaxyPeer Termination

Once again we take advantage of the MonoBehaviour Event Functions. Here, we use two of them, called one after another: `OnDisable()` and `OnApplicationQuit()`.

`OnDisable()` makes sure that all GOG GALAXY features and listeners are properly closed by calling `ListenersDispose()` and `ShutdownAllFeatureClasses()`.

`OnApplicationQuit()` calls `GalaxyInstance.Shutdown()` and then destroys the instance of **GalaxyManager**. It should be the last method called, since the listeners should be disposed of beforehand.

`GalaxyInstance.Shutdown()` is necessary to close the GalaxyPeer instance and should always be called when closing the game.

```c#
    void OnDisable()
    {
        ShutdownAllFeatureClasses();
        ListenersDispose();
    }

    void OnApplicationQuit()
    {
        /* Shuts down the working instance of GalaxyPeer.
        NOTE: Shutdown should be the last method called, and all listeners should be closed before that. */
        GalaxyInstance.Shutdown(true);
        Instance = null;
        Destroy(this);
    }
```

!!! Important
    Keep in mind that the `GalaxyInstance.Shutdown()` method requires the **unloadModule** parameter. It is important only in the PS4 implementation (it indicates whether the internal `sceKernelStopUnloadModule()` for prx file will be called), so on other platforms the best practice is to set it to `true`.

## GOG GALAXY Features Initialization

All scripts containing GOG GALAXY feature classes (StatsAndAchievements, Leaderboards, Friends, Matchmaking, Networking, Invitations and Storage) are added as components to the **GalaxyManager** GameObject. Not all **GalaxyManager** feature classes are enabled along with the **GalaxyManager**, since not all of them are necessary.

Every GOG GALAXY feature class initializes its own listeners when it is enabled and disposes of them when it is disabled. You can see this in the example below, using the **StatsAndAchievements** class:

```c#
    private void ListenersInit()
    {
        Listener.Create(ref achievementRetrieveListener);
        Listener.Create(ref achievementChangeListener);
        Listener.Create(ref achievementStoreListener);
    }

    private void ListenersDispose()
    {
        Listener.Dispose(ref achievementStoreListener);
        Listener.Dispose(ref achievementRetrieveListener);
        Listener.Dispose(ref achievementChangeListener);       
    }
```

This way we can be sure that the appropriate listeners are only active when it is necessary.

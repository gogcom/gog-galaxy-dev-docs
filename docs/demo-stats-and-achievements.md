# Stats and Achievements

## Description

This class is responsible for handling game achievements and user statistics. It contains functions that wrap the SDK methods for unlocking achievements and setting values of user statistics.

## Class Initialization and Termination

When enabled, the **StatsAndAchievements** class initializes three listeners:

- **UserStatsAndAchievementsRetrieveListener**, which receives callbacks from the [`RequestUserStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a38f5c146772f06dfd58c21ca599d7c25) method,
- **StatsAndAchievementsStoreListener**, which gets callbacks from the  [`StoreStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a0e7f8f26b1825f6ccfb4dc26482b97ee) method, and
- **AchievementChangeListener**, which receives notifications about unlocking a particular achievement.

When the class is disabled, we make sure all of above listeners are disposed of.

## Definitions of Listeners

### UserStatsAndAchievementsRetrieveListener

```c#
    private class UserStatsAndAchievementsRetrieveListener : GlobalUserStatsAndAchievementsRetrieveListener
    {
        public bool retrieved;

        public override void OnUserStatsAndAchievementsRetrieveSuccess(GalaxyID userID)
        {
            retrieved = true;
            GalaxyManager.Instance.StatsAndAchievements.SetAchievement("launchTheGame");
            Debug.Log("User " + userID + " stats and achievements retrieved");
        }

        public override void OnUserStatsAndAchievementsRetrieveFailure(GalaxyID userID, FailureReason failureReason)
        {
            retrieved = false;
            Debug.LogWarning("User " + userID + " stats and achievements could not be retrieved, for reason " + failureReason);
        }
    }
```

Inside the **UserStatsAndAchievementsRetrieveListener** class, we define a public Boolean named `retrieved`, and then use it to store information about the result of calling the [`RequestUserStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a38f5c146772f06dfd58c21ca599d7c25) method. More specifically, we keep track of whether the method was successful (returned `true`), which means the data on user’s statistics and achievements was retrieved. The `retrieved` variable is checked first, before methods that further deal with achievements and statistics are called (you must retrieve stats and achievements data to be able to set it).

### StatsAndAchievementsStoreListener

```c#
    private class StatsAndAchievementsStoreListener : GlobalStatsAndAchievementsStoreListener
    {

        public override void OnUserStatsAndAchievementsStoreFailure(FailureReason failureReason)
        {
            Debug.LogWarning("OnUserStatsAndAchievementsStoreFailure: " + failureReason);
        }

        public override void OnUserStatsAndAchievementsStoreSuccess()
        {
            Debug.Log("OnUserStatsAndAchievementsStoreSuccess");
        }
    }
```

The methods of this listener are called after it receives a callback on storing user’s stats and achievements data as a result of the [`StoreStatsAndAchievements()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a0e7f8f26b1825f6ccfb4dc26482b97ee) method. Here, we have implemented a debug message that shows information regarding whether data storage was finished successfully.

### AchievementChangeListener

```c#
    private class AchievementChangeListener : GlobalAchievementChangeListener
    {
        public override void OnAchievementUnlocked(string name)
        {
            Debug.Log("Achievement \"" + name + "\" unlocked");
        }
    }
```

The [`OnAchievementUnlocked()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IAchievementChangeListener.html#a0c2290aab40bc3d2306ce41d813e89f3) method of this listener is called when a notification about an unlocked achievement is received after the [`SetAchievement()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#aa5f8d8f187ae0870b3a6cb7dd5ab60e5) call. We do not implement any usage of this notification here, but it can be used to detect the unlocking event for a particular achievement — for example, to show an in-game pop-up or to store information about statuses of achievements in-game.

!!! Info
    When enabled, the [GOG GALAXY Overlay](gc-overlay.md) shows notifications on unlocking achievements.

## General Remarks

- Keep in mind that it’s important to call `RequestStatsAndAchievements()` before calling any other method related to achievements and statistics. However, you need to request them only once per game run.
- In order to send information about an achievement or statistic being set to the GOG GALAXY backend, you need to call the `StoreStatsAndAchievements()` method. In this demo, we call it within every **Set** method, but if you intend to have a large number of statistics with their values changing often, you should consider calling `StoreStatsAndAchievements()` from time to time, and also when the player quits the game.
- Operations that deal with setting and storing user stats and achievements are performed every time a user is logged in to the GOG GALAXY client — even if they are offline at the time. In such case, relevant data is stored locally and will be sent to the backend the next time the user runs the game while logged in and online. As a result, any achievement unlocked while in offline mode will show up in the GOG GALAXY client.

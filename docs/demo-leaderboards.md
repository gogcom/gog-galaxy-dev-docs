# Leaderboards

## Description

This class is responsible for handling user leaderboards. After the leaderboard entries (user rank, GalaxyID and score) are fetched, we store them in a private local list of object arrays named `leaderboardEntries`. To make it easily accessible from outside of the **Leaderboards** class, we create a public **LeaderboardEntries** get method as well.

!!! Important
    Similarly as with statistics and achievements, you need to request and receive leaderboards definitions before calling any other methods related to them. You can request leaderboard definitions by calling [`RequestLeaderboards()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a7290943ee81882d006239f103d523a1d).

## Class Initialization and Termination

In our demo game, the **Leaderboards** class is instantiated by adding the **Leaderboards** script to **GalaxyManager** GameObject with the `GalaxyManager.StartLeaderboards()` method called within `GalaxyManager.AuthenticationListener.OnAuthSuccess()` callback. To make sure that the user is online (Internet connection is required to use leaderboards), we check the user’s [`IsLoggedOn`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a3e373012e77fd2baf915062d9e0c05b3) status before enabling Leaderboards.

## Definitions of Listeners

### LeaderboardsRetrieveListener

```c#
    private class LeaderboardsRetrieveListener : GlobalLeaderboardsRetrieveListener
    {
        public bool retrieved = false;

        public override void OnLeaderboardsRetrieveSuccess()
        {
            Debug.Log("Leaderboard definitions retrieved");
            retrieved = true;
        }
        public override void OnLeaderboardsRetrieveFailure(FailureReason failureReason)
        {
            Debug.LogWarning("Leaderboard definitions couldn't be retrieved, for reason: " + failureReason);
        }

    }
```

It listens for an event of receiving definitions of all available leaderboards. You should request these definitions by calling [`GalaxyInstance.Stats().RequestLeaderboards()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a7290943ee81882d006239f103d523a1d).

Once the leaderboards definitions are retrieved, the `OnLeaderboardsRetrieveSuccess()` event is fired and we set the `LeaderboardsRetrieveListener.retrieved` Boolean to `true`. If the definitions could not be retrieved for any reason, `OnLeaderboardsRetrieveFailure()` is fired and we set `LeaderboardsRetrieveListener.retrieved` to `false`.

Either way, we use `LeaderboardsRetrieveListener.retrieved` later to ensure that the leaderboards definitions are retrieved.

### LeaderboardEntriesRetrieveListener

```c#
    private class LeaderboardEntriesRetrieveListener : GlobalLeaderboardEntriesRetrieveListener
    {
        public override void OnLeaderboardEntriesRetrieveSuccess(string leaderboardName, uint entryCount)
        {
            Debug.Log("Leaderboard \"" + leaderboardName + "\" entries retrieved\nEntry count: " + entryCount);

            leaderboardEntries.Clear();
            leaderboardEntries.TrimExcess();

            for (uint i = 0; i < entryCount; i++)
            {
                GalaxyID userID = new GalaxyID();
                uint rank = 0;
                int score = 0;
                string username = null;
                object[] entryDetails = new object[] { rank, score, username };
                
                GalaxyInstance.Stats().GetRequestedLeaderboardEntry(i, ref rank, ref score, ref userID);
                username = GalaxyManager.Instance.Friends.GetFriendPersonaName(userID);
                entryDetails[0] = rank;
                entryDetails[1] = score; 
                entryDetails[2] = username;
                Debug.Log("Created object #" + i + " | " + rank + " | " + score + " | " + username);
                leaderboardEntries.Add(entryDetails);
            }

            if (GameObject.Find("LeaderboardsScreen")) GameObject.Find("LeaderboardsScreen").GetComponent<LeaderboardsController>().DisplayLeaderboard();

        }

        public override void OnLeaderboardEntriesRetrieveFailure(string leaderboardName, FailureReason failureReason)
        {
            Debug.LogWarning("Could not retrieve leaderboard \"" + leaderboardName + "\" entries for reason: " + failureReason);
            if (GameObject.Find("LeaderboardsScreen"))
            {
                if (failureReason == FailureReason.FAILURE_REASON_NOT_FOUND) GameObject.Find("LeaderboardsScreen").GetComponent<LeaderboardsController>().DisplayMessage("No entry for the current user in selected leadeboard");
                if (failureReason == FailureReason.FAILURE_REASON_UNDEFINED) GameObject.Find("LeaderboardsScreen").GetComponent<LeaderboardsController>().DisplayMessage("Failure reason undefined");
            }
        }
    }
```

This listener callbacks are fired when entries for a specific leaderboard have been retrieved. To request leaderboard entries, you can call:

- [`GalaxyInstance.Stats().RequestLeaderboardEntriesGlobal(string leaderboardName, uint rangeStart, uint rangeEnd)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a8001acf6133977206aae970b04e82433) to retrieve entries from a specified range,
- [`GalaxyInstance.Stats().RequestLeaderboardEntriesAroundUser(string leaderboardName, uint countBefore, uint countAfter, GalaxyID userID)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a1215ca0927c038ed5380d23d1825d92d) to retrieve entries from the specified user vicinity,
- [`GalaxyInstance.Stats().RequestLeaderboardForUsers(string leaderboardName, ref GalaxyID[] userArray)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#ac06a171edf21bb4b2b1b73db0d3ba994) to retrieve entries for users specified in the **userArray** array.

In our project, we only use the first two methods. No matter which method is used inside the `LeaderboardEntriesRetrieveListener.OnLeaderboardEntriesRetrieveSuccess(string leaderboardName, uint entryCount)`, we define the following local variables:

| Variable          | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `uint rank`       | Used as a reference for the currently processed user’s rank later in the method |
| `int score`       | Used as a reference for the currently processed user’s score later in the method |
| `GalaxyID userID` | Used as a reference for the currently processed user’s GalaxyID later in the method |
| `string username` | Used as a reference for the currently processed user’s name later in the method |

First, we make sure that **Leaderboards.leaderboardEntries** is empty by calling `leaderboardEntries.Clear()` and `leaderboardEntries.TrimExcess()`.

Next, we use a `for` loop to iterate through all entries in the leaderboard table (each entry represents a score of one user). Inside this loop we:

1. Initialize the `userID` variable and define the `rank`, `score` and `username` variables.
2. Create a new object array for the entry.
3. Retrieve data for each entry separately with the `for` loop index using the [`GalaxyInstance.Stats().GetRequestedLeaderboardEntry(uint i, uint ref rank, int ref score, GalaxyID ref userID)`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IStats.html#a2a83a60778dafd8375aa7c477d9f7bff) method.
4. Add the created object array to the `Leaderboards.leaderboardEntries` list.

Once the `Leaderboards.leaderboardEntries` list is ready, we display the list using the `LeaderboardsController.DisplayLeaderboard()` method. Note that we first check if the **LeaderboardsScreen** GameObject is enabled.

If leaderboard entries retrieval failed for any reason, we display a message informing the user that the leaderboards couldn’t be retrieved.

### LeaderboardScoreUpdateListener

```c#
    private class LeaderboardScoreUpdateListener : GlobalLeaderboardScoreUpdateListener
    {
        public override void OnLeaderboardScoreUpdateSuccess(string leaderboardName, int score, uint oldRank, uint newRank)
        {
            Debug.Log("Set leaderboard \"" + leaderboardName + "\" score to " + score);
        }
        public override void OnLeaderboardScoreUpdateFailure(string leaderboardName, int score, FailureReason failureReason)
        {
            Debug.LogWarning("Could not set leaderboard \"" + leaderboardName + "\" entry for reason: " + failureReason);
        }
    }
```

This listener fires when a user leaderboard score was updated. We use it for debug purposes only as we don’t want to show this information to the player.

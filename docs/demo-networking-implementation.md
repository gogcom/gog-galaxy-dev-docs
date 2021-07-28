# Networking: Examples of Implementation

## Sending and Receiving P2P Packets

### User Experience

During an online multiplayer game, it is necessary for the players to share the information on the game and state of the game objects, so that every player can see the actions of others in real time.

### Solution

The transfer of information about the game and/or game objects is accomplished by sending and reading P2P packets containing this information between players in the same lobby. It is important to send packets with an adequate frequency, so the players don’t experience any lag (at least until the connection itself is not overloaded with other data). In this demo, we implemented an online multiplayer game mode for 2 players using peer-to-peer connection. We send one P2P packet containing the positions of all pool balls (`ballPacketBuffer`) every 0.015 s (which is around 60 packets per second) using the `SendPacketCoroutine()`. A packet containing the information about the last shot made (`switchPacketBuffer`) is only sent when needed, right after a finished shot.  

### Variables

| Variable             | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `balls`              | Array of GameObjects representing pool balls                 |
| `receiverID`         | GalaxyID of the user who will receive packets from this GalaxyPeer |
| `switchPacketBuffer` | Array containing bytes with the information on the last shot made, namely last foul type (if there was any), the expected color of the ball for the next shot, and the shot score |
| `ballPacketBuffer`   | Array containing the **x** and **z** coordinates of all balls (**y** is constant) translated into bytes, together with a byte containing the information whether a **ball** GameObject is active in the game scene |

### Methods and Usage

#### Networking.OnEnable

```c#
    void OnEnable()
    {
        ListenersInit();
        balls = GameObject.Find("PoolBalls").GetComponent<PoolBalls>().poolBalls;

        if (receiverID == null)
        {
            receiverID = GalaxyManager.Instance.Matchmaking.GetSecondPlayerID();
            Debug.Log("receiverID was null. ReceiverID set to: " + receiverID);
        }
        sendPacketCoroutine = SendPacketCoroutine(waitTime);
        StartCoroutine(sendPacketCoroutine);
    }
```

When the script is enabled, **NetworkingListener()** is initialized, **GameManager.poolBalls** array is assigned to the `balls` variable and GalaxyID of the second player in the lobby is retrieved and assigned to the `receiverID` variable.

#### Networking.SendP2PPacket

```c#
    public void SendP2PPacket(GalaxyID galaxyID, byte[] data, P2PSendType sendType = P2PSendType.P2P_SEND_RELIABLE, byte channel = 0)
    {
        uint dataSize = (uint)data.Length;
        try
        {
            GalaxyInstance.Networking().SendP2PPacket(galaxyID, data, dataSize, sendType, channel);
        }
        catch (GalaxyInstance.Error e)
        {
            Debug.LogError("Couldn't send packet for reason: " + e);
        }
    }
```

This method is a wrapper for the GOG GALAXY SDK [`SendP2PPacket()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html#a1fa6f25d0cbd7337ffacea9ffc9032ad) method, which allows us to send a P2P packet to a user or a lobby with a specified GalaxyID. Please note that communication is possible only among users connected to the same lobby.

The type of a P2P packet is specified by [`P2PSendType`](https://docs.gog.com/galaxyapi/group__api.html#gac2b75cd8111214499aef51563ae89f9a); here we use *P2PSendType.P2P_SEND_RELIABLE* type.

Since there is only a single connection between two GOG GALAXY Peers, using channels is a way of separating communication layers between them while using the existing connection. Here, we use two different channels (0 and 1) to separate packets with the positions of the balls from those with the information on the most recent shot. When using channels, it is important to also specify them in the GOG GALAXY Peer receiver when reading or peeking a packet.

#### Networking.SendPacketWithPlayerShot

```c#
    public void SendPacketWithPlayerShot(GameManager.FoulEnum foul, GameManager.BallColorEnum ballOn, int shotScore)
    {
        Debug.Log("Packet switch sent");

        SendPacketWithBallPositions();
        GetBytesFromInt((int)foul, 0, ref switchPacketBuffer);
        GetBytesFromInt((int)ballOn, 4, ref switchPacketBuffer);
        GetBytesFromInt(shotScore, 8, ref switchPacketBuffer);
        SendP2PPacket(receiverID, switchPacketBuffer, P2PSendType.P2P_SEND_RELIABLE, (byte)1);
    }
```

This method sends the `switchPacketBuffer` packet with the information on the last shot made (foul type, the expected color of the ball for the next shot, and the shot score). It uses the `GetBytesFromInt()` helper function to translate the information mentioned above from a byte array into an integer.

Those packets are sent over the channel 1.

#### Networking.SendPacketWithBallPositions

```c#
    void SendPacketWithBallPositions()
    {
        Debug.Log("Packet position sent");

        int startIndex = 0;
        for (int i = 0; i < balls.Length; i++)
        {
            startIndex = i * 9;
            GetByteFromBool(balls[i].activeInHierarchy, startIndex, ref ballPacketBuffer);
            GetBytesFromFloat(balls[i].GetComponent<Rigidbody>().position.x, startIndex + 1, ref ballPacketBuffer);
            GetBytesFromFloat(balls[i].GetComponent<Rigidbody>().position.z, startIndex + 5, ref ballPacketBuffer);
        }
        SendP2PPacket(receiverID, ballPacketBuffer);
    }
```

This method sends the `ballPacketBuffer` packet. It uses two helper functions:

- `GetByteFromBool()` to translate the information on a **ball** GameObject state
- `GetBytesFromFloat()` to translate the ball position.

Those packets are sent over the default channel 0.

#### Networking.ReadPacketWithPlayerShot

```c#
    void ReadPacketWithPlayerShot(byte[] switchPacketReceived)
    {
        Debug.Log("Packet switch received");

        GameManager.FoulEnum foul;
        GameManager.BallColorEnum ballOn;
        int shotScore;

        if (switchPacketReceived != null)
        {
            foul = (GameManager.FoulEnum)GetIntFromBytes(0, ref switchPacketReceived);
            ballOn = (GameManager.BallColorEnum)GetIntFromBytes(4, ref switchPacketReceived);
            shotScore = GetIntFromBytes(8, ref switchPacketReceived);

            GameManager.Instance.foul = foul;
            GameManager.Instance.ballOn = ballOn;
            GameManager.Instance.shotScore = shotScore;

            GameManager.Instance.ShotShort();
        }
    }
```

This method is used inside the `OnP2PPacketAvailable()` method of `NetworkingListener()` when the received packet comes to channel 1. Two enumerator variables, `foul` and `ballOn`, are instantiated, together with the `shotScore` integer variable.

The `GetIntFromBytes()` helper function is used to translate information about the last shot from the received packet. Translated information on the foul type and the next ball color is cast to **GameManager.FoulEnum** and **GameManager.BallColorEnum**, and assigned to the `foul` and `ballOn` variables respectively. The integer with the actual score is assigned to `shotScore`. Then, all three variables are assigned to their counterparts in **GameManager.Instance**, which sets their actual in-game values.

#### Networking.ReadPacketWithBallPositions

```c#
    void ReadPacketWithBallPositions(byte[] positionPacketReceived)
    {
        Debug.Log("Packet position received");

        int startIndex = 0;
        float x;
        float z;
        bool active;
        if (positionPacketReceived != null)
        {
            for (int i = 0; i < balls.Length; i++)
            {
                startIndex = i * 9;
                active = GetBoolFromByte(startIndex, ref positionPacketReceived);
                x = GetFloatFromBytes(startIndex + 1, ref positionPacketReceived);
                z = GetFloatFromBytes(startIndex + 5, ref positionPacketReceived);
                balls[i].SetActive(active);
                balls[i].GetComponent<Rigidbody>().MovePosition(new Vector3(x, balls[i].GetComponent<Rigidbody>().position.y, z));
            }
        }
    }
```

This method is used inside the `OnP2PPacketAvailable()` method of `NetworkingListener()` when the received packet comes to channel 0. The `for` loop goes through all of the balls and assigns proper values to their positions and to the flag on whether they are active in the scene at that time.

It uses `GetBoolFromByte()` method to translate the byte received in a packet containing the information on the ball status in the game scene (1 when the ball is active and hasn’t been put in a pocket yet) into the `active` Boolean variable. Then, this variable is used as a parameter for the Unity `SetActive()` method to set the ball as active or not.

The `GetFloatFromBytes()` helper function is used to translate the bytes from the received packet containing the information on the **x** and **z** coordinates of the ball position. The `x` and `z` variables are then used to change the position of the ball with the `Unity MovePosition(Vector3)` method.

#### Helpers Functions

Before sending the packet info, we manually encode each piece of information into byte arrays and form a packet from a collection of those. We have created methods for transforming the information from their default data types to byte arrays and adding them to the packet byte array at a specific index. You can find these methods at the very bottom of this class, in the *Helpers* region.

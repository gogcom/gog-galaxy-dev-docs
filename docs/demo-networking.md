# Networking

## Description

The **Networking** class is responsible for sending and reading peer-to-peer (P2P) packets in a multiplayer game. Packets are sent to a user in the same lobby, specified with that user’s GalaxyID (`receiverID`).

In our demo, we have implemented two types of P2P packets with different content: **switchPacket** (represented by the `switchPacketBuffer` variable), which contains the information about the result of the player’s turn, and **ballPacket** (represented by the `ballPacketBuffer` variable), which contains the positions of all pool balls in the game world.

## Class Initialization and Termination

The **Networking** class is initialized when a online 2-player game begins.

This class initializes variables of a byte array type, which store P2P packets (`switchPacketBuffer` and `ballPacketBuffer`). In order to retrieve the positions of balls and then send them in packets, an array with all of the ball objects from the **GameManager** class is assigned to `private GameObject[] balls` inside the `OnEnable()` method.

## Definitions of Listeners

This class initializes only one listener — **NetworkingListener()** — which receives callbacks when a P2P packet is available.

### NetworkingListener

The purpose of `NetworkingListener()` is to listen for the events related to packets that come to the client. We use this listener in the demo because it only has 2-player lobbies in which the players can communicate directly. If the goal is to send packets to the entire lobby, and not to a particular user, you should consider using **ServerNetworkingListener()** instead.

The `OnP2PPacketAvailable()` method checks if a new packet is received. Inside this method, we peek the packet (if it is available) using the GOG GALAXY SDK [`PeekP2PPacket()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1INetworking.html#ae17ea8db06874f4f48d72aa747e843b8) method, and assign it to the `packetReceived` variable in order to read it and translate from a byte array to an array.

We use two channels to send and receive packets, so we can easily differentiate between packets containing ball positions from the ones with information that a player’s turn was finished. To check this, we use the `switch` statement to distinguish packets from the two channels and to use a proper reading method for the received packet.

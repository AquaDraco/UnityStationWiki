##  The ECS Architecture
Unitystation is following ECS patterns: [https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system) 

So that means every behavior should be broken up into a component. I.E. PickUpTrigger.cs is a component that you can add to your item and it will allow players to pick it up and put it in their inventories. 

## Networking (Unet)

The networking code is all done in Unet: https://docs.unity3d.com/Manual/UNet.html

Basically all of the networking code is written into the same component for both the serverside logic and the clientside logic. This can get quite confusing for people who have never done any netcode before but after debugging or creating new networked features it becomes really easy to understand.

You need to derive your script from NetworkBehaviour and also have a NetworkIdentity on your object. Also make sure when you are done with your object to turn it into a prefab and place it inside a resources folder so that the NetworkManager can find it and cache it for use in multiplayer. 

**Warning:** Sometimes if you press play without the object being in a resources folder you will screw the internal Unet cache up as it routes all of the networked members and you will need to remove the prefab and start again

### There are three methods to creating a networking communication:
1. Command and ClientRpc, these are called Remote Actions: https://docs.unity3d.com/Manual/UNetActions.html
    - [Command]'s are methods that are called on the server remotely by a client
    - [ClientRpc]'s are methods that are called on all clients remotely by the server
    - Remember that you need to name your methods beginning with Cmd or Rpc respectively (i.e. RpcUpdateFire())
    - These are good for general purpose communications that all clients need to know about and are generally used for actions that do not need a high level of security
  

2. SyncVars and SyncVars with hooks: https://docs.unity3d.com/ScriptReference/Networking.SyncVarAttribute.html
    - [SyncVar] can be used to update a variable serverside and if it update on all of the clients. Also new clients who join half way through the game will get the updated value
    - [SyncVar(Hook="MethodNameHere")] SyncVar hooks can be used to call a method whenever the SyncVar variable is updated. Useful for creating an action on the client whenever a server updates it
    - These are good for general purpose communications as all clients will get the update. Do not use if you need to shield data for a specific client from other clients
    - Not good for variables that are updated rapidly (like movement etc)

3. NetMessages: https://docs.unity3d.com/Manual/UNetMessages.html
    - NetMessages are objects that can carry any serializable data and are targetted to specific components on the receiving end
    - Derive your netmsg from ClientMessage if you want to create a message to send from Client to the Server
    - Derive your netmsg from ServerMessage if you want to create a message to send from Server to the Client
    - Net Messages allow you to send data to specific clients so that others do not get the data and is good for secure communications
    - NetMessage create a small about of garbage for the GarbageCollector so keep that in mind

## The Matrix

The matrix keeps track of every item in the game and its grid position. You can query the matrix to find out what objects are at a specific co-ordinate. Its also used to detect collisions when players are moving and is currently being updated to hold all of the atmos data that is simulated on the server. The matrix sits on top of the Tilemap system.

## The Tilemap System

The Tilemap System (https://docs.unity3d.com/Manual/Tilemap.html) is a unity tile management feature which allows us to create maps really quickly. You can experiment with the mapping by clicking on Window--> Tile Palette


###  The ECS Architecture
Unitystation is following ECS patterns: [https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system) 

So that means every behavior should be broken up into a component. I.E. PickUpTrigger.cs is a component that you can add to your item and it will allow players to pick it up and put it in their inventories. 

### Networking (Unet)

The networking code is all done in Unet: https://docs.unity3d.com/Manual/UNet.html

Basically all of the networking code is written into the same component for both the serverside logic and the clientside logic. This can get quite confusing for people who have never done any netcode before but after debugging or creating new networked features it becomes really easy to understand.

There are three methods to creating a networking communication:
1. Command and ClientRpc, these are call Remote Actions: https://docs.unity3d.com/Manual/UNetActions.html
    - [Command]'s are methods that are called on the server remotely by a client
    - [ClientRpc]'s are methods that are called on all clients remotely by the server
    - Remember that you need to name your methods beginning with Cmd or Rpc respectively (i.e. RpcUpdateFire())
    - These are good for general purpose communications that all clients need to know about and are generally used for actions that do not need a high level of security
  

2. 
# Introduction
In this guide we'll explore the fundamental concepts and components you'll need in order to create items in the UnityStation project.

If you want to go straight to the nitty-gritty, check out the TL;DR section.

# Prefabs and you

**Prefabs** are the Unity way of creating templates of objects that will be re-used several times in a scene or across several scenes, such as scenery, items or NPC's.

To create a prefab, create an empty GameObject in your scene, name the object, attach whatever components the prefab should have to the object and drag it into the asset view, as demonstrated here.

![Prefab](https://i.imgur.com/jRnlGTN.gif)

A prefab asset will be created in the asset viewer and the name of the object in your scene turns blue, indicating that the object is now a prefab.

You can now either drag your prefab into the scene in order to instantiate more copies of it, or you can use the method `Instantiate(original, ...)` from a script, where original is a reference to the prefab you just created.

## A note about changing a prefab
Instanced prefabs will be independent. This means that if you make any changes to a prefab, no other prefab will be changed.

However, you might want to apply a change to all prefabs in your scene. 

To assist you, the inspector will show an additional three buttons at the top of the inspector when you inspect a prefab: select, revert and apply.
1. Select will select the prefab asset in your assets.
2. Revert will revert any changes to that of the prefab in your assets.
3. Apply will apply any changes to that prefab to all prefabs of the same type.

Any changes made to the asset prefab will also apply to all prefabs instantiated from it.

References:
* https://docs.unity3d.com/Manual/Prefabs.html
* https://docs.unity3d.com/Manual/InstantiatingPrefabs.html
* https://unity3d.com/learn/tutorials/topics/interface-essentials/prefabs-concept-usage

# The Network Identity Component
The **Network Identity component** is an essential component for working with Unity Networking ([UNet](https://docs.unity3d.com/Manual/UNet.html)).

It ensures that the GameObject has a unique identity (called the Network Instance Id) and is synced across clients and servers, and as such most GameObject used in UnityStation will need a Network Identity component.

The Network Identity component only has two toggleable properties: Server Only and Local Player Authority.

Server Only means that the object is only spawned on the server.

Local Player Authority gives authoritative network control to the player, which means leaving the responsibility for the GameObject to the client instead of the server.

![Network Identity](https://i.imgur.com/gQvX5fm.png)

References:
* https://docs.unity3d.com/Manual/UNet.html
* https://docs.unity3d.com/Manual/class-NetworkIdentity.html
* https://docs.unity3d.com/Manual/UNetConcepts.html
* https://docs.unity3d.com/Manual/UNetAuthority.html
* https://docs.unity3d.com/Manual/UNetUsingHLAPI.html

# Items
Items in unity station are composed of four primary components, **Item Attributes**, **Custom Net Transform**, **Network Identity** and the **Pickupable** component. Of these, Custom Net Transform, Network Identity and Pickupable can just be attached to an object without any further changes to the fields provided by these components.

Other components you might wish to attach are **Food Behaviour** for food items or components derived **Health Behaviour** for items that can be damaged.

## The Item Attributes component

The **Item Attributes** component defines the basic characteristics of an item.

The component exposes the following public fields:
* Cloth: **(NOT CURRENTLY IN USE)**
* Clothing reference: Sprite position on the clothing sprite sheet.
* Hierarchy: This one is special, we will return to it in another section.
* In hand reference Left & Right: Sprite positions from the Inhand sprite sheet.
* Item Description: A string describing the item.
* Item Name: The name of the item.
* Size: The size of the item.
* Sprite Type: An enum describing what type of sprite the item uses, and more importantly from which sprite-sheet the sprite comes. Possible values are **Items**, **Clothing**, and **Guns**.
* Type: An enum describing what type of item it is.
* Network channel: QoS channel to use for updates for this script, derived from Network Behaviour.
* Network send interval: Attribute which is derived from NetworkBehaviour. Determines maximum update rate in seconds.

## The Hierarchy attribute.
The hierarchy attribute in Item Attributes can be used to parse SS13 metadata from the hier.txt file.
An example of a hier string would be, for optical meson scanners, `/obj/item/clothing/glasses/meson`.
After you fill this in, it would grab sprites, inhand references, description, other attributes from the metadata automatically.

![Item attributes](https://i.imgur.com/4LseziQ.png)

## The Custom Net Transform Component
The **Custom Net Transform** component is a component that allows items to be moved in various ways, such as being pushed, appear and disappear.

The Custom Net Transform component derives from `ManagedNetworkBehaviour` which in turn derives from `NetworkBehaviour`.
`NetworkBehaviour` is a UNet component that in turn derives from `MonoBehaviour`, and as such it is intended to be used where MonoBehaviour would normally be used outside of a UNet context.

The Managed Network Transform component also provides the `OnEnable()` and `OnDisable()` methods that add or remove the object instance to the custom UpdateManager.

The component exposes the following fields:
* Speed multiplier: Factor for flying/lerp'ing speed. Higher numbers mean objects travel faster.
* Is Pushing: Enables the item to push other objects.
* Predictive Pushing: Tells the server to try to predict where the item will be pushed.

It also exposes the Network Channel and Network Send Interval variables from NetworkBehaviour.

![Custom net transform](https://i.imgur.com/APfCmFX.png)

References:
* https://docs.unity3d.com/Manual/class-NetworkBehaviour.html
* https://docs.unity3d.com/ScriptReference/Networking.NetworkBehaviour.html

# Pickupable
The **Pickupable** component allows an item to be picked up. Contains some additional server-side functionality for predicting the success or failure of an attempt to grab the item.

# The Custom Network Manager and caching
The **custom network manager** component is a singleton component that manages the network.

This means that it handles connections, syncing player data and more importantly (with regards to items) it caches all prefabs that have attached Network Identity components at startup so that each item has the same Network Instance number across all clients.

It does this by calling the `SetSpawnableList()` method and loading all prefabs in all **Resources** folders and then proceeds to store the prefabs in the `SpawnPrefabs` list inherited from NetworkManager.

References:
* https://docs.unity3d.com/ScriptReference/Networking.NetworkManager.html

# Adding your item to the scene
In order to place your item in the game, you will need to add it to the tile palette.
To do that, 
1. Drag your prefab into the scene.
2. find it in the hierarchy and drag it inside the Matrix objects layer. In the current development version that would be in the `OutpostDeathmatch` scene, `OutpostStation->Level 0->Objects`.
3. Lastly, use the **Transpose** tool in order to align the prefab to the grid.

# TL;DR
This is for the people that just want to add a new item.

Use the following asset workflow:
1. Create your GameObject and name it. 
2. Add the **Item Attributes** component and it will tell you all the other required components to add, which are: 
    * **Network Identity**
    * **Custom Net Transform**
    * **Pickupable**
    * **Object Behaviour**
    * **Register Item**
    * **Sprite Rotation Matrix**
    * Plus any other components which are necessary.
3. Edit the fields to suit your needs.
4. Add your prefab to suitable directory under `Assets/Resources/Prefabs/Items/[ Item Type ]/`.

## OR

Clone existing prefab matching the item you wish to create and edit that.
- [Overview](#overview)
- [Usage and FAQs](#usage-and-faqs)
  * [How do I implement an interaction?](#how-do-i-implement-an-interaction-)
  * [Can I have multiple Interactable Components on an object?](#can-i-have-multiple-interactable-components-on-an-object-)
  * [How do I inform the client what happened?](#how-do-i-inform-the-client-what-happened-)
  * [I don't need any networking, I just want client-side interaction logic](#i-don-t-need-any-networking--i-just-want-client-side-interaction-logic)
  * [I implemented an interaction but my component isn't getting triggered, why?](#i-implemented-an-interaction-but-my-component-isn-t-getting-triggered--why-)
  * [Why should I override WillInteract?](#why-should-i-override-willinteract-)
  * [How can I implement client side prediction and rollback?](#how-can-i-implement-client-side-prediction-and-rollback-)
- [Reference](#reference)
  * [Interaction Types](#interaction-types)
  * [Precedence of Interaction Components](#precedence-of-interaction-components)
  * [Interaction Logic Flow](#interaction-logic-flow)

For info on the Right Click Menu, see [[Right Click Menu]].

This page describes the interaction system for Unitystation, also called Interaction Framework 2 (IF2), due to being a replacement of the previous approach to interactions.

All of the code lives in Scripts/Input System/InteractionV2 and is also heavily documented if you need further info.

# Overview
Interactions are implemented by way of **Interactable Components**, or **ICs** for short. These are components which extend Interactable (see InteractionV2/CoreComponents) or 
implement the core IF2 interfaces (see the IInteractable interface).

Additionally, each component can support one or more different **Interaction Types**. These interaction types represent the different sorts of things a user can do to interact with the game.
Here's a brief description of the current interaction types:

  * HandApply - Click something in the game world. The item in the active hand (or empty hand) is applied to the thing that was clicked. Targets a specific object or tile.
  * HandActivate - Triggers when using the Activate (defaults to "Z") key or clicking the item while it is in the active hand.
  * InventoryApply - Like HandApply, but targeting something in the inventory rather than in the world. Triggers when clicking an item in the inventory when the active hand has an item or is empty (in which case UsedObject will be null).
  * AimApply - like hand apply, but does not have a specific targeted object (it simply aims where the mouse is) and can occur at some interval while the mouse is being held down after being clicked in the game world. For things like shooting a semi-auto or automatic weapon, spraying fire extinguisher, etc...
  * MouseDrop - Click and drag a MouseDraggable object in the game world and release it to drop.
  * PositionalHandApply - like hand apply, but stores (and transmits) the specific position on the object that was clicked - useful for large objects which have different behavior based on where they are clicked (such as tilemaps). This is separate from HandApply so that the length of netmessages can be reduced (the vector of the position is only added to the message for PositionalHandApply but can be excluded for HandApply).
  
For example, here's a simple IC, HasNetTab, which pops up a console tab (such as the shuttle console) when the component's object is clicked:
```csharp
public class HasNetworkTab : Interactable<HandApply>
{
	[Tooltip("Network tab to display.")]
	public NetTabType NetTabType = NetTabType.None;

    //This is invoked server side when this component's object is clicked on the client
	protected override void ServerPerformInteraction(HandApply interaction)
	{
		TabUpdateMessage.Send( interaction.Performer, gameObject, NetTabType, TabAction.Open );
	}
}
```

As shown above, by Interactable, the interactable component gains a few nice capabilities:
  * Automatic networking - IF2 takes care of informing the server of the interaction. All you need to implement is the server-side logic of the interaction and a way to communicate the result back to the client (if needed).
  * No mouse / keyboard logic - IF2 figures out when your component's interaction logic should be invoked.
  * Interaction info object - the HandApply class contains all the info you should need in order to decide what should happen.
  * Customization - if the default behavior doesn't meet your needs, you can override additional methods to customize how it works. This can be done to implement client side prediction, 
    or to change the circumstances under which the interaction logic should fire (reducing the amount of messages sent to the server).

# Usage and FAQs

The following sections describe IF2 in more detail for various use cases.

## How do I implement an interaction?
If your interaction involves using one object on another, you should first decide which side of the interaction you want the component to live on. For example, if you have 
a machine that starts emitting radiation when you use an Emag on it, you could put the component on the machine or the Emag object.

Once you've decided, create a new component (or modify an existing one on that object) that extends Interactable (see InteractionV2/CoreComponents) corresponding to the interaction type you want to handle:

```csharp
public class MyInteractableComponent : Interactable<HandApply>
```

For example, you can extend Interactable&lt;HandApply> or Interactable&lt;HandActivate>. Or, if you need to support both types of interactions, extend Interactable&lt;HandApply, HandActivate>.

Interactable&lt;> extends MonoBehavior. If you need access to NetworkBehavior features like SyncVar, you can instead extend NB(Interaction)Interactable. For example, NBHandApplyInteractable:
```csharp
public class MyInteractableComponent : NBHandApplyInteractable {
	//now we can use syncvars
	[SyncVar] bool isOnFire;
```

Now you need to implement the interaction logic. Here's an example interactable component ExplodeWhenWelded, which explodes when a player tries to weld it. This demonstrates all of the overridable methods
and what they do:
```csharp
public class ExplodeWhenWelded : Interactable<HandApply>
{

  //this method is invoked on the client side before informing the server of the interaction.
  //If it returns false, no message is sent.
  //If it returns true, the message is sent to the server. Then this is invoked on the server side, and if 
  //it returns true, the server finally performs the interaction.
  //We don't NEED to implement this method, but by implementing it we can cut down on the amount of messages
  //sent to the server.
  public override bool WillInteract(HandApply interaction, NetworkSide side)
  {
    //the base method defines the "default" behavior for when a HandApply should occur, which currently
	//checks if the player is conscious and standing next to the thing they are clicking.
    if (!base.WillInteract(interaction, side)) return false;
	
    //we only want this interaction to happen when a lit welder is used
    var welder = interaction.HandObject != null ? interaction.HandObject.GetComponent<Welder>() : null;
    if (welder == null) return false;
    if (!welder.isOn) return false;
    return true;
  }
  
  //this is invoked when WillInteract returns true on the client side.
  //We can override to add client prediction logic to make the game feel more responsive.
  public override void ClientPredictInteraction(HandApply interaction)
  {
    //display an explosion effect on this client that has no effect on their actual health
	DisplayExplosion();
  }
  
  //invoked when the server recieves the interaction request and WIllinteract returns true
  public override void ServerPerformInteraction(HandApply interaction)
  {
    //Server-side trigger the explosion and inform all clients of it
	Explode(interaction.TargetObject);
  }
}
```

An additional note about WillInteract - there are useful util methods you can use in Validations.cs which are designed to be used in WillInteract.

## Can I have multiple Interactable Components on an object?
Yes, and this is encouraged if the interaction logic makes sense as its own component. For example, all objects which can be picked up have a Pickupable component,
yet some objects also have other interactable components for their object-specific interactions.

When there are multiple interactable components on an object for the same interaction type, IF2 checks the components in the order
they appear in the GameObject's component list in Unity (drag to rearrange them). It triggers the first component whose WillInteract method
returns true.

Additionally, if the interaction involves multiple objects (such as using an item on a machine), the components are checked
on the used object first and the target object second. If ANY interactable component's WillInteract method returns true,
that component's interaction logic is triggered and no further components are checked.

Refer to the section "Precedence of Interaction Components" for the full details.

## How do I inform the client what happened?
In ServerPerformInteraction, if the server makes some change to the game state, usually they will need to ensure the client
knows about the new state. This is currently not part of IF2, but there are various ways this can be done, many of which
can be accomplished using already implemented methods...For the most part, you can update a SyncVar, broadcast a net message to all clients or just one,
or invoke a ClientRpc. Refer to the other articles on this wiki which discuss networking for more information.


## I don't need any networking, I just want client-side interaction logic
Sometimes you may not need any of the networking features of Interactable (such as already having Cmds or other messages
that handle it for you), or you have an interaction which only has an effect client-side. In this case, you don't need to extend Interactable. You can 
instead implement IInteractable (which is an interface rather than an abstract class):
```csharp
public class MyClientSideInteraction : MonoBehavior, IInteractable<HandApply>
{

  //invoked when this is clicked. 
  //Return value convention is the same as WillInteract, except this is only invoked on the client side.
  //If it returns true, the interaction is "consumed" - no more components will recieve the current interaction.
  //If it returns false, the interaction is "passed on" - additional components will recieve the interaction.
  public bool Interact(HandApply interaction)
  {
     //do something client side, or send a message to the server or invoke a Cmd
	 //if we did something
		return true;
	 //if we didn't
		return false;
  }
}
```

## I implemented an interaction but my component isn't getting triggered, why?
There's a few things to check:
1. Does your component implement WillInteract? If so, check if it is returning true when you try to interact. If it returns false, your component's interaction will not be triggered.
  If there is no Willinteract method, take a look at the DefaultWillInteract class to see what default logic
  is being used.
2. Are there any other interactable components on the object or the other object involved in the interaction? If so, check if they appear first in the object's component list,
	and check if they have Willinteract methods which are returning true. To fix this, there are a few options to consider:
    * add more logic to another object's WillInteract method
	* rearrange the components so your interaction comes first
	* move your interactable component to the other object involved in the interaction
	
## Why should I override WillInteract?
You don't HAVE to override WillInteract - you could just put all of your validation logic and checks in ServerPerformInteraction. This will result in the 
logic in DefaultWillInteract.cs being used for the WillInteract check. This is probably fine for many situations.

However, this can cause some problems:
  * Your component's interaction logic will always be triggered when it recieves an interaction, preventing other interactable components on the object from receiving the interaction.
  * Your component will send interaction messages to the server more often, even when the ServerPerformInteraction logic would not end up doing anything. This increases the network 
    load on the server.

Instead, if you implement WillInteract so that it only returns true when your interaction logic acually has something to do, then you can avoid those problems and improve network performance.

## How can I implement client side prediction and rollback?
Client side prediction makes the game appear a lot more responsive to the user, even if they are having to wait for the server
to process their action. Basically, the client predicts what the interaction will do and updates the game state accordingly for the user. However, if it later turns
out that the server disagrees with that prediction, the client needs to "roll back" the prediction, resetting its state to whatever the server says is correct.

For client side prediction, you can override ClientPredictInteraction. This is invoked when the client is sending the message to the server after WillInteract returns true.
In that method, you can make your prediction and update the game state for the local client.

For rollback, you can write a rollback method which informs the client (sending a net message, updating a SyncVar, invoking ClientRpc, etc...) what to roll back to.
Then, call this method from your server-side WillInteract logic (side == NetworkSide.Server) and/or your ServerPerformInteraction methods when the interaction doesn't do what the 
client would've predicted.

There is a util method called Validations.ValidateWithServerRollback which can be used to simplify your WillInteract logic if you are doing prediction. See how it is used
in Pickupable.WillInteract.

# Reference
The remaining sections server as a reference for the details of IF2.

## Interaction Types
Here are the current interactions. More may be added as different objects require different use cases:
* MouseDrop - Click and drag a MouseDraggable object in the game world and release it to drop.
* HandApply - click something in the game world. The item in the active hand (or empty hand) is applied to the thing that was clicked. Targets a specific object or tile.
* PositionalHandApply - like hand apply, but stores (and transmits) the specific position on the object that was clicked - useful for large objects which have different behavior based on where they are clicked (such as tilemaps). This is separate from HandApply so that the length of netmessages can be reduced (the vector of the position is only added to the message for PositionalHandApply but can be excluded for HandApply).
* AimApply - like hand apply, but does not have a specific targeted object (it simply aims where the mouse is) and can occur at some interval while the mouse is being held down after being clicked in the game world. For things like shooting a semi-auto or automatic weapon, spraying fire extinguisher, etc...
* HandActivate - Triggers when using the "Z" key or clicking the item while it is in the active hand.
* InventoryApply - Like HandApply, but targeting something in the inventory rather than in the world. Triggers when clicking an item in the inventory when the active hand has an item or is empty (in which case UsedObject will be null).

## Precedence of Interaction Components
This list indicates the current order of precedence for checking for an interaction on a given frame. Consider this an "abridged version" of the next section.

Remember that there can be multiple components on the used object or the targeted object which implement IInteractable&lt;>, for multiple interaction types, so this list can help you figure out which will be invoked first. Further checking of interactions will be stopped as soon as any of these components indicates that an interaction has occurred (in the old system, this is done by returning true from InputTrigger, in IF2 it is done by returning InteractionControl.STOP_PROCESSING).

1. alt click
2. throw
3. PositionalHandApply
    1. Components on used object (for the object in the active hand, if occupied), in component order.
    2. Components on target object in component order.
3. HandApply
    1. Components on used object (for the object in the active hand, if occupied), in component order.
    2. Components on target object in component order.
5. AimApply (this runs last so you can still melee / click things if adjacent when a gun is in hand)
    1. Components on used object (object in the active hand), in component order.

## Interaction Logic Flow
Because the mouse can do so many things, the logic for interactions is a bit complicated. This section describes it in detail.

Due to wanting to make guns more usable, there are 2 main different cases - when you have a loaded gun in the active hand vs. not.

Alt click and throw are always checked first and have no special logic.

When the active hand doesn't have a loaded gun:
1. Mouse Clicked Down
    1. Is the mouse over an object with a MouseDraggable? We need to wait and see if we should drag it or click on it. 
       Save the MouseDraggable and wait until mouse is dragged or mouse 
       button is released.
    2. If mouse is not over a MouseDraggable...
        1. IF2 - PositionalHandApply - check interactions in the following order until one occurs.
            1. IInteractable&lt;PositionalHandApply> components on used object (for the object in the active hand, if 
               occupied), in 
               component order.
            2. IInteractable&lt;PositionalHandApply> components on target object in component order.
        1. IF2 - HandApply - if no PositionalHandApply interactions happened, check interactions in the following order until one occurs.
            1. IInteractable&lt;HandApply> components on used object (for the object in the active hand, if occupied), in 
               component order.
            2. IInteractable&lt;HandApply> components on target object in component order.
        2. If no HandApply interactions occurred, check the old system to see if a click interaction occurs - uses 
           InputTrigger and stop as soon as one occurs.
        3. If no interactions have occurred, check IF2 AimApply interactions and stop as soon as one occurs. This runs 
           last so you can still melee / click things if adjacent when a gun is in hand)
           1. Checks for IInteractable&lt;AimApply> components on used object (object in the active hand), in component 
              order.
2. Mouse held down.
    1. If we saved a MouseDraggable during the initial click and the mouse has been dragged far enough (past MouseDragDeadzone), initiate a drag and drop (show the drag shadow of the object being dragged).
       Until the object is dropped, no further interactions will occur.
3. Mouse Button Released
    1. If we are dragging something, drop it and trigger MouseDrop interactions in the following order... 
        1. IInteractable&lt;MouseDrop> components on dropped object in 
               component order.
        2. IInteractable&lt;MouseDrop > components on target object in component order.
    2. If we saved a MouseDraggable during the initial click but the mouse never moved past the drag deadzone
       and we have not held the mouse button down longer than MaxClickDuration...
         1. IF2 - PositionalHandApply - check interactions in the following order until one occurs.
            1. IInteractable&lt;PositionalHandApply> components on used object (for the object in the active hand, if 
               occupied), in 
               component order.
            2. IInteractable&lt;PositionalHandApply> components on target object in component order.
        1. IF2 - HandApply - if no PositionalHandApply interactions happened, check interactions in the following order until one occurs.
            1. IInteractable&lt;HandApply> components on used object (for the object in the active hand, if occupied), in 
               component order.
            2. IInteractable&lt;HandApply> components on target object in component order.
        2. If no HandApply interactions occurred, check the old system to see if a click interaction occurs - uses 
           InputTrigger and stop as soon as one occurs.


When there is a loaded gun in the active hand.
1. Mouse Clicked Down
    1. Are we on Harm intent? If so, shoot (trigger IInteractable&lt;AimApply> components on Gun).
    2. If not on Harm intent...
         1. IF2 - PositionalHandApply - check interactions in the following order until one occurs.
            1. IInteractable&lt;PositionalHandApply> components on used object (for the object in the active hand, if 
               occupied), in 
               component order.
            2. IInteractable&lt;PositionalHandApply> components on target object in component order.
        1. IF2 - HandApply - if no PositionalHandApply interactions occurred, check interactions in the following order until one occurs.
            1. IInteractable&lt;HandApply> components on used object (for the object in the active hand, if occupied), in 
               component order.
            2. IINteractable&lt;HandApply> components on target object in component order.
        2. If no HandApply interactions occurred, check the old system to see if a click interaction occurs - uses 
           InputTrigger and stop as soon as one occurs.
        3. If no interactions have occurred, check IF2 AimApply interactions and stop as soon as one occurs. This runs 
           last so you can still melee / click things if adjacent when a gun is in hand)
           1. Checks for IInteractable&lt;AimApply> components on used object (object in the active hand), in component 
              order.
2. Mouse held down - continue shooting if we have an automatic (keep triggering IInteractable&lt;AimApply> components on Gun).

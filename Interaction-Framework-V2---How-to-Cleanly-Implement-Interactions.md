- [Using Interaction Framework V2](#using-interaction-framework-v2)
  * [1. Extend Interactable - Most Convenient Approach](#1-extend-interactable---most-convenient-approach)
  * [2. Delegate to InteractionCoordinator - More Control but More Verbose](#2-delegate-to-interactioncoordinator---more-control-but-more-verbose)
  * [3. Implement IInteractable - Most Flexible, Least Convenient](#3-implement-iinteractable---most-flexible--least-convenient)
- [Which Approach Is Best?](#which-approach-is-best-)
- [Interaction Types](#interaction-types)
- [Multiple Interaction Components](#multiple-interaction-components)
- [Precedence of Interaction Components](#precedence-of-interaction-components)
- [Interaction Logic Flow](#interaction-logic-flow)
- [Migration](#migration)
- [Motivation for IF2](#motivation-for-if2)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

For info on the Right Click Menu, see [[Right Click Menu]].

This page describes how to use the new interaction system, which I've dubbed "Interaction Framework V2" (I guess you could call it IF2 for short, at least until everything is ported over to it, and you could call the old system IF1). Please contact me @chairbender on Discord for any questions or concerns.

At the time of writing, everything has been ported over to IF2 and there are no more traces of the old system.

Without further ado, here's the many ways you can use it...

# Using Interaction Framework V2
All code lives in Input System/InteractionV2 and is heavily documented, so refer to that for more info on each part of IF2.

When a player drags and drops their character on a chair, they should become buckled. Here's the various ways this can be implemented, starting with the approach which is most convenient and ending with the approach that provides the most control.

## 1. Extend Interactable - Most Convenient Approach

```csharp
// Extend Interaction and specify the type of interaction as the generic type argument
public class BuckleInteract : Interactable<MouseDrop>
{
    //Define the validations that must be done on client / server side for this interaction.
    protected override InteractionValidationChain<MouseDrop> InteractionValidationChain()
    {
        return InteractionValidationChain<MouseDrop>.Create()
            .WithValidation(IsDroppedObjectAtTargetPosition.IS)
            .WithValidation(DoesDroppedObjectHaveComponent<PlayerMove>.DOES)
            .WithValidation(CanApply.EVEN_IF_SOFT_CRIT)
            .WithValidation(ComponentAtTargetMatrixPosition<PlayerMove>.NoneMatchingCriteria(pm => pm.IsRestrained))
            .WithValidation(new FunctionValidator<MouseDrop>(AdditionalValidation));
    }

    private ValidationResult AdditionalValidation(MouseDrop drop, NetworkSide side)
    {
        //additional custom validation logic which is only needed for this class
    }

    //invoked after client-side validation succeeds - NOT invoked for server player!
    protected override void ClientPredictInteraction(HandApply interaction)
    {
        //can do client-side prediction stuff here if desired
    }

    //invoked on the server after server-side validation fails
    protected override void OnServerInteractionValidationFail(HandApply interaction)
    {
        //Can tell client to rollback their prediction
    }

    // Define what the server should do once the interaction is validated server side
    protected override void ServerPerformInteraction(MouseDrop drop)
    {
        // perform the interaction on the server side and update each
        // client by setting a syncvar that indicates the player is restrained
    }
}
```

The Interactable base class provides the maximum convenience for implementing an interaction component. It inherits from MonoBehavior. If you wish to use NetworkBehavior things (like SyncVar) you must use or create a different base class to extend because
Unity does not allow you to have a generic class that extends NetworkBehavior. See examples of how to do this in InteractionV2/Components, look at any class prefixed with "NB" (for NetworkBehavior), such as NBHandApplyInteractable. All you need to do is extend the generic class and convert it to non-generic by specifying type arguments, then name it "NB(interactions)Interactable". For example, if we want hand apply and mouse drop interaction, we would make it "NBHandApplyMouseDropInteractable". I know it's a long name but it's just trying to get around the inability to use generics.

It works as follows:
1. When the client performs an interaction involving this object, each validator (implementation of IInteractionValidator interface) is invoked. Validators are able to tell if validation is happening on client or server side, so they can customize the validation if needed. All validators live in Input System/InteractionV2/Validations. It is highly recommended to re-use, modify, or implement new validators so that common validation logic can be shared.
2. If all validators succeed, the client sends an interaction message (in this case RequestMouseDropInteract) to the server and the ClientPredictInteraction method is invoked if this isn't the server player (if defined). No further interaction components will be invoked on the client side for this interaction type during the current update.
3. The server gets the RequestMouseDropInteract and invokes all of the validators again (each validator can modify the validation logic on the server-side if needed).
4. If all validators succeed, the server invokes ServerPerformInteraction and from there can do whatever it wants to update the state of the game and communicate it to the players - setting syncvars, invoking Rpcs, or sending messages to clients. If validation fails on the server-side, the OnServerInteractionValidationFail method is invoked (if defined).

If common use cases emerge for the final server-side part of the interactions we can add this to IF2 to reduce code duplication. For example, a likely addition will be having a way to automatically broadcast a message to all clients so they can perform the update client-side, yet make it so that all of the logic for the client-side update still lives in the component that is handling the interaction.

So, if your interaction fits into that pattern, you can use this approach. The limitation is that you must extend the Interactable class so you can't extend anything else.

However, we have provided several versions of Interactable which accept additional interaction types. For example, if we want to implement logic so that the player is unbuckled when the chair is clicked, we would simply change the class definition to:
`public class BuckleInteract : Interactable<MouseDrop,HandApply>`
...and then implement a second set of validators and server-side interaction logic for HandApply.
We have defined Interactable to support from 1-5 generic type arguments.

And, as mentioned above, if you wish the use NetworkBehavior features, you will need to copy the example from
NBHandApplyInteractable or re-use an existing NB Interactable class.

## 2. Delegate to InteractionCoordinator - More Control but More Verbose
```csharp
// we don't extend now - we only implement the interactable and 
// interactionprocessor interfaces,
// and we delegate the implementation of the interface 
// methods to the InteractionCoordinator
public class Chair: IInteractable<MouseDrop>, IInteractionProcessor<MouseDrop>
{
    //...
    //...(Other logic that already existed on our chair component)
    //...

    private InteractionCoordinator<MouseDrop> coordinator;

    private void Start()
    {
        coordinator = new InteractionCoordinator<MouseDrop>(this, 
            InteractionValidationChain(), ServerPerformInteraction);
    }

    private InteractionValidationChain<MouseDrop> InteractionValidationChain()
    {
        return InteractionValidationChain<MouseDrop>.Create()        
            .WithValidation(IsDroppedObjectAtTargetPosition.IS)
            .WithValidation(DoesDroppedObjectHaveComponent<PlayerMove>.DOES)
            .WithValidation(CanApply.EVEN_IF_SOFT_CRIT)
            .WithValidation(ComponentAtTargetMatrixPosition<PlayerMove>
                .NoneMatchingCriteria(pm => pm.IsRestrained))
            .WithValidation(new FunctionValidator<MouseDrop>(AdditionalValidation));
    }

    private ValidationResult AdditionalValidation(MouseDrop drop, NetworkSide side)
    {
        //if the player to buckle is currently downed, we cannot 
        //buckle if there is another player on the tile
        //(because buckling a player causes the tile to become unpassable,
        //thus a player could end up
        //occupying another player's space)
        var playerMove = drop.UsedObject.GetComponent<PlayerMove>();
        var registerPlayer = playerMove.GetComponent<RegisterPlayer>();
        if (!registerPlayer.IsDown) return ValidationResult.SUCCESS;
        return ComponentAtTargetMatrixPosition<PlayerMove>.NoneMatchingCriteria(pm =>
            pm != playerMove &&
            pm.GetComponent<RegisterPlayer>().IsBlocking)
            .Validate(drop, side);
    }

    private void ServerPerformInteraction(MouseDrop drop)
    {
        // server-side interaction logic + inform clients
    }

   //from IInteractable
   //delegate the interface method implementations to coordinator
    public InteractionControl Interact(MouseDrop interaction)
    {
        return coordinator.ClientValidateAndRequest(interaction);
    }

    //from IInteractionProcessor
    public InteractionControl ServerProcessInteraction(MouseDrop interaction)
    {
        return coordinator.ServerValidateAndPerform(interaction);
    }
}
```
The flow for this approach is exactly the same as the first approach, but we are not limited by extending Interactable. 

The biggest advantage in flexibility of this approach is that you can use it on any existing component, since all you need to do is implement the indicated interfaces. In this example, we've added it on to our existing chair component. This CAN be useful to avoid having to create a new component for each new type of interaction, but you should be careful about putting too much logic into a single component. Unity components are designed to be re-usable, and combining lots of logic into a single component can hinder that. In general, I would suggest creating a new component and using approach 1 as a "sensible default". 

The cost of that flexibility is that we have to add extra code to delegate to the coordinator. This is done for us in Interactable. 

The IInteractable interface tells IF2 that our component can process a MouseDrop interaction, as such the implementation of the interface method will be invoked when this component is on an object involved in the interaction. The IInteractionProcessor interfaces tells IF2 that our component can perform the server-side portion of an interaction. In order to use the InteractionCoordinator we have to pass it a component which implements IInteractionProcessor. We could pass it a different gameobject's IInteractionprocessor component if we wanted to, but for most use cases it should make the most sense to implement this directly on this component.

With this approach, we can also modify the implementation of ServerProcessInteraction and Interact so that we are not only limited to the functionality provided by the InteractionCoordinator.

## 3. Implement IInteractable - Most Flexible, Least Convenient
```csharp
public class BuckleInteractDIY : IInteractable<MouseDrop>, IInteractionProcessor<MouseDrop>
{
    //from IInteractable
    public InteractionControl Interact(MouseDrop interaction)
    {
        // We can still re-use validators if we want:
        var validationResult = CanApply.EVEN_IF_SOFT_CRIT.Validate(interaction, NetworkSide.CLIENT);

        // do whatever we want - but remember this is only invoked
        // on the client performing the interaction.
        // It's up to us to communicate with the server and
        // perform validation if needed.
        
        //if desired, we can send a RequestMouseDropMessage like so,
        //indicating this component will process the interaction on the server side
        // in ServerProcessInteraction:
        InteractionMessageUtils.SendRequest(interaction, this);
    }

    //from IInteractionProcessor, this will be invoked when the server gets the RequestMouseDropMessage
    public InteractionControl ServerProcessInteraction(MouseDrop interaction)
    {
        //validate and perform the update server side after it gets the RequestMouseDropMessage,
        //then inform all clients.
    }
}
```
With this approach, we implement IInteractable so that IF2 will invoke this component if the object it's on is involved in that type of interaction. However, it's up to us to handle all the rest of this interaction. If this was something that should only be client-side, for example, we could just implement the client-side logic here. But if there is any communication needed with the server we would have to implement that logic as well. 

We can re-use validators here as well as interaction messages like RequestMouseDropMessage / IInteractionProcessor as shown. Each interaction type has its own message and can be sent using InteractionMessageUtils.SendRequest.

As with the previous approach, we can also implement this on any existing component if we want.
# Which Approach Is Best?
Each approach is a tradeoff between convenience and control. In general I would suggest approach 1. If approach 1 isn't meeting your needs, you can consider adding logic to IF2 to try to capture your specific use case, and follow approach 1's example for how to make it convenient to use. If you do decide to go with approach 2 or 3 be careful of code duplication - try to avoid duplication by building your logic into helper classes or adding functionality directly to IF2 to support your use case.

Be warned that any changes to the core of IF2 will be inspected closely...we want to ensure that interaction code is as clean as possible since interaction is such a foundational aspect of the game.

# Interaction Types
Here are the current interactions. More may be added as different objects require different use cases:
* MouseDrop - Click and drag a MouseDraggable object in the game world and release it to drop.
* HandApply - click something in the game world. The item in the active hand (or empty hand) is applied to the thing that was clicked. Targets a specific object or tile.
* PositionalHandApply - like hand apply, but stores (and transmits) the specific position on the object that was clicked - useful for large objects which have different behavior based on where they are clicked (such as tilemaps). This is separate from HandApply so that the length of netmessages can be reduced (the vector of the position is only added to the message for PositionalHandApply but can be excluded for HandApply).
* AimApply - like hand apply, but does not have a specific targeted object (it simply aims where the mouse is) and can occur at some interval while the mouse is being held down after being clicked in the game world. For things like shooting a semi-auto or automatic weapon, spraying fire extinguisher, etc...
* HandActivate - Triggers when using the "Z" key or clicking the item while it is in the active hand.
* InventoryApply - Like HandApply, but targeting something in the inventory rather than in the world. Triggers when clicking an item in the inventory when the active hand has an item or is empty (in which case UsedObject will be null).

# Multiple Interaction Components
In the old system, you could only have one interaction component per object. In the new system, **you can have multiple components on an object which implement interaction logic, even for the same type of interaction**. This means you no longer should need to do things like extending PickupTrigger. You can define a Pickupable component and a separate component for that object's specific interaction logic.

# Precedence of Interaction Components
This list indicates the current order of precedence for checking for an interaction on a given frame. Consider this an "abridged version" of the next section.

Remember that there can be multiple components on the used object or the targeted object which implement IInteractable<>, for multiple interaction types, so this list can help you figure out which will be invoked first. Further checking of interactions will be stopped as soon as any of these components indicates that an interaction has occurred (in the old system, this is done by returning true from InputTrigger, in IF2 it is done by returning InteractionControl.STOP_PROCESSING).

1. Old system - alt click.
2. Old system - throw
3. IF2 - PositionalHandApply
    1. Components on used object (for the object in the active hand, if occupied), in component order.
    2. Components on target object in component order.
3. IF2 - HandApply
    1. Components on used object (for the object in the active hand, if occupied), in component order.
    2. Components on target object in component order.
4. old system - click (checks for InputTrigger).
5. IF2 - AimApply (this runs last so you can still melee / click things if adjacent when a gun is in hand)
    1. Components on used object (object in the active hand), in component order.

# Interaction Logic Flow
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
     
# Motivation for IF2

We've all noticed some issues working with the old InputTrigger-based interaction system. Since interaction is such a vital and foundational part of the game, we decided it would be best to make it as polished as possible. The key design goals of the new system are:
* Minimize code duplication for existing and new interactions. There was a lot of this going on in the old system due to various challenges with trying to re-use code between components. The new system provides better ways to share common interaction logic, and should be easier to add on to when additional common use cases become needed.
* Flexibility. The old system was completely based on inheritance from a base class, and extending that class was pretty much the only way of implementing a new interaction. The new system provides flexibility by providing different ways of implementing interactions, where each way provides different levels of control vs. convenience. Additionally, the core of the new system is based on interfaces and containment rather than inheritance, so it should be easier to add functionality to the core system as more use cases become apparent.
* Clarity and consistency. The old system had many abstract methods in the base class which would be invoked under various unclear circumstances, and whose names did not make it obvious what those situations were. The new system provides a clear vocabulary and definitions of each different kind of interaction.
* Separation of concern. In the old system, the client / server side portions of interactions sometimes would happen in the component, but sometimes would also happen in the NetMessage that corresponded to that interaction. In the new system, it's easier to keep all of the logic related to the interaction on both sides of the network all in a single component (though still possible to deviate from this if you really wanted to, but this should rarely / never be needed).
* Networking support. The old system was basically a free-for-all in terms of how you could implement networking features, even though MANY of the interactions followed the same pattern of communication between client and server. The new system retains the flexibility of the old system while also providing functionality for common networking use cases that show up in our interaction system. This should simplify the networking aspect of interaction, making it easier to work with and understand.


This page describes how to use the new interaction system, which I've dubbed "Interaction Framework V2" (I guess you could call it IF2 for short, at least until everything is ported over to it, and you could call the old system IF1). Please contact me @chairbender on Discord for any questions or concerns.

We've all noticed some issues working with the old InputTrigger-based interaction system. Since interaction is such a vital and foundational part of the game, we decided it would be best to make it as polished as possible. The key design goals of the new system are:
* Minimize code duplication for existing and new interactions. There was a lot of this going on in the old system due to various challenges with trying to re-use code between components. The new system provides better ways to share common interaction logic, and should be easier to add on to when additional common use cases become needed.
* Flexibility. The old system was completely based on inheritance from a base class, and extending that class was pretty much the only way of implementing a new interaction. The new system provides flexibility by providing different ways of implementing interactions, where each way provides different levels of control vs. convenience. Additionally, the core of the new system is based on interfaces and containment rather than inheritance, so it should be easier to add functionality to the core system as more use cases become apparent.
* Clarity and consistency. The old system had many abstract methods in the base class which would be invoked under various unclear circumstances, and whose names did not make it obvious what those situations were. The new system provides a clear vocabulary and definitions of each different kind of interaction.
* Separation of concern. In the old system, the client / server side portions of interactions sometimes would happen in the component, but sometimes would also happen in the NetMessage that corresponded to that interaction. In the new system, it's easier to keep all of the logic related to the interaction on both sides of the network all in a single component (though still possible to deviate from this if you really wanted to, but this should rarely / never be needed).
* Networking support. The old system was basically a free-for-all in terms of how you could implement networking features, even though MANY of the interactions followed the same pattern of communication between client and server. The new system retains the flexibility of the old system while also providing functionality for common networking use cases that show up in our interaction system. This should simplify the networking aspect of interaction, making it easier to work with and understand.

Without further ado, here's the many ways you can use it...

# Using Interaction Framework V2
All code lives in Input System/InteractionV2 and is heavily documented, so refer to that for more info on each part of IF2.

When a player drags and drops their character on a chair, they should become buckled. Here's the various ways this can be implemented, starting with the approach which is most convenient and ending with the approach that provides the most control.

## 1. Extend CoordinatedInteraction - Most Convenient Approach

```csharp
// Extend CoordinatedInteraction and specify the type of interaction as the generic type argument
public class BuckleInteract : CoordinatedInteraction<MouseDrop>
{
    //Define the validations that must be done on client / server side for this interaction.
    protected override IList<IInteractionValidator<MouseDrop>> Validators()
    {
        return new List<IInteractionValidator<MouseDrop>>
        {
            //use existing validators so we can re-use common validation logic
            IsDroppedObjectAtTargetPosition.IS,
            DoesDroppedObjectHaveComponent<PlayerMove>.DOES,
            CanApply.EVEN_IF_SOFT_CRIT,
            ComponentAtTargetMatrixPosition<PlayerMove>
                .NoneMatchingCriteria(pm => pm.IsRestrained),
            new FunctionValidator<MouseDrop>(AdditionalValidation)
        };
    }

    private ValidationResult AdditionalValidation(MouseDrop drop, NetworkSide side)
    {
        //additional custom validation logic which is only needed for this class
    }

    // Define what the server should do once the interaction is validated server side
    protected override InteractionResult ServerPerformInteraction(MouseDrop drop)
    {
        // perform the interaction on the server side and update each
        // client by setting a syncvar that indicates the player is restrained

        return InteractionResult.SOMETHING_HAPPENED;
    }
}
```


The InteractionCoordinator base class provides the maximum convenience for implementing an interaction component. It works as follows:
1. When the client performs an interaction involving this object, each validator (implementation of IInteractionValidator interface) is invoked. Validators are able to tell if validation is happening on client or server side, so they can customize the validation if needed. All validators live in Input System/InteractionV2/Validations. It is highly recommended to re-use, modify, or implement new validators so that common validation logic can be shared.
2. If all validators succeed, the client sends a RequestInteractMessage to the server. No further interaction components will be invoked on the client side for this interaction type during the current update.
3. The server gets the RequestInteractMessage and invokes all of the validators again (each validator can modify the validation logic on the server-side if needed).
4. If all validators succeed, the server invokes ServerPerformInteraction and from there can do whatever it wants to update the state of the game and communicate it to the players - setting syncvars, invoking Rpcs, or sending messages to clients. 

If common use cases emerge for the final server-side part of the interactions we can add this to IF2 to reduce code duplication. For example, a likely addition will be having a way to automatically broadcast a message to all clients so they can perform the update client-side, yet make it so that all of the logic for the client-side update still lives in the component that is handling the interaction.

So, if your interaction fits into that pattern, you can use this approach. The limitation is that you must extend the CoordinatedInteraction class so you can't extend anything else.

However, we have provided several versions of CoordinatedInteraction which accept additional interaction types. For example, if we want to implement logic so that the player is unbuckled when the chair is clicked, we would simply change the class definition to:
`public class BuckleInteract : CoordinatedInteraction<MouseDrop,HandApply>`
...and then implement a second set of validators and server-side interaction logic for HandApply.
We have defined CoordinatedInteraction to support from 1-5 generic type arguments.

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
            Validators(), ServerPerformInteraction);
    }

    private IList<IInteractionValidator<MouseDrop>> Validators()
    {
        return new List<IInteractionValidator<MouseDrop>>
        {
            IsDroppedObjectAtTargetPosition.IS,
            DoesDroppedObjectHaveComponent<PlayerMove>.DOES,
            CanApply.EVEN_IF_SOFT_CRIT,
            ComponentAtTargetMatrixPosition<PlayerMove>
                .NoneMatchingCriteria(pm => pm.IsRestrained),
            new FunctionValidator<MouseDrop>(AdditionalValidation)
        };
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

    private InteractionResult ServerPerformInteraction(MouseDrop drop)
    {
        // server-side interaction logic + inform clients

        return InteractionResult.SOMETHING_HAPPENED;
    }

   //from IInteractable
   //delegate the interface method implementations to coordinator
    public InteractionResult Interact(MouseDrop interaction)
    {
        return coordinator.ClientValidateAndRequest(interaction);
    }

    //from IInteractionProcessor
    public InteractionResult ServerProcessInteraction(MouseDrop interaction)
    {
        return coordinator.ServerValidateAndPerform(interaction);
    }
}
```
The flow for this approach is exactly the same as the first approach, but we are not limited by extending CoordinatedInteraction. 

The biggest advantage in flexibility of this approach is that you can use it on any existing component, since all you need to do is implement the indicated interfaces. In this example, we've added it on to our existing chair component. This CAN be useful to avoid having to create a new component for each new type of interaction, but you should be careful about putting too much logic into a single component. Unity components are designed to be re-usable, and combining lots of logic into a single component can hinder that. In general, I would suggest creating a new component and using approach 1 as a "sensible default". 

The cost of that flexibility is that we have to add extra code to delegate to the coordinator. This is done for us in CoordinatedInteraction. 

The IInteractable interface tells IF2 that our component can process a MouseDrop interaction, as such the implementation of the interface method will be invoked when this component is on an object involved in the interaction. The IInteractionProcessor interfaces tells IF2 that our component can perform the server-side portion of an interaction. In order to use the InteractionCoordinator we have to pass it a component which implements IInteractionProcessor. We could pass it a different gameobject's IInteractionprocessor component if we wanted to, but for most use cases it should make the most sense to implement this directly on this component.

With this approach, we can also modify the implementation of ServerProcessIneteraction and Interact so that we are not only limited to the functionality provided by the InteractionCoordinator.

## 3. Implement IInteractable - Most Flexible, Least Convenient
```csharp
public class BuckleInteractDIY : IInteractable<MouseDrop>, IInteractionProcessor<MouseDrop>
{
    //from IInteractable
    public InteractionResult Interact(MouseDrop interaction)
    {
        // We can still re-use validators if we want:
        var validationResult = CanApply.EVEN_IF_SOFT_CRIT.Validate(interaction, NetworkSide.CLIENT);

        // do whatever we want - but remember this is only invoked
        // on the client performing the interaction.
        // It's up to us to communicate with the server and
        // perform validation if needed.
        
        //if desired, we can send a RequestInteractMessage like so,
        //indicating this component will process the interaction on the server side
        // in ServerProcessInteraction:
        RequestInteractMessage.Send(interaction, this);
    }

    //from IInteractionProcessor, this will be invoked when the server gets the RequestInteractMessage
    public InteractionResult ServerProcessInteraction(MouseDrop interaction)
    {
        //validate and perform the update server side after it gets the RequestInteractMessage,
        //then inform all clients.
    }
}
```
With this approach, we implement IInteractable so that IF2 will invoke this component if the object it's on is involved in that type of interaction. However, it's up to us to handle all the rest of this interaction. If this was something that should only be client-side, for example, we could just implement the client-side logic here. But if there is any communication needed with the server we would have to implement that logic as well. 

We can re-use validators here as well as RequestInteractMessage / IInteractionProcessor as shown.

As with the previous approach, we can also implement this on any existing component if we want.
# Which Approach Is Best?
Each approach is a tradeoff between convenience and control. In general I would suggest approach 1. If approach 1 isn't meeting your needs, you can consider adding logic to IF2 to try to capture your specific use case, and follow approach 1's example for how to make it convenient to use. If you do decide to go with approach 2 or 3 be careful of code duplication - try to avoid duplication by building your logic into helper classes or adding functionality directly to IF2 to support your use case.

Be warned that any changes to the core of IF2 will be inspected closely...we want to ensure that interaction code is as clean as possible since interaction is such a foundational aspect of the game.

# Multiple Interaction Components
In the old system, you could only have one interaction component per object. In the new system, **you can have multiple components on an object which implement interaction logic, even for the same type of interaction**. This means you no longer should need to do things like extending PickupTrigger. You can define a PickupInteraction component and a separate component for that object's specific interaction logic.

# Precedence of Interaction Components
For something like a MouseDrop or HandApply interaction, there is the player performing it (performer) the object they are using or dropping (used object) and the object they are dropping on or clicking in the game world (target). 

In IF2, for a given type of interaction, interaction components will be searched for on the used object and the target, so you can put the implementation of the interaction on whichever one makes the most sense. 

At the time of writing, when any interaction component returns InteractionResult.SOMETHING_HAPPENED (which happens in CoordinatedInteraction if all validations succeed), no further interaction checks will be done. 

The order of checking is:
1. All components (in the order they appear in editor) are checked on the used object.
2. All components (in the order they appear in editor) are checked on the target object.

If more sophisticated coordination is needed between components (such as changing the order of precedence on an as-needed basis), we can expand IF2's interaction checking logic and modify InteractionResult so that this can be done.

# Migration
Migration will be done in a piecemeal fashion. Any time we discover usability, code duplication, or design issues with IF2, we will nip them at the bud before continuing to migrate. Please bring any design concerns to Discord. 

IF2 is designed such that it lives alongside the old interaction system. Once the old interaction system is complete, we should remove the last vestiges of the old code.

# Interaction Types
Currently only HandApply and MouseDrop interactions are implemented. Others will be implemented as needed. Here are the planned types of interactions:
* MouseDrop - Click and drag a MouseDraggable object in the game world and release it to drop.
* HandApply - click something in the game world. The item in the active hand (or empty hand) is applied to the thing that was clicked.
* HoldHandApply - like hand apply, but occurs at some interval while the mouse is being held down after being clicked in the game world. For things like shooting an automatic weapon, spraying fire extinguisher, etc...
* Activate - clicking an item in the active hand or pressing Z
* Combine - dragging and dropping an item from on UI slot to another, which may or may not be occupied
* DragApply - dragging and dropping an item from a UI slot to the game world, which I think always just drops it on the ground but I'm making it separate for now just in case not.

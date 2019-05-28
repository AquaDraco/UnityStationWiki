- [Overview](#overview)
  * [1. Generate Options On-Demand with IRightClickable](#1-generate-options-on-demand-with-irightclickable)
  * [2. Register Right Click Options Ahead of Time](#2-register-right-click-options-ahead-of-time)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


This page describes how the new right click menu works. The new system uses Scriptable Objects and allows for dynamic changes to an object's right click menu (allows changing what options are presented based on an object's state). It also allows overriding the text, sprite, and background color of the menu items.

# Overview
All possible right click options are defined as ScriptableObjects. These define the appearance of the right click option but do not define what happens when the option is selected. These are all in Resources/ScriptableObjects/Interaction/RightclickOptions.

To ensure consistent ordering, there is also a RightClickOptionOrder ScriptableObject which allows defining the display order of RightClickOptions. You can modify this in the Right click canvas.prefab's Rightclick Manager component.

RightclickManager (formerly Rightclick) lives on Right click canvas.prefab and manages all right click logic, similarly to what Rightclick was doing before.

Each object which is right-clickable now has a RightClickMenu component which holds the current menu items for that object (RightClickMenu.GetCurrentOptions). This may not exist on all objects that have right click options, so various approaches are used (discussed below) to ensure this component is added to the object if needed.

RightClickMenu's fields can be used to customize the appearance of the object's menu item. The RightClickOption can be used to customize the appearance of a particular option.

There are 2 ways for a component to define right click menu options on its object.

## 1. Generate Options On-Demand with IRightClickable
A component can implement IRightClickable to define which options should be shown based on its current state.
This is the generally recommended approach because it centralizes the right click option logic. The biggest limitation of this approach is that the options are always generated
on-demand when the right click occurs, which might cause some lag if the logic is particularly complicated. Usually
it is not, so this is the recommended approach.

To use this in a component:
1. Create a RightClickOption for your option if it doesn't already exist, and set up its appearance. You can do this in Unity by the Create > Interaction > Right click option menu item.
1. Add a field to cache the RightClickOption. Make it public so it is inspectable / editable. - `public RightClickOption someOption`
1. Add `[RequireComponent(typeof(RightClickMenu))]` to the component ensure that the RightClickMenu is always added
alongside your component. This doesn't effect existing prefabs...therefore
2. In Start() or Awake() (or whatever startup method), add `RightClickMenu.EnsureComponentExists(gameObject)`. This 
ensures that the component is added at runtime if it isn't present. Many objects do not have RightClickMenu since it
was created after they were originally created. You can omit this if you know for sure that RightClickMenu exists
on all prefabs that use your component.
3. Implement IRightClickable, returning a dictionary of the currently-available options based on the component's current
state.

You can have this on multiple components on a given object - all of their right click options will be added together.

Refer to the following example for PickUpTrigger:
```csharp
//Ensures the RightClickMenu is added when this component is added. 
//This doesn't affect existing prefabs though.
[RequireComponent(typeof(RightClickMenu))]
public class PickUpTrigger : InputTrigger, IRightClickable
{
  //Allows caching of the option and overriding the display of this option in editor.
  public RightClickOption pickUpOption;

  public void Start()
  {
    //This ensures the RightClickMenu component exists at runtime.
    RightClickMenu.EnsureComponentExists(gameObject);
  }

  //implementation of the interface method. This is invoked by RightClickMenu when
  //it's time to display menu options.
  public Dictionary<RightClickOption, Action> GenerateRightClickOptions()
  {
    //will return a dictionary mapping from the option to the Action to invoke 
    //when the option is selected
    var result = new Dictionary<RightClickOption, Action>();
    
    //Default to the specified option if pickUpOption is null.
    //this assignment allows the resource to be cached so the lookup doesn't need to be performed every time.
    pickUpOption = RightClickOption.DefaultIfNull("ScriptableObjects/Interaction/RightclickOptions/PickUp",
			pickUpOption);

    //if player can pick this up
    if (...) //various validations
    {
      //add the pick up option, call GUIInteract if it is selected.
      result.Add(pickUpOption, GUIInteract);
    }
    
    return result;
  }

  void GUIInteract()
  {
    //...performs the pickup logic
  }
}
```
## 2. Register Right Click Options Ahead of Time
Rather than generating options on-demand, you can instead directly register and unregister RightClickOptions on
RightClickMenu. This can be used for options which should always be present or which have expensive logic to
determine if they should be shown which you want to calculate ahead of time. In general, however, it is recommended to use
IRightClickable unless you have a good reason not to.

To use this approach:
1. Create a RightClickOption for your option if it doesn't already exist, and set up its appearance. You can do this in Unity by the Create > Interaction > Right click option menu item.
1. Add a field to cache the RightClickOption. Make it public so it is inspectable / editable. - `public RightClickOption someOption`
2. Now you can simply use RightClickMenu.AddRightClickOption and RightClickMenu.RemoveRightClickOption
to register/unregister options as they should be made available / unavailable. These are static methods which
also take care of creating RightClickMenu on your object if it doesn't already exist.

For example, here is how it is used in ItemAttributes for the Examine option:
```csharp
public class ItemAttributes : NetworkBehaviour
{
  //allows caching / overriding in editor.
  public RightClickOption examineOption;

  private void Awake()
  {
    //Register our right click options. 
    examineOption = RightClickMenu.AddRightClickOption("ScriptableObjects/Interaction/RightclickOptions/Examine",
        gameObject, OnExamine, examineOption);
  }

  void OnExamine()
  {
    //...examine logic goes here
  }
}
```
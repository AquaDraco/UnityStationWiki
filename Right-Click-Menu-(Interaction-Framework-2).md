This page describes how the new right click menu works. The new system uses Scriptable Objects and allows for dynamic changes to an object's right click menu (allows changing what options are presented based on an object's state). It also allows overriding the text, sprite, and background color of the menu items.

# Overview
All possible right click options are defined as ScriptableObjects. These define the appearance of the right click option but do not define what happens when the option is selected. These are all in Resources/ScriptableObjects/Interaction/RightclickOptions.

Each object which is right-clickable now has a RightClickMenu component which holds the current menu items for that object (RightClickMenu.GetCurrentOptions) as well as generator functions (discussed below). When an attempt is made to register a right click option, RightClickMenu is added dynamically to the object if it doesn't exist. Any other component on the object can register a new right click menu option by invoking AddRightClickOption and specifying the RightClickOption as well as the Action to perform when the option is clicked. For example, this shows how PushPull component registers the "Pull" action:
```csharp
class PushPull {
  //allows overriding the RightClickOption - this is assignable in editor.
  //but RightClickMenu.AddRightClickOption allows defaulting to a specific RightClickOption if this
  //is left null.
  public RightClickOption pullOption;
  
  //...snip...

  protected override void Awake() {
    base.Awake();

    //init our right click option.
    //If pullOption was null in Editor, it will default to the Pull option specified at the below path.
    //When the option is clicked, it will invoke TryPullThis.
    pullOption = RightClickMenu.AddRightClickOption("ScriptableObjects/Interaction/RightclickOptions/Pull", 
      gameObject, TryPullThis, pullOption);
  }

  void TryPullThis()
  {
    //...pull logic here...
  }
}
```

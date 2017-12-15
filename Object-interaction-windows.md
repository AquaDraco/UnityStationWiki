## Currently in SS13
Currently SS13 works with popop windows for every object-interaction-related screen.
Such as: Computers, SMES, PDA's and so on. These windows are made outside of the actual game screen and can be put aside for a moment if need be.

## Why not direct port it to unity
Unity is designed to keep all game content, in the actual game screen. This hinders interaction with you environment when using such devices, as you have less actual game-screen to monitor.

## The solution
To solve this problem, we should use the tab-system we already utilise for Alt-LMB. This means that every action that would popup a windows in SS13 would open a tab in unitystation.

This solution also puts more emphasis on the tabwindow and thus makes it less of a cheat to fully hide it. As you actually lose some game-functions along the way. 

## Expected implementation issues
The tabsystem currently hard codes a lot of things, dynamic opening and closing of tabs is more a hack than a feature at the moment. As can be seen in the alt-LMB related content.

To solve this, one should write some functions which can be called from, for example, a device script to easily add and remove the device tab from the UI

## Future expansion
In the future, a right-side tabwindow may not be enough or it may not fit all players. For that reason, tabs should eventually be moveable to, for example, a left-side tabwindow
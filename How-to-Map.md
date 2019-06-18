This page explains the details of how to do mapping within unitystation.

All mapping is done in Unity.

### Setting Up Your Workspace
You should have the following windows open. To open a window, go to the _Window_ tab at the top of Unity.
* Window > General > __Scene__
* Window > General > __Project__
* Window > General > __Hierarchy__
* Window > General > __Inspector__
* Window > 2D > __Tile Palette__

### Introduction to Scenes
Unity uses [scenes](https://docs.unity3d.com/Manual/CreatingScenes.html) to define environments and menus. Let's look at one.
* Open an existing scene: Project > Assets > Scenes > OutpostStation
* Make sure you have your Scene window focused
* Look around the Scene with the [Hand Tool](https://docs.unity3d.com/Manual/SceneViewNavigation.html)

Notice that in the Hierarchy window selecting _OutpostStation_ selects the entire station, but not other shuttles, etc.
You can save maps and areas as [prefabs](https://docs.unity3d.com/Manual/Prefabs.html) by dragging them from the Hierarchy tab into the Project tab.

### Tile Palette
* The Tile Palette tab lets you place walls, floors, doors, tables etc. into the Scene tab
* Make sure to select the right Active Tilemap in the Tile Palette when editing
* The new tiles and objects will be added to the right categories within the active tilemap automatically

### Edit a Palette
Tiles are added to the palettes as they are created. To create a new Tile follow this example:
- Choose a tile from /Tilemaps/Tiles/Objects
- Drag a tile to a palette

### Create a new map
* TODO: How do you create a new map from a station palette?
* TODO: How do you add your new map for map selection, other than this: [Testing custom maps in Multiplayer](https://github.com/unitystation/unitystation/wiki/Building-And-Testing#testing-custom-maps-in-multiplayer)?

## Object-Specific Mapping Info
This section describes the specifics of how various objects are mapped (anything that isn't obvious from within the editor).

### Wallmounts
Wallmounts always appear upright, but you need to indicate which side of the wall they are on (they should only be visible from one side of a wall). To do this, simply set the Directional component's Initial Direction to the direction it should be facing.

Enabling Gizmos for WallmountBehaviours will show helper arrows so that you wouldn't need to guess their direction:
![how-to-enable](https://user-images.githubusercontent.com/10403536/58015542-0f0e2800-7aeb-11e9-8729-5a06fb88072e.png)

## Chairs
Chairs can be rotated by setting their Directional component's Initial Direction.

## Pull Requests for Tile Palette Changes
Almost never would you need to actually PR a palette change, if you do please make sure __NOT__ to include anything other than the palette file and its .meta file.

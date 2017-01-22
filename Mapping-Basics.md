## Mapping Quick Guide

To access the mapping tool go to Window --> Map Editor

Activate the map editor by ticking the 'Map Editor Mode' box

From here you can adjust some options like using the mouse Left click to place tiles and to enable the tile placement preview. Alternatively you can use keyboard shortcuts:

- A = place tile
- D = delete selected tile

To create a new section to work on simply create a GameObject underneath the parent Map_BoxStation(DO_NOT_PREFAB) GameObject in the hierarchy and give it the name of your new section (i.e MedBay). Then on the Map Editor click on 'Refresh Data' and your new section will now appear in the sections roll out. The Map Editor will create the child GameObjects (walls, floors, doors, tables etc) automatically and place the tiles accordingly. 

Tiles are added to the map editor as they are needed. To create a new Tile follow this example:

### Example: Adding a new floor tile

- Choose a floor tile from Prefabs --> Floors and drag it into the scene
- With the tile highlighted in the Hierarchy go to GameObject --> Break Prefab Instance (in the menu)
- Rename the tile to what you want (i.e MedBayTile_RightBlue)
- Now click on the child GameObject named sprite and look in the inspector for the Sprite Renderer
- By clicking on the Sprite already in the Sprite Renderer it should bring you to the SpriteSheet with all the floor sprites (this only works if you have the Project Files window open when you click on the sprite)
- Find the sprite you are looking for and drag it into the Sprite Renderer
- When you are all done, drag your new Tile GameObject into Prefabs --> Floors in the project files window to make it a prefab
- Then go back to your Map Editor and click Refresh Data. Your new tile will now be ready to use
- Don't forget to clean up the Hierarchy by removing the GameObject after you have turned it into a prefab

### Pull Requests

When you have made all your changes, you must submit a pull request to the mapping branch where your PR will be inspected and then merged. Any questions please join ##unitystation on freenode

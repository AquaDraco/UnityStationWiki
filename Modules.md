**PlayerManager**

Sets up the local player.


**SoundManager**

Loads all sounds and plays them on demand.


**GameData**

Loads/saves game data locally, and sets other managers on scene load.


**SpriteManager**

Loads sounds and provides sprites on demand.


**GameManager**

Sets up match and manages occupations.


**UIManager**

Manages the HUD and tooltips.


**ObjectManager**

Handles the PoolManager and Cloth factory. Factories are only available to the server.

_Question: Is a new PoolManager instanced each round?_


**PoolManager**

Manages pooling. There are methods for both clients and server.

_Question: When pooling bullets, the server (through a Command) is using the clientpool function instead the server pool. Is this because the server is currently not dedicated?_


**EventManager**

Handles UI events. Works as a shortcut for UI events rather than adding logic.


**NetworkManager**

The custom NetworkManager. Handles connections, the lobby, player prefabs...


**CraftingManager**

Component storing all receips.


**EquipmentManager**

Doesn't seem to be doing nothing at all. Script contains Enums of equipment slots.


**DisplayManager**

Handles client resolutions and lighting.

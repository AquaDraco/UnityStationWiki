This is a complicated codebase and it can be difficult for developers to how to check something or get a reference to what they need. This is a cheat sheet of common functions to help you quickly find what you need so you don't waste a lot of time looking or reimplementing something that already exists. This is not intended to be a complete list of functions, but just the most commonly used ones. Please expand it or modify it as you see fit.

# Logging

To write to the debug log:
```
Logger.Log(string msg, Enum Category)
```


To set the log level for different categories, go to unitystation/UnityProject/Assets/Scripts/Debug/Logger.cs, and modify 

```
private static readonly Dictionary<Category, Level> LogOverrides = new Dictionary<Category, Level>{
		[Category.Unknown]  = Level.Info,
		[Category.Movement] = Level.Error,
		[Category.Health] = Level.Trace, 
```
to your liking.

# Player
Check if player has spawned
```
bool playerSpawned = (PlayerManager.LocalPlayer != null);
```

Check if local player is dead
```
PlayerManager.LocalPlayerScript.playerHealth.IsDead
```
Check if local player is in critical health
```
PlayerManager.LocalPlayerScript.playerHealth.IsCrit
```

functions related to items the player has equipped
```
PlayerManager.Equipment
```

Functions related to player taking actions with the server
```
PlayerManager.LocalPlayerScript.NetworkActions
```

# Networking
Check if this is the server, but only inside a NetworkBehaviour
```
if(isServer)
```

Check if this is the server

```
CustomNetworkManager.Instance._isServer == false
```




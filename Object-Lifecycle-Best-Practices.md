Object Lifecycle refers to how objects are created and destroyed.

All components in the game still follow the [normal Unity function flow as documented here](https://docs.unity3d.com/Manual/ExecutionOrder.html), however there are some special circumstances in Unitystation:

1. **Object Pooling** - For some objects, we use a technique called object pooling. Objects are created ahead of time and added to a pool, then simply moved into position in the game when an object of that type needs to be spawned. When a pooled object is despawned, it is simply returned to the pool. This avoids expensive object creation and destruction logic. It is particularly useful for things like bullets or casings, which spawn rapidly. But it creates some complications due to re-using objects which may be in different states from when they were originally despawned.
2. **Cloning** - We have the capability to clone objects, the intention being that the cloned object should start with the same state as the object it was cloned from.

**What lifecycle methods must your component implement?**
1. All components should put their init logic in the normal Unity methods (`Start`, `Awake`, etc...) regardless of the situation (if using SyncVars, also see [[SyncVar Best Practices for Easy Networking]].
1. If your component is going to be on pooled objects or needs special cloning logic, you should ALSO implement init logic using `IOnStageServer` / `IOnStageClient` / `IOffStageServer` so it can be safely reused and / or cloned. Components on non pooled objects can ignore these, but if there is any chance your component will be used on a pooled object you should still implement these.

**To test that your component is "pool safe", i.e. that it initializes itself properly even when being re-used:**
1. Do something to the object in the game that would modify your component's state.
2. Right click the object and choose the "Respawn" option. This will simulate putting it into the pool and taking it back out.
3. Validate that the respawned object behaves as if it was newly-spawned and initialized, and has nothing left over from its state before it was respawned.
4. If there is some leftover state or any errors with initialization, make sure you have properly implemented any necessary `IOnStageServer` / `IOnStageClient` / `IOffStageClient` hooks and validate the logic you have for any Unity methods such as Awake, Start, OnStartClient, OnStartServer.

**To test that your component clones properly, and preserves the state of the object it was cloned from:**
1. Do something to the object in the game that would modify your component's state.
2. Use the Dev Cloner tool to clone the object
3. Validate that the new clone has the same state as the object it was cloned from. It should not reset to its initial state.
4. If its state is not cloning, make sure you have properly implemented cloning logic in `IOnStageServer` / `IOnStageClient` (check `info.IsCloned` and `info.ClonedFrom`) and validate the logic you have for any Unity methods such as Awake, Start, OnStartClient, OnStartServer. 

## Examples

See `Integrity.GoingOnStageServer`. It implements cloning and normal pool-safe spawning.

## Lifecycle Flow Details

This section clarifies which lifecycle methods are called in different situations.

**When an object is _mapped_ in the scene and the scene loads**:
1. The object components follow the [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html) (Start, Awake, etc...).
2. IOnStageServer / IOnStageClient are NOT called!

**When a _non-pooled_ object will be spawned**:
1. The object components follow the [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html) (Start, Awake, etc...).
2. IOnStageServer / IOnStageClient are called on each component. But, typically non-pooled object components do not need to implement these hooks unless they have special cloning logic.

**When a _pooled_ object is going to be spawned and doesn't exist in the pool**:
1. The object is created and the components follow the [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html) (Start, Awake, etc...).
2. The new object is moved into the proper location on the scene and made visible / active
3. `IOnStageServer` and `IOnStageClient` hooks are called on each component.

**When a _pooled_ object is going to be spawned and an instance already exists in the pool**:
1. The pooled instance is moved into the proper location on the scene and made visible / active.
2. `IOnStageServer` and `IOnStageClient` hooks are called on each component.

**When _any_ object is going to be cloned**:
1. A clone is created from the object's prefab using the [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html) (`Start`, `Awake`, `OnStartClient`, `OnStartServer`, etc... will all be called as per usual logic).
2. The new object is moved into the proper location on the scene and made visible / active
3. `IOnStageServer` and `IOnStageClient` hooks are called on each component. The OnStageInfo contains a reference to the GameObject that the object was cloned from so any needed state can be copied.

**When a _non-pooled_ object is going to be despawned**:
1. `IOffStageServer` hooks are called on each component
2. Object is destroyed via normal Unity approach, each component follows [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html)

**When a _pooled_ object is going to be despawned**:
1. `IOffStageServer` hooks are called on each component
2. The object is moved back into the pool - back to HiddenPos, made invisible / inactive.
3. If the pool is full, the object is destroyed via normal Unity approach, each component follows [normal Unity function flow](https://docs.unity3d.com/Manual/ExecutionOrder.html)

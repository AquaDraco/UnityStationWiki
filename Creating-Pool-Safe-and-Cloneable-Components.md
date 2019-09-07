We use a technique called object pooling. Objects are created ahead of time and added to a pool, then simply moved into the right location in the game when an object of that type needs to be spawned. When a pooled object is despawned, it is simply returned to the pool. This avoids expensive object creation logic. It is particularly useful for things like bullets or casings, which spawn rapidly. But it creates some complications due to re-using objects which may be in different states from when they were originally despawned.

"Non-pooled" objects are those which bypass the pool. Only some of our objects are actually pooled - this is configurable. Typically we would want objects which are frequently created to be pooled. Non-pooled objects are simply created / destroyed using normal Unity methods and do not have problems with re-use.

"Mapped objects" (objects which are placed in the scene in the editor) do not spawn from the pool. They are just there when the scene loads in whatever state they are mapped in.

We also discuss how to ensure objects are properly cloneable.

**To test that your component is "pool safe", i.e. that it initializes itself properly even when being re-used:**
1. If your component is only used on non-pooled objects, it is inherently pool safe, otherwise...
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

## Lifecycle Details

All components still follow the [normal Event function flow as documented here](https://docs.unity3d.com/Manual/ExecutionOrder.html). However, objects which are being re-used from the pool will bypass the normal Unity init events (particularly `Start`, `Awake`, `OnStartServer`, `OnStartClient`), as those were already called when it was first created.

Objects in the pool are simply placed at `TransformState.HiddenPos` and have `ObjectBehavior.VisibleState = false` (they are basically invisible and inactive).

Note that "mapped objects" (objects which are placed in the scene in the editor) do not spawn from the pool. They are just there when the scene loads in whatever state they are mapped in.

**When an object is mapped and the scene is loading**:
1. The object follows the normal Unity event flow - IOnStageServer / IOnStageClient are NOT called!

**When an object is going to be spawned and doesn't exist in the pool**:
1. If there is no instance of the object in the pool, it is created using normal Unity methods (`Start`, `Awake`, `OnStartClient`, `OnStartServer`, etc... will all be called as per usual logic).
2. The new object is moved into the proper location on the scene and made visible / active, then `IOnStageServer` and `IOnStageClient` hooks are called on each component.

**When an object is going to be spawned and an instance already exists in the pool**:
1. The pooled instance is moved into the proper location on the scene and made visible / active, then `IOnStageServer` and `IOnStageClient` hooks are called on each component.

**When an object is going to be cloned**:
1. A clone is created from the object's prefab using normal Unity methods (`Start`, `Awake`, `OnStartClient`, `OnStartServer`, etc... will all be called as per usual logic).
2. The new object is moved into the proper location on the scene and made visible / active, then `IOnStageServer` and `IOnStageClient` hooks are called on each component. The OnStageInfo contains a reference to the GameObject that the object was cloned from so any needed state can be copied.

**When an object is going to be despawned**:
1. `IOffStageServer` hooks are called on each component
2. The object is moved back into the pool - back to HiddenPos, made invisible / inactive. If an object is not to be pooled or pool is full, it is simply destroyed

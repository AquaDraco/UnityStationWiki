The [SyncVar attribute](https://docs.unity3d.com/Manual/UNetStateSync.html) can save a lot of time and code for syncing data from server to all clients (because it avoids having to implement a custom Net Message), but many of us have struggled with understanding its peculiarities (of which it has many) especially as it pertains to how Unitystation is put together. Having used it quite a bit now, I've developed a set of simple "best practices" for how to use them which should hopefully save future developers from struggling with it.

# Proper SyncVar Usage
These are things you should almost ALWAYS do if using syncvar.
TODO: Code examples
1. Add SyncVar to the field you want to sync. The field should almost ALWAYS be private. NEVER allow the field to be directly modified by other components.
    * If the field needs to be viewable externally, create a public readonly accessor.
    * If other components need to know when the syncvar changes, create a UnityEvent they can subscribe to which you invoke in your hook method.
2. Define a private hook method named "Sync(name of field)". The first line of the hook should update the field based on the new value. Do NOT make a protected or public hook method.
    * There don't seem to be many cases where you would want a syncvar without a hook, because that implies the 
   client would need to be polling the SyncVar field on a regular basis.
3. Override OnStartClient and invoke the hook, passing it the current value of the field. If you are extending a component, make sure to call base.OnStartClient(). This ensures the SyncVar hook is called based on the initial value of the field that the server sends.
4. Define an Awake method (if not already defined) and invoke the hook. This ensures the hook is called on the server side for the initial value of the field as well as ensuring the hook is called when the component is disabled / enabled, which can happen due to VisibleBehavior and object pooling logic.
5. The ONLY place you are allowed to change the value of the syncvar field is via the syncvar hook and only on the server! Never change the value on the client side, and never modify the field directly. If you are on the server and you want to change the field value, call the hook method and pass it the new value. This ensures that the hook logic will always be fired on both client and server side.

# Surprising SyncVar Facts
Here are some of the things you might not know about how they work which explains the reasoning for the above practices. Not 100% certain on all of these but they might as well be true.
1. SyncVar hooks don't fire on the server when the server changes the field. They only fire on the client when the client receives the new value. Thus the server player will not have the hook called. This is the reason to always change the syncvar by way of the hook method on the server.
2. SyncVar hooks don't fire when client connects and receives the initial value of the field. This is why you should call the syncvar hook manually in OnStartClient.
3. SyncVar only works properly on specific types of fields ([see the docs](https://docs.unity3d.com/Manual/UNetStateSync.html)), but Unity does not seem to complain very noisily if you use it on an invalid type - the game still builds. If your syncvar doesn't appear to be working, chances are you are using it on an invalid type.
4. If you don't update the field's value in the syncvar hook using the new value passed to the hook method, the client-side value won't be updated. This is why you should always update the field in the first line of the hook method.
5. It takes a varying amount of time for the server to send the updated SyncVar field to the client, so make sure the code doesn't depend on the syncvar change arriving at a particular time or in a particular order in relation to other network messages or syncvar updates. If you change 2 syncvars on the server and also send a net message, they could arrive on the client in any order.
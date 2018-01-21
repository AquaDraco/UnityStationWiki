So you want to get something networked and want it fast?
There you go:

## Syncvar

## Client message
This is a message the client sends that get executed by the server.

1. Create a client message script:
/Assets/Script/messages/Client/PlaceholderMessage.cs

2. Add this base content:
https://unitystation.org/wp-content/uploads/PlaceholderMessage.cs

3. add the code you want the server to execute under process()


4. Add the message type to  Messagetypes.cs like so:
![](https://unitystation.org/wp-content/uploads/ScreenshotMessageTypes.png)

5. Call the client message like this:
![](https://unitystation.org/wp-content/uploads/Screenshotcallmessage.png)

## Client Interact Message
If you just want your interact to be networked, there is a simple solution.
You can just call the client message Interact from any InputTrigger script:

`
UI_ItemSlot slot = UIManager.Hands.CurrentSlot;
InteractMessage.Send(gameObject, position, slot.eventName);
`

This runs your whole interaction script as server.

## Server message to all


## Server message to one


## Command

## ClientRPC


### Disclaimer: this is a half-assed and outdated page. Ask the guys on Discord for up-to-date networking techniques.

So you want to get something networked and want it fast?
There you go:

## SyncVar

## Custom Client message
This is a custom message the client sends that get executed by the server.

1. Create a client message script:
/Assets/Script/messages/Client/PlaceholderMessage.cs

2. Add this base content:
[SimpleInteractMessage](https://github.com/unitystation/unitystation/blob/acbc90d217cf0aa9c8a914f17a838555399396b7/UnityProject/Assets/Scripts/Messages/Client/SimpleInteractMessage.cs)

3. add the code you want the server to execute under process()


4. Add the message type to  Messagetypes.cs like so:
![](https://unitystation.org/wp-content/uploads/ScreenshotMessageTypes.png)

5. Call the client message like this:
![](https://unitystation.org/wp-content/uploads/Screenshotcallmessage.png)

## Client Interact Message
See [Interaction Framework V2](https://github.com/unitystation/unitystation/wiki/Interaction-Framework-V2---How-to-Cleanly-Implement-Interactions)

## Server message to all


## Server message to one


## Command

## ClientRPC


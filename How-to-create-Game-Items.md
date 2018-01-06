## THIS TUT IS OBSOLETE BECAUSE OF UNICLOTH AND TMS - Leaving for reference

***

*Please note this tutorial is written for beginners and goes into detail about basic unity functions

### Creating items from scratch:

1. First create a new GameObject in the Hierarchy and rename it:

![](https://cdn.discordapp.com/attachments/305964906359029761/305964928802750464/unknown.png)

2. Right click the new object and choose 'Create Empty' to create a child GameObject:

![](https://cdn.discordapp.com/attachments/305964906359029761/305965894901694465/unknown.png)

3. Rename this GameObject and call it sprite (putting sprites on child objects will help with setting up animations later on):

![](https://cdn.discordapp.com/attachments/305964906359029761/305966155212914698/unknown.png)

4. Highlight the sprite GameObject and in the Inspector add a new SpriteRenderer component:

![](https://cdn.discordapp.com/attachments/305964906359029761/305966892932136961/unknown.png)

5. In the Project files, search for the sprite of your item (Usually found in /textures/obj/) and drag the sprite into the sprite slot on the SpriteRenderer:

![](https://cdn.discordapp.com/attachments/305964906359029761/305967493191696384/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305969174067281921/unknown.png)

6. Now select the Sorting Layer rollout in the SpriteRenderer and choose items. Sorting layers affect the order in which a sprite is rendered. Lower layers will be hidden by sprites on higher layers. The 'Order in Layer' parameter is used to fine tune the sort order:

![](https://cdn.discordapp.com/attachments/305964906359029761/305968918097559554/unknown.png)

7. In the Hierarchy, choose the root object of your item (the parent of the sprite object you were working on) and in the Inspector, add a new component called 'Edit Mode Control':

![](https://cdn.discordapp.com/attachments/305964906359029761/305967967311495170/unknown.png)

8. The Edit Mode Control component handles the Snap To Grid behavior and it also sets the position of the transform on the z axis (The depth setting). Remember that setting negative values (i.e. -0.2) means you are moving the item closer to the camera. On the component, toggle the 'Use In Game' switch so that it is true and set the Depth to -0.2 (so later on the pick up trigger will not be hidden by tables etc):

![](https://cdn.discordapp.com/attachments/305964906359029761/305985268798849025/unknown.png)

9. Click on the Add Component button and search for 'Item Attributes'. Fill in the item name and make sure the Sprite Type is set to 'Items' (If you were creating clothes you would change this to clothes etc). Next we will find the inhand sprites and fill out the In Hand Reference fields:

![](https://cdn.discordapp.com/attachments/305964906359029761/305969768857468929/unknown.png)

10. The easiest way to find the InHand sprite sheets is to search for 'InHands' in the project window. You can also navigate to 'Assets > textures > Resources > mobs > inhands' to find them:

![](https://cdn.discordapp.com/attachments/305964906359029761/305970928108568577/unknown.png)

11. Find the items_lefthand sheet and search through the sprites to find the inhand sprite that corresponds with your item. If you were creating clothes or guns you would use the clothing or guns sheets:

![](https://cdn.discordapp.com/attachments/305964906359029761/305971285899345920/unknown.png)

12. For the cigarettes we found the inhand sprite at position 193. Most of the time the numbers will match up for both righthand and lefthand sheets (i.e. 193 for left and 193 for right) but sometimes they do not, so check the righthand sheet to make sure. In this case the first sprite for the righthand was off by 1 (194).

![](https://cdn.discordapp.com/attachments/305964906359029761/305973668511744010/unknown.png)

13. Enter the values in ItemAttributes on your item:

![](https://cdn.discordapp.com/attachments/305964906359029761/305981618038898699/unknown.png)

14. Next, click the Add Component button and search for 'Pick Up Trigger'. This adds the pick up item behavior to the object:

![](https://cdn.discordapp.com/attachments/305964906359029761/305993940706918401/unknown.png)

^ as you can see, it also added the Network Identity component. This allows the item to be tracked over the network

15. For the Pick Up Trigger to work we need a to add a BoxCollider2D component. Then we set the collider to act as a trigger by selecting the isTrigger toggle:

![](https://cdn.discordapp.com/attachments/305964906359029761/305994603809603589/unknown.png)

16. Adjust the bounds of the trigger by selecting 'Edit Collider' and use the handles to move the collider so it is snug around the item sprite:

![](https://cdn.discordapp.com/attachments/305964906359029761/305977569981759488/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305977766648610816/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305977889340260382/unknown.png)

17. So the server can sync up the transform position of the object across all clients, we need to add a NetworkTransform component:

![](https://cdn.discordapp.com/attachments/305964906359029761/305981873149181952/unknown.png)

18. Now it is time to turn the item into a prefab. First search the Project files and find the Prefabs > Items folder. Look to see if a folder already exists for your item. If not, then create one:

![](https://cdn.discordapp.com/attachments/305964906359029761/305974751476580352/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305974940044361728/unknown.png)

19. Now drag your item from the Hierarchy into the folder:

![](https://cdn.discordapp.com/attachments/305964906359029761/305975449496977408/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305975576798298113/unknown.png)

20. For the item to work across the network we need to add it to the 'Registered Spawnable Prefabs' list on the NetworkManager. Find the NetworkManager in the hierarchy underneath the Managers object:

![](https://cdn.discordapp.com/attachments/305964906359029761/305982429309698068/unknown.png)

21. Now add a new entry and drag your prefab from the Project Files into the slot:

![](https://cdn.discordapp.com/attachments/305964906359029761/305982726882983937/unknown.png)

![](https://cdn.discordapp.com/attachments/305964906359029761/305983171240001546/unknown.png)

That's it!

Remember if you want to make changes to your prefab to always apply them when you are done. The Prefab buttons are located at the top of the Inspector:

![](https://cdn.discordapp.com/attachments/305964906359029761/305982119455227904/unknown.png)

Another thing to note is that uNet prefers items to be spawned on server start (I.E NetworkServer.Spawn(gameObject) from a script on the server). If you were to connect as a client with the item as part of the scene then you will notice some weird behavior (wrong transform position and a missing ID exception on the connecting client). It is fine to ignore this step and leave the item in the scene on game start if you are just testing the functionality of the item and not planning on testing it with two clients.






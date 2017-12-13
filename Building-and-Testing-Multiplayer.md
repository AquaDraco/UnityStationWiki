Before reading, make sure you've read the basics article.

To test network interactions, you should set up a realistic environment. 
That means two instances: Client and Host (Server that's client, too).

You can run one instance from editor by pressing play button, 
but for another one you'll have to make a **standalone build**.

## Initial setup: Build settings
To make a proper standalone build, you should set up build settings (only once).

* Go to `File > Build Settings`:

![](https://image.prntscr.com/image/RUTrofZFQzyxMhQ9jMxGyA.png)

* Make sure that **current scene is the only one ticked**:

![](https://image.prntscr.com/image/cm3rlu2ySwuSH6iXWZ_yYA.png)

(If you don't see your current scene in the list, press `Add Open Scenes` button)


* Mark `Development Build` and `Script Debugging` checkboxes, that will show all Build's logs in Editor's console:
![](https://image.prntscr.com/image/Lh-2leBxSw6AHV_Xck8OdA.png)

* Press `Build And Run`. 

![](https://image.prntscr.com/image/riIkJGY2Re6vWGzZoDMfpA.png)

First time you'll get a folder selection dialog where the build files will be stored. 
Create a separate folder outside of unitystation's project hierarchy. 
Then building will start, Unity will freeze until it's finished. 

![](https://image.prntscr.com/image/_VDEEGXtTH6lln4GtmQGFQ.png)

When it's ready a game window will pop up. Now you can enjoy smooth framerates and drag window around, also launch several instances if required (that doesn't work for OSX).

Notice how Build's logs appear in Editor's console:

![](https://image.prntscr.com/image/UN0o10vPTZeW6yQpqaMgFw.png)

Next time you'll just have to press `Ctrl+B`/`Cmd+B` to Build And Run.

Important note:
> When you change stuff/recompile you'll need to **rebuild** to make Editor compatible with the Build (unless you changed client-ONLY stuff, then you can try, but it's not guaranteed).


## Set up ping simulation
To simulate network latency you need to select `Network Manager` in Hierarchy tab:

![](https://image.prntscr.com/image/ErpN3x62SfK3bcTzsjE5Zw.png)

Then find `Custom Network Manager` script in Inspector tab and tick `Use network simulation`:

![](https://image.prntscr.com/image/9lknAZmmR021gW8pCeNhsQ.png)

Set up desirable ping (200 is recommended) and save (so that you don't lose that setting next time Unity crashes)!

![](https://image.prntscr.com/image/QGX1GsbWTxqxOlEOPKs3wg.png)

Note that actual ping will be more than expected, `200 ping` setting usually makes it 250-750

## Start up
In general it's usually like this: Build And Run, then start up game in Editor when it finishes building.
Doesn't matter much who should host and who should join, but keep in mind that Build performs better.

![](https://image.prntscr.com/image/e_3gMpjMQz_cu801A8fHxg.png)

Happy spess testing!
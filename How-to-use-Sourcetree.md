## Intro
Starting off with this guide I expect you to already have installed sourcetree, logged into your git account with sourcetree and forked the unitystation repository. This guide will explain how you setup sourcetree to be best suited to work on unitystation and explains some less clear features of sourcetree. 

This guide is meant as a quick start and may or may not follow all best-practices, when there is an easier way for new developers to do things. It also may not include all features (yet) because things take time, but it will at the very least enable you to start off.

## Setting up the repo
Firstoff on your own fork, click "Clone or Download":
![](https://cdn.discordapp.com/attachments/290339969879375881/400562872910086144/Screen_Shot_2018-01-10_at_09.07.05.png)

Then select "Open in desktop" and sourcetree should open:
![](https://cdn.discordapp.com/attachments/290339969879375881/400562872490393601/Screen_Shot_2018-01-10_at_09.07.11.png)

In said window, chose the name and location to checkout the repo in. I prefer /User/GIT/ as a root, but your preference may vary. Always make sure to checkout the "Develop" branch in your first checkout:
![](https://cdn.discordapp.com/attachments/290339969879375881/400562869910896651/Screen_Shot_2018-01-10_at_09.07.43.png)

When you click "Clone" it clones the repo into said folder, this may take a while. Don't worry to let it run for half an hour, depending on connection speed.

*Post Install*
When the clone is done, the following window opens:

First you should check if everything is setup right. Go to preferences (in OSX, Sourcetree->Preferences.) and check under general if (atleast) your email adres is the same as in github. Change if it's not:

Under "Accounts" make sure your Github accounts is added:

last, but not least, under "advanced" make sure "allow force push" is checked:

In the middle you'll see the branches sourcetree know about them and their relations to eachother.
To the left under "branches" you'll see you local branches.
Under "remotes" you will find all branches on github itself, currently thats only "Origin" aka your own fork. To be able to checkout the upstream repo unitystation/unitystation, it need to be added to the remotes.

First go to the unitystation main-repo and copy the URL under "Clone or download":

Then in sourcetree, right click "Remotes" and select "add new remote".

In the window that pops up, enter the name for the new remote (I prefer Upstream) and paste the link you copied earlier under URL:


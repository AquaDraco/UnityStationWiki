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
![](https://cdn.discordapp.com/attachments/290339969879375881/400566166013214722/Screen_Shot_2018-01-10_at_09.21.47.png)

First you should check if everything is setup right. Go to preferences (in OSX, Sourcetree->Preferences.) and check under general if (atleast) your email adres is the same as in github. Change if it's not:
![](https://media.discordapp.net/attachments/290339969879375881/400562879700402187/Screen_Shot_2018-01-10_at_09.04.47.png?width=1030&height=958)

Under "Accounts" make sure your Github accounts is added:
![](https://media.discordapp.net/attachments/290339969879375881/400562881264877568/Screen_Shot_2018-01-10_at_09.04.39.png)

last, but not least, under "advanced" make sure "allow force push" is checked:
![](https://cdn.discordapp.com/attachments/290339969879375881/400566158178123816/Screen_Shot_2018-01-10_at_09.25.49.png)

Lets go back to the previous screen, shall we? In the middle you'll see the branches sourcetree know about them and their relations to eachother.
To the left under "branches" you'll see you local branches.
Under "remotes" you will find all branches on github itself, currently thats only "Origin" aka your own fork. To be able to checkout the upstream repo unitystation/unitystation, it need to be added to the remotes.

First go to the unitystation main-repo and copy the URL under "Clone or download":
![](https://cdn.discordapp.com/attachments/290339969879375881/400566164293419009/Screen_Shot_2018-01-10_at_09.23.27.png)

Then in sourcetree, right click "Remotes" and select "new remote":
![](https://cdn.discordapp.com/attachments/290339969879375881/400566171105230861/Screen_Shot_2018-01-10_at_09.21.36.png)

In the window that pops up, enter the name for the new remote (I prefer Upstream) and paste the link you copied earlier under URL:
![](https://cdn.discordapp.com/attachments/290339969879375881/400566160564944918/Screen_Shot_2018-01-10_at_09.23.35.png)

When you "okey"-ed both pop-up window away, you'll see your new remote on the left side of the screen, beneat origin:
![](https://cdn.discordapp.com/attachments/290339969879375881/400568093895819264/Screen_Shot_2018-01-10_at_09.34.25.png)

lets make sure sourcetree knows all branches on the up upstream repo and click-fat "Fetch" button on the top left. Click "Ok" in the screen that pops up:
![](https://cdn.discordapp.com/attachments/290339969879375881/400568094369775658/Screen_Shot_2018-01-10_at_09.32.02.png)

Now you are ready to use Sourcetree for normal issues.

## Setting up Sourcetree to checkout PR's
Sometimes a you want to test changes in a PR yourself, for example when reviewing. This guide explains how you can PR's to sourcetree so you can quick and easy check them out without using the commandline.

Firstoff, this guide is based on a system where you added the Upstream repository (named "Upstream") as an extra remote in GIT beside your fork. If you named it differently, please change the name accordingly. if you didn't add the remote, don't continue as it wont work anyway. Also worth to note: You cant directly push to the PR, even if you have write access to the repository.

*Git config*
Locate the section for your github remote in the `.git/config` file. It looks like this:

```
[remote "Upstream"]
	url = https://github.com/unitystation/unitystation.git
	fetch = +refs/heads/*:refs/remotes/Upstream/*
```

Now add the line `fetch = +refs/pull/*/head:refs/remotes/Upstream/pr/*` to this section. It ends up looking like this:

```
[remote "Upstream"]
	url = https://github.com/unitystation/unitystation.git
	fetch = +refs/heads/*:refs/remotes/Upstream/*
	fetch = +refs/pull/*/head:refs/remotes/Upstream/pr/*
```

Now fetch go to Sourcetree and use fetch to get PR branches filled:



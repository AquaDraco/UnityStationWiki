## Intro
Someones a reviewer may request changes or may want to make the changes himself. His guide explains how you can edit your own PR and/or those of others.

## Edit your own PR
To edit your own PR, you can just push new changes to the branch on your own REPO, those changes will be added to the PR automatically. 

## Edit someone else's PR
To edit the PR of someone else, you need writeaccess to the repository.
Firstoff, this guide is based on a system where you added the Main repository (named "Main") as an extra remote in GIT beside your fork.

Writeacces is only granted to moderators, administrators and people with special need to have it, like external contracts.

### Setup Easy checkout of PR's
Locate the section for your github remote in the `.git/config` file. It looks like this:

```
[remote "Main"]
	url = https://github.com/unitystation/unitystation.git
	fetch = +refs/heads/*:refs/remotes/Main/*
```

Now add the line `fetch = +refs/pull/*/head:refs/remotes/origin/pr/*` to this section. Obviously, change the github url to match your project's URL. It ends up looking like this:

```
[remote "Main"]
	url = https://github.com/unitystation/unitystation.git
	fetch = +refs/heads/*:refs/remotes/Main/*
	fetch = +refs/pull/*/head:refs/remotes/Main/pr/*
```

Now fetch all the pull requests:

```
$ git fetch origin
From github.com:unitystation/unitystation
 * [new ref]         refs/pull/1000/head -> origin/pr/1000
 * [new ref]         refs/pull/1002/head -> origin/pr/1002
 * [new ref]         refs/pull/1004/head -> origin/pr/1004
 * [new ref]         refs/pull/1009/head -> origin/pr/1009
...
```

To check out a particular pull request:

```
$ git checkout pr/999
Branch pr/999 set up to track remote branch pr/999 from origin.
Switched to a new branch 'pr/999'
```

### Pushing your changes to the PR
To push your changes directly into the PR, use the following command:
git push https://github.com/_USER_/unitystation.git  pr/_PR-NR_:_REMOTE-BRANCH_

Where _USER_ is the user who send in the PR.
_PR-NR_ is the PR number of the PR
and _REMOTE-BRANCH_ is the branch from the users fork the user send the PR from.
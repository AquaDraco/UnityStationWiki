Updating the actual active website can only be done by someone with github write access. Which shouldn't mind because you shouldn't be trying to anyway, if you do not have said access.

To push the changes from /Website/httpdocs to the gh-pages branch in which the active website reside, you just have to subtree push with commandline git.

CD to your git root directory (not the Website nor the UnityProject folder, the one before those) and run the following command:
git subtree push --prefix Website/httpdocs _InsertNameOfRemote_ gh-pages

Be sure to replace _InsertNameOfRemote_ with your local alias for the main repository remote.
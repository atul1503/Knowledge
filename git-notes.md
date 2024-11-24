## If `git push` has failed saying `failed to push`

If it happens because the remote branch is ahead than the local branch then do:
```
git pull --rebase
git push
```

`git pull --rebase` will put your local commit on top of all the commits from remote branch. So that when you push after that, your commits will appear on top of other commits which were pushed to remote by others earlier.

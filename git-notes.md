## If `git push` has failed saying `failed to push`

If it happens because the remote branch is ahead than the local branch then do:
```
git pull --rebase
git push
```

`git pull --rebase` will put your local commit on top of all the commits from remote branch. So that when you push after that, your commits will appear on top of other commits which were pushed to remote by others earlier.


## If you want to create version
This is a very simple but not the industry standard to determine versions of code. 

To calculate version ,first use this command to determine how many lines have changed in your git repo in the repo's history:
```
git log --pretty=tformat: --numstat | awk '{added+=$1; deleted+=$2} END {print added + deleted}'
```
It will give back a number. Let this number be `lines`

Version should have 3 parts:

Here, all divides (`/`) are integer divides, that is you should ignore the decimal part of normal divide answer.

1. First part is calculated as:
   ```
   lines / 1000
   ```
   That is, lines integer divided by 1000. This should give you an integer. This integer is the first part.

2. Second part is calculated as:

   ```
   (lines % 1000) / 100
   ```
   That is, lines remainder divided by 1000 and then integer divide that integer by 100.

3. Third part is calucalted as:
   ```
   (lines % 100)  
   ```

That version will look like this:
```
FirstPart.SecondPart.ThirdPart
```

For eg,
For lines=`3233`
Version will be:
`3.2.33`

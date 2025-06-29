Notes:

1. **Git is like a time machine for your code.** It tracks every change you make and lets you go back to any previous version. Think of it as saving checkpoints in a video game.

2. **When you mess up and need to undo things:**
   ```bash
   # Undo the last commit but keep your changes
   git reset --soft HEAD~1
   
   # Undo the last commit and throw away changes (be careful!)
   git reset --hard HEAD~1
   
   # Undo changes to a specific file (before committing)
   git checkout -- filename.txt
   
   # Undo all uncommitted changes (nuclear option)
   git reset --hard HEAD
   ```

3. **When `git push` fails saying "failed to push":**
   ```bash
   # Someone else pushed changes before you - rebase puts your commits on top
   git pull --rebase    # Get their changes and put yours on top
   git push             # Now push your changes
   
   # Alternative: merge instead of rebase (creates a merge commit)
   git pull             # This does a merge by default
   git push
   ```

4. **Working with branches (the right way to develop features):**
   ```bash
   # Create and switch to a new branch
   git checkout -b feature-name
   
   # Switch between existing branches
   git checkout main
   git checkout feature-name
   
   # See all branches
   git branch -a        # -a shows remote branches too
   
   # Delete a branch (after merging)
   git branch -d feature-name
   
   # Force delete a branch (even if not merged)
   git branch -D feature-name
   ```

5. **When you want to save work but aren't ready to commit:**
   ```bash
   # Stash your changes (like putting them in a temporary drawer)
   git stash
   
   # Get your stashed changes back
   git stash pop
   
   # See what you have stashed
   git stash list
   
   # Stash with a message so you remember what it is
   git stash save "work in progress on login feature"
   ```

6. **Fixing commit messages and adding forgotten files:**
   ```bash
   # Fix the last commit message
   git commit --amend -m "New commit message"
   
   # Add forgotten files to the last commit
   git add forgotten-file.txt
   git commit --amend --no-edit    # --no-edit keeps the same message
   
   # Change author of last commit
   git commit --amend --author="Your Name <your.email@example.com>"
   ```

7. **When you committed to the wrong branch:**
   ```bash
   # Move the last commit to a different branch
   git log --oneline -5             # Find the commit hash
   git reset --hard HEAD~1          # Remove commit from current branch
   git checkout correct-branch      # Switch to the right branch
   git cherry-pick <commit-hash>    # Apply the commit here
   ```

8. **Dealing with merge conflicts (when Git can't figure out what to keep):**
   ```bash
   # When you see conflict markers in files like this:
   # <<<<<<< HEAD
   # Your changes
   # =======
   # Their changes
   # >>>>>>> branch-name
   
   # Edit the file to keep what you want, remove the markers
   # Then:
   git add conflicted-file.txt      # Mark as resolved
   git commit                       # Complete the merge
   
   # Or abort the merge if you want to start over
   git merge --abort
   ```

9. **Checking what changed and when:**
   ```bash
   # See what files changed
   git status
   
   # See exactly what changed in files
   git diff                         # Uncommitted changes
   git diff --staged               # Changes that are staged for commit
   git diff HEAD~1                 # Changes since last commit
   
   # See commit history
   git log --oneline               # Compact view
   git log --graph --oneline       # Visual tree view
   git log -p                      # Show actual changes in each commit
   
   # See who changed what line in a file
   git blame filename.txt
   ```

10. **Working with remote repositories:**
    ```bash
    # Add a remote repository
    git remote add origin https://github.com/user/repo.git
    
    # See all remotes
    git remote -v
    
    # Push a new branch to remote
    git push -u origin branch-name    # -u sets up tracking
    
    # Get latest changes from remote
    git fetch                         # Download but don't merge
    git pull                          # Download and merge
    
    # Push all branches
    git push --all origin
    ```

11. **When you need to work on someone else's pull request:**
    ```bash
    # Add their fork as a remote
    git remote add contributor https://github.com/contributor/repo.git
    
    # Get their branches
    git fetch contributor
    
    # Check out their branch
    git checkout -b their-feature contributor/their-feature
    
    # Make changes and push to your fork
    git push origin their-feature
    ```

12. **Cleaning up your repository:**
    ```bash
    # Remove files that are ignored but still tracked
    git rm --cached filename.txt
    
    # Clean up untracked files (be careful!)
    git clean -n                     # Preview what will be deleted
    git clean -f                     # Actually delete untracked files
    git clean -fd                    # Delete untracked files and directories
    
    # Remove remote branches that no longer exist
    git remote prune origin
    ```

13. **Creating and applying patches (for sharing changes without pushing):**
    ```bash
    # Create a patch file from last commit
    git format-patch -1 HEAD
    
    # Create patch from multiple commits
    git format-patch HEAD~3
    
    # Apply a patch
    git apply patch-file.patch
    
    # Apply patch as a commit
    git am patch-file.patch
    ```

14. **Searching through your code history:**
    ```bash
    # Find commits that added or removed specific text
    git log -S "function_name"
    
    # Search commit messages
    git log --grep="bug fix"
    
    # Find when a file was deleted
    git log --follow -- filename.txt
    
    # See changes in a specific commit
    git show <commit-hash>
    ```

15. **Tagging releases (marking important versions):**
    ```bash
    # Create a lightweight tag
    git tag v1.0.0
    
    # Create an annotated tag with message
    git tag -a v1.0.0 -m "Release version 1.0.0"
    
    # Push tags to remote
    git push origin --tags
    
    # List all tags
    git tag -l
    
    # Delete a tag
    git tag -d v1.0.0
    git push origin :refs/tags/v1.0.0    # Delete from remote
    ```

16. **When you accidentally committed sensitive information:**
    ```bash
    # Remove file from last commit
    git rm --cached sensitive-file.txt
    git commit --amend --no-edit
    
    # Remove file from entire history (nuclear option)
    git filter-branch --force --index-filter \
      'git rm --cached --ignore-unmatch sensitive-file.txt' \
      --prune-empty --tag-name-filter cat -- --all
    
    # Force push to update remote (dangerous!)
    git push origin --force --all
    ```

17. **Useful Git configurations to set up once:**
    ```bash
    # Set your identity
    git config --global user.name "Your Name"
    git config --global user.email "your.email@example.com"
    
    # Better diff and merge tools
    git config --global merge.tool vimdiff
    git config --global diff.tool vimdiff
    
    # Colorful output
    git config --global color.ui auto
    
    # Useful aliases
    git config --global alias.st status
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --global alias.unstage 'reset HEAD --'
    git config --global alias.last 'log -1 HEAD'
    git config --global alias.visual '!gitk'
    ```

18. **Common workflows for different scenarios:**

    **Feature development workflow:**
    ```bash
    git checkout main
    git pull origin main
    git checkout -b feature/new-login
    # ... make changes ...
    git add .
    git commit -m "Add new login functionality"
    git push -u origin feature/new-login
    # ... create pull request in GitHub/GitLab ...
    ```

    **Hotfix workflow:**
    ```bash
    git checkout main
    git pull origin main
    git checkout -b hotfix/critical-bug
    # ... fix the bug ...
    git add .
    git commit -m "Fix critical security bug"
    git push -u origin hotfix/critical-bug
    # ... create pull request and merge immediately ...
    ```

    **Release workflow:**
    ```bash
    git checkout main
    git pull origin main
    git tag -a v1.2.0 -m "Release version 1.2.0"
    git push origin v1.2.0
    git checkout develop
    git merge main                   # If using git-flow
    ```

19. **When things go really wrong (emergency commands):**
    ```bash
    # Find a lost commit (even after reset --hard)
    git reflog                       # Shows everything you've done
    git checkout <commit-hash>       # Go back to that commit
    git checkout -b recovery-branch  # Create branch from that point
    
    # Recover deleted branch
    git reflog                       # Find the last commit on that branch
    git checkout -b recovered-branch <commit-hash>
    
    # Undo a merge
    git reset --hard HEAD~1          # If merge was the last commit
    git revert -m 1 <merge-commit>   # If merge was pushed already
    ```

20. **Versioning strategies (how to number your releases properly):**

    **Semantic Versioning (SemVer) - the industry standard:**
    ```bash
    # Format: MAJOR.MINOR.PATCH (e.g., 2.1.3)
    # MAJOR: Breaking changes (users need to update their code)
    # MINOR: New features (backward compatible)
    # PATCH: Bug fixes (backward compatible)
    
    # Examples:
    git tag v1.0.0    # Initial release
    git tag v1.0.1    # Bug fix
    git tag v1.1.0    # New feature added
    git tag v2.0.0    # Breaking change
    ```

    **Calendar versioning (CalVer) - based on dates:**
    ```bash
    # Format: YYYY.MM.DD or YY.MM.MICRO
    git tag 2024.03.15    # Released on March 15, 2024
    git tag 24.03.1       # First release in March 2024
    ```

    **Simple incremental versioning:**
    ```bash
    # Just count up: v1, v2, v3, etc.
    git tag v1
    git tag v2
    git tag v3
    ```

    **Pre-release versions:**
    ```bash
    git tag v1.0.0-alpha.1    # Alpha version
    git tag v1.0.0-beta.2     # Beta version
    git tag v1.0.0-rc.1       # Release candidate
    git tag v1.0.0            # Final release
    ```

21. **Git ignore patterns for common files you never want to commit:**
    ```bash
    # Create .gitignore file with common patterns
    echo "node_modules/" >> .gitignore
    echo "*.log" >> .gitignore
    echo ".env" >> .gitignore
    echo "dist/" >> .gitignore
    echo ".DS_Store" >> .gitignore
    echo "*.pyc" >> .gitignore
    echo "__pycache__/" >> .gitignore
    echo "target/" >> .gitignore
    echo "*.class" >> .gitignore
    ```

22. **Completely removing a file from Git history (like it never existed):**
    ```bash
    # Remove file from Git history and working directory
    git filter-branch --force --index-filter \
      "git rm --cached --ignore-unmatch path/to/file.txt" \
      --prune-empty --tag-name-filter cat -- --all
    
    # Clean up and remove Git's backup refs
    git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
    
    # Garbage collect all references
    git reflog expire --expire=now --all
    git gc --prune=now --aggressive
    
    # Force push all branches to remote (warning: rewrites history!)
    git push origin --force --all
    git push origin --force --tags
    ```
    
    **Why each step is needed:**
    - `filter-branch` rewrites history to remove the file from all commits
    - `--force` overwrites any existing backups
    - `--index-filter` modifies the repo without checking out files (faster)
    - `--prune-empty` removes commits that would be empty
    - `--tag-name-filter cat` updates all tags to point to rewritten commits
    - `reflog expire` and `gc` clean up all traces of the file
    - `push --force` updates remote to match your rewritten history

    **⚠️ Warning:** Only use this for sensitive data or large files accidentally committed. It rewrites Git history, which can cause problems for other developers.


23. **Removing a file from Git tracking but keeping it in history:**
    ```bash
    # Remove file from Git tracking but keep it locally
    git rm --cached sensitive_file.txt
    
    # Add it to .gitignore to prevent re-tracking
    echo "sensitive_file.txt" >> .gitignore
    
    # Commit these changes
    git commit -m "Stop tracking sensitive_file.txt"
    
    # Push changes to remote
    git push origin main
    ```

    **What this does:**
    - `rm --cached` removes the file from Git tracking but keeps it in your working directory
    - Adding to `.gitignore` prevents accidentally re-adding it
    - The file remains in Git history but won't be tracked going forward
    - Unlike `filter-branch`, this preserves history and is safer for collaboration
    
    **Use this when:**
    - You want to stop tracking a file but keep it locally
    - The file's history isn't sensitive and can stay in the repo
    - You want a simpler, safer solution than rewriting history

    

24. **Working with remote branches and linking them to local:**
    ```bash
    # See all remote branches
    git branch -r                   # -r shows only remote branches
    
    # Create local branch from remote and link them (tracking)
    git checkout -b feature-local origin/feature-remote    # Creates and switches to new local branch
    
    # Alternative way to create tracking branch
    git branch --track feature-local origin/feature-remote
    git checkout feature-local
    
    # Set up tracking for existing local branch
    git branch -u origin/feature-remote feature-local      # -u or --set-upstream-to
    
    # Verify tracking relationships
    git branch -vv                  # Shows which remote branches are tracked
    
    # Push local branch and set up tracking in one command
    git push -u origin feature-local    # -u or --set-upstream
    ```

    **What each command does:**
    - `branch -r` lists all remote branches so you can see what's available
    - `checkout -b` creates new local branch and links it to remote in one step
    - `branch --track` creates the link without switching to the branch
    - `branch -u` sets up tracking for an existing local branch
    - `branch -vv` shows which local branches are tracking which remote branches
    - `push -u` both pushes your branch and sets up tracking for future pushes

    **Use this when:**
    - You need to work on a feature that exists on remote
    - You want to ensure your local branch pushes to the correct remote branch
    - You're setting up a new feature branch that needs to be shared with team


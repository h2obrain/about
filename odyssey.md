
# GIT setup

Setup user (be aware, this will appear in all your commits)
```bash
git config --global user.name "Oliver Meier"
git config --global user.email "h2obrain@gmail.com"
```

Setup editor (Important, needs to be run as a single instance)
```bash
git config --global core.editor "geany -imnst"
```

Setup diff/merge tool
```bash
git config --global diff.tool meld
git config --global difftool.prompt false
git config --global merge.tool meld
git config --global mergetool.prompt false
```

Set our default branch to *main*
```bash
git config --global init.defaultBranch main
```

We can also edit the global/local config directly
```bash
# Edit global config
git config --global -e
# Edit local config
git config -e
```

Disable pager for presentation purposes
```bash
export GIT_PAGER=cat
```


# Workflow
Basic rules
- *main* branch is never overwritten
- All new features are introduced in separate branches, then merged into *main* to keep it clean
- Prefer *rebase* over *merge*
- **TBC** There are many implicit rules I forgot to mention here


# A week long odyssey in version control

## Sunday: Cloning a repository and creating a working branch
> Playing with stuff

Clone our repository from github..
```bash
git clone git@github.com:h2obrain/odyssey.git
```

.. or create a new repository
- Initialise a local repository
  ```bash
  mkdir odyssey
  cd odyssey
  git init
  ```
- Add a README file
  ```bash
  echo "# Odyssey" > README.md
  git add -A
  git commit -m"Initial commit"
  ```
- Create an empty project on github.com
- Add remote to local repository and push it
  ```bash
  git remote add origin git@github.com:h2obrain/odyssey.git
  git push --set-upstream origin main
  ```

Create a new working branch
```bash
git checkout -b gebaschtel
```

> It's Sunday so it is fine to commit passwords. I need to store them somewhere, don't I?

Commit some malicious data
```bash
echo "My super secret password: 12345?" > password.txt
git add --all
git commit -m"Intermediate storage of password, it will be fine :). Do not push this to github"
```


## Monday: Review, add and commit changes
> Let's get some work done

Do some work
```bash
echo "Some new feature" > feature.txt
git status
git add --all
git commit -m"Added new feature"
```

Change our work
```bash
echo "More stuff added" >> feature.txt
echo "Help me, it's Monday." > note.txt
```

Review our changes
```bash
git status
git diff
git difftool -d
```

Fyrabig, let's just commit everything
```bash
git add --all
git commit -m"Extended new feature"
```


## Tuesday: Pushing changes
> Ups, apparently I forgot to push my changes to github.

We also did some more work
```bash
echo "All work and no play" > dull.txt
git add dull.txt
git commit -m"More work"
```

Store changes on github, aka push to origin
```bash
git status
git push
git push --set-upstream origin gebaschtel
```


## Wednesday: Pull changes and compare to origin
> It works! Let's put the changes into the main branch as fast as possible

Pull all our commits into main
```bash
git checkout main
git pull . gebaschtel
```

> Be careful when pushing to main.  
Compare the changes with the remote => Use the difftool
```bash
git difftool -d origin/main
```

> Oh s*! There is a password checked in, abort!
```bash
# NOTE: git reset --hard origin/main == git reset origin/main && git checkout .
git reset origin/main --hard
```

Cherry-pick only the wanted commits
- Get the commit range from our git log
```bash
git log --oneline gebaschtel
git cherry-pick AAAAAA^..BBBBBBB
```

Check changes again
```bash
git difftool -d origin/main
```

> Oh no! The history still looks bad, I do not want that *note.txt*
> Let's cleanup the history in our branch first tomorrow


## Thursday: A step back. Let's mess with history!

Reset main and checkout our working branch
```bash
git reset origin/main --hard
git checkout gebaschtel
```

> Let's change our last commit, just for the sake of it
```bash
git commit --amend
```

### Edit, remove and squash commits
Using git rebase interactively is a very nice way to clean up our history
- Run interactive rebase against our main branch
  ```bash
  git rebase -i main
  ```
- We want to get rid of the committed passwords, so let's mark it as drop (or just delete the line)
- We do not want the note.txt in our main branch, so let's edit the commit containing it.
    - Mark line as e, so rebasing stops there
- Close the editor  
  => Rebasing starts and is paused at the commit we marked for edits
- Dismiss the last commit (the one we marked for edit), but keep all its changes
  ```bash
  git reset HEAD~
  git status
  ```
- Add changes to *feature.txt* and *note.txt* as separate commits
  ```bash
  git add feature.txt
  git commit -m"Some new feature"
  git add note.txt
  git commit -m"Added note"
  git log --oneline
  git status
  ```
- Finally continue our rebase
  ```bash
  git status
  git rebase --continue
  ```

> We are still not fully happy with our history, the changes to feature could be one commit

Merge commits (squash)
- Rebase against *main*
  ```bash
  git rebase -i main
  ```
    - Mark "Some new feature" commit with s, to squash it together with the previous commit
    - Close editor
- Choose new commit message
    - Close editor

> Let's check with origin, if we did not mess up

Review our changes
```bash
git log --oneline
git difftool -d origin/gebaschtel
```

Everything looks fine, let's push our new history
```bash
git push
```

Oh! That is not allowed because we changed the history, let's force overwriting it
```bash
git push -f
```


## Friday: Cherry picking into and rebasing onto main branch
> Carefully pull our changes into our main branch

Cherry pick certain commits into main
```bash
git checkout main
git log --oneline gebaschtel
git cherry-pick AAAAAAA BBBBBBB
```

Check one last time our changes are correct, then release our selected commits into main
```bash
git difftool origin/main -d
git push
```

Rebase our working branch onto *main*
```bash
git checkout gebaschtel
git rebase main
```
The commits applied to main are automatically removed  

Check our main branch
```bash
git diff origin/gebaschtel
git push -f
```

> TGIF: Let's Party! 🎉🍺



## Saturday: Recover lost files
> Ah, my head! Dude, where is my password!?


> Remember we deleted our commit with the password the other day?

Fear not, we can restore it using the local reflog
- Find the commit with the password
  ```bash
  git reflog
  ```
- Let's cat the whole change object
  ```bash
  git cat-file -p AAAAAAA
  ```
- Get the file tree
  ```bash
  git cat-file -p BBBBBBB
  ```
- Get the file
  ```bash
  git cat-file -p CCCCCCC
  ```
- Voilà! Password restored!

Alternatively we could just use cherry-picking (or so) to get the file
```bash
git checkout -b pwd_restore
git cherry-pick 6ac760e
git checkout gebaschtel
git branch -D pwd_restore
```

=> The room fills with light and angels sing!



# Sunday: Cleaning history
Did we just restore something that should be lost forever? Let's (carefully) clean up!  
> or let's really mess up our history!

Remove unreachable entries from reflog

- As long as the local reflog points to an object, it is not considered dangling, so we need to expire (delete) those references from our local reflog.
  ```bash
  git fsck --full --dangling
  git reflog
  #git reflog expire --expire=now --all
  git reflog expire --expire-unreachable=now --all
  git fsck --full --dangling
  git reflog
  ```
- We need to extract dangling objects from our object packs
  ```bash
  git repack -ad
  ```
- Now we can remove them
  ```bash
  git prune
  git push
  ```
- Alternatively we could use `git gc`
  ```bash
  git gc --prune=now --aggressive
  ```

But what about github? It surely is still available there..

- Using the the [github api](https://docs.github.com/en/rest) we can access the "event log" (basically reflog on the server side). Other git servers have different apis, but the same functionality is available in most..
  [https://api.github.com/repos/h2obrain/odyssey/events](https://api.github.com/repos/h2obrain/odyssey/events)
  > see also here https://objectpartners.com/2014/02/11/recovering-a-commit-from-githubs-reflog/
- Find "password" commit, click the link
- Find the "raw" entry, click the link
  > Aah! My password is on the internet!
- Removing dangling commits from github is not straight forward (AFAIK).  
    - We can follow the steps explained in the github documentation:  
      [https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).
- The GitLab documentation also provides some information about purging files:  
  [https://docs.gitlab.com/ee/user/project/repository/reducing_the_repo_size_using_git.html](https://docs.gitlab.com/ee/user/project/repository/reducing_the_repo_size_using_git.html)



# to be continued..

Repo cleaning tools
- [git-repo-filter](https://github.com/newren/git-filter-repo)
- [bfg-repo-cleaner](https://rtyley.github.io/bfg-repo-cleaner/)


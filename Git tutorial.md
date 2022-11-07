# Git tutorial

[toc]



### Introduction

1. Version control system (VCS) for tracking changes in computer files.
2. Git is a d**istributed version control system (DVCS)** or **decentralized control system**, meaning many developers can work on the **same** **project** without having to be on the **same network**. 

3. you can **push** you local **repository** to a **remote repository**.
4. keeps **track of code history** with **snapshot** of your files.
5. Take a **snapshot** by making a **commit**.
6. you can **visit any snapshot** at **any time.**
7. You can put the code in **staging area** before doing commit and writing it to a snapshot. `add`

8. After code **being pushed to** the remote repository, other developers can **pull** it.



1. `git init` Initialize local git repository
2. `git add <file>` Add files to the **staging** area, run the command as many times as you like
3. `git status` check status
4. `git commit` 
5. `git push` push to remote repository
6. `git pull` pull lates version from remote repository
7. `git clone` clone/download repository to my local machine



### Create a new repository on the command line

1. ```
   echo "# UQproject" >> README.md
   git init
   git add README.md
   git commit -m "first commit"
   git branch -M main
   git remote add origin https://github.com/Renshui-MC/UQproject.git
   git push -u --force origin main //--force to merge
   ```



### Push an existing repository from the command line

```
git remote add origin https://github.com/Renshui-MC/UQproject.git
git branch -M main
git push -u origin main
```



### Delete commits from a branch in Git

##### For not pushed

```
git reset --hard HEAD~1 or git reset --hard HEAD^ //remove the last commit from git
git reset --hard HEAD~2 //remove the last two commits (you can increase the number to remove even more commits)


```

If you want to "uncommit" the commits, but keep the changes around for reworking, remove the `--hard`: `git reset HEAD^` which will evict the commits from the branch and from the index, but leave the working tree around. 

If  you want to save the commits on a new branch name, then run `git branch newbranchname` before doing the git reset.



If you'd like to delete the commits up until a specific commit, running `<git log>`into the command line to find the specific commit id and then running

```
git reset --hard <shal-commit-id>
```

will discard all working tree changes and move HEAD to the commit chosen. 



##### For already pushed

```
git reset --hard <sha1-commit-id> //if you want to delete a particular commit: "git log" to find its commit id
git reset --hard HEAD^//must delete locally first
git push origin HEAD --force
```

Please note if others have pulled this branch you would be better off starting a new branch. If you don't do this when someone else pulled, it will just merge it into their work, and you will get it pushed back up again. **Once you have done `git reset --hard <sha1-commit-id>` a pointer will named "HEAD" will move to the current state, go check it (git log) with provided commit id to see where you are now.** Then `git push origin HEAD --force` will delete the already pushed commit remotely. **ALWAYS check your current status(`git status` and ensure it is empty) before `git reset` because you will lose all uncommitted changes!!** 



**If want to delete all commits history and start taking new commits history from the current local status**

`````

-- Remove the history from 
rm -rf .git

-- recreate the repos from the current content only
git init
git add .
git commit -m "Initial commit"

-- push to the github remote repos ensuring you overwrite history
git remote add origin https://github.com/Renshui-MC/UQproject.git
git push -u --force origin main
`````

**Very good explanation:** [How to delete commits](https://stackoverflow.com/questions/9529078/how-do-i-use-git-reset-hard-head-to-revert-to-a-previous-commit)



### git branch

[push local branch on Github](https://stackoverflow.com/questions/5423517/how-do-i-push-a-local-git-branch-to-master-branch-in-the-remote)

` git push origin develop:master`

`git push <remote> <local branch name>:<remote branch to push into>`

[branching](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging#_basic_merging)

1. `git checkout -b <newbranch>` switch to a new branch "newbranch"

   equivalent to 

   `git branch <newbranch>`  create "newbranch"

   `git checkout <newbranch>` switch to "newbranch"

2. After you feel satisfied with the content in "newbranch", merge "newbranch" into your "main"

   `git checkout <newbranch>`

   `git merge <newbranch>`

3. Then you no longer need "newbranch" any more and need to delete it locally

   `git branch -d <newbranch>`

   delete it remotely

   `git push origin --delete <newbranch>` for more useful commands refer to [deleting branches](https://gist.github.com/cmatskas/454e3369e6963a1c8c89)



### git stash

`git stash`

Now you want to switch branches, but you don’t want to commit what you’ve been working on yet, so you’ll stash the changes. To push a new stash onto your stack, run `git stash` or `git stash push`



`git stash list`

At this point, you can switch branches and do work elsewhere; your changes are stored on your stack. To see which stashes you’ve stored, you can use `git stash list`:



`git stash apply`

In this case, two stashes were saved previously, so you have access to three different stashed works.
You can reapply the one you just stashed by using the command shown in the help output of the original stash command:



`git stash apply stash@{2}`

If you want to apply one of the older stashes, you can specify it by naming it, like this: `git stash apply stash@{2}`

[git stash explanation](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning)



### [How can I determine the URL that a local Git repository was originally cloned from?](https://stackoverflow.com/questions/4089430/how-can-i-determine-the-url-that-a-local-git-repository-was-originally-cloned-fr)

`git config --get remote.origin.url`



### How to undo a git add with Git Reset

`git reset <file>` or `git reset` to unstage all changes

[how to undo a git add with Git reset](https://forum.freecodecamp.org/t/how-to-undo-a-git-add-with-git-reset/13237)

/home/minghan/OpenFOAM/minghan-v1812/src/TurbulenceModels/turbulenceModels/MyUQ/MyUQ.C



### Git to get the files that have changed

`git diff --name-only HEAD (HEAD~3)` note `()` is added to indicate this is optional: in last 3 commits

`git diff --name-status HEAD HEAD~3` add action on each file to know if the file was deleted (D), modified (M), or added (A) 

`git diff --name-only HEAD~10 HEAD~5`Note differences between the tenth latest commit and the fifth latest 

 


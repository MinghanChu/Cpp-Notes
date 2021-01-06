# Git tutorial

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



**git branch**

[push local branch on Github](https://stackoverflow.com/questions/5423517/how-do-i-push-a-local-git-branch-to-master-branch-in-the-remote)

```
1. git checkout -b newbranch-Openfoam //checkout the current branch and create a new branch named "newbranch-Openfoam"

2. go created a new branch named "newbranch-Openfoam" on github

3. git push origin newbranch-Openfoam:newbranch-Openfoam //push files under the "newbranch-Openfoam" to the branch named "newbranch-Openfoam" on github (first newbranch-Openfoam means the branch github and the second one means that created on your local machine)

git checkout main //checkout the current branch and move to the branch wanted ("main" for this case)

```


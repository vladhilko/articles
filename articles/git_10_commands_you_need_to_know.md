## Overview:
> In this article, you will learn the 10 essential Git commands that every programmer should know. These commands will allow you to efficiently manage your code repository, collaborate with your team, and streamline your workflow. Whether you are a beginner or an experienced programmer, these commands will be valuable tools in your toolkit for mastering Git.

## Here are 10 GIT commands that we will discuss today:


1. `git commit --amend --no-edit`
2. `git cherry-pick oldest_commit^..newest_commit`
3. `git stash`
4. `git branch -m old_name new_name`
5. `git clean -fd`
6. `git-bisect`
7. `git fetch origin your_teammate_branch`
8. `git blame --incremental`
9. `gitk`
10. `git merge origin/main`


---

### Let's start.

---

## 1. `git commit --amend --no-edit`

**Problem:**

You realize that you forgot to include some minor changes, but these changes are too small to create a separate commit and potentially confuse the commit history. For example, adding missing spaces between variables or fixing issues highlighted by a code linter.

**Solution:**

Instead of creating a new commit to fix the mistake, you can use `git commit --amend --no-edit` to modify the most recent commit and add any missing changes to it.

```console
foo@bar:~$ git add .
foo@bar:~$ git commit --amend --no-edit
```

## 2. `git cherry-pick oldest_commit^..newest_commit`

**Problem:**

You want to apply a series of commits from one branch to another branch, but you don't want to merge the entire branch.
For example, you and your colleague are working on the same features. You both need to implement the same basic logic to complete the task. So you can wait for him to implement them and then add them to your branch.

**Solution:**

By using the `git cherry-pick` command with a range of commits specified by `oldest_commit^..newest_commit`, you can selectively apply specific commits from one branch to another.

```console
foo@bar:~$ git cherry-pick oldest_commit^..newest_commit
```

## 3. `git stash`

**Problem:**

Someone asked you to fix an urgent bug right now, but you're in the middle of your development and don't want to commit the raw code.

**Solution:**

By using `git stash`, you can easily store your changes and switch branches or restore your working directory to a clean state, without losing your local changes or having to commit them prematurely. You can then apply the stashed changes later when you're ready to work on them again.

```console
foo@bar:~$ git stash
foo@bar:~$ git checkout -b <urgent-bug-fix>
# Bug has been fixed
foo@bar:~$ git checkout <your-feature-branch>
foo@bar:~$ git stash apply
```

## 4. `git branch -m old_name new_name`

**Problem:**

You accidentally created a branch with the wrong name, but realized it too late and all the work had already been done. You could cherry-pick all commits to a new branch but it's too complicated.

**Solution:** 

By using `git branch -m old_name new_name`, you can quickly and easily rename your branch.

```console
foo@bar:~$ git branch -m old_name new_name
```

## 5. `git clean -fd`

**Problem:**

You're trying to implement a draft feature or test something on a production-ready branch. After testing, you want to reset your working directory to a clean state, discarding all local changes and untracked files.

**Solution:** 

By using `git clean -fd`, you can quickly and easily remove untracked files and directories from your working directory.
The `-f` flag tells git clean to forcibly remove the files, and the `-d` flag tells it to remove directories as well.
To remove changes in the existing files you can use `git checkout .`

```console
foo@bar:~$ git checkout .
foo@bar:~$ git clean -fd
```

## 6. `git-bisect`

**Problem:**

Let's say that you have been working on a project for several weeks, and everything has been running smoothly. Suddenly, you notice that one of your features is not working as expected. You suspect that the problem was introduced in one of your recent commits, but you are not sure which one.

**Solution:** 

You can use `git bisect` to perform a binary search through your commit history to find the commit that introduced the bug. This can save you a lot of time and effort in debugging, as you can narrow down the problem to a specific commit rather than having to manually check each commit individually.

```console
# To start a git bisect session, use the command git bisect start. 
# This will mark the current commit as the "bad" commit, and you will need to specify a commit that you know is good
# (i.e., it does not contain the bug or regression you are trying to identify). 
foo@bar:~$ git bisect start
# You can specify a good commit using the git bisect good command.
foo@bar:~$ git bisect good c7a946e
# or
foo@bar:~$ git bisect bad
# After running the git bisect command multiple times, you will eventually arrive at the commit that introduced the bug or regression. 
```

## 7. `git fetch origin your_teammate_branch`

**Problem:**

You are in the middle of a code review. You notice that you can improve something, but you're not sure if it works or not. That's why, before you add a comment, you want to test your theory locally.

**Solution:** 

You can use `git fetch origin branch` to download the specific branch from the remote repository and check these changes locally.

```console
foo@bar:~$ git fetch origin <your-teammate-branch>
foo@bar:~$ git checkout <your-teammate-branch>
```

## 8. `git blame --incremental`

**Problem:**

You want to change some piece of code, but you need to figure out the original reason why that code was done that way. The standard blame points to Pull Request, where the person just changed all project tabs to spaces, and the git history is now overwritten. 

**Solution:** 

By using `git blame` with the `--incremental` option, you can see the history of changes made to a specific line of code, which can be helpful when you are trying to understand how a particular code block has evolved over time. But `git blame` searches the commit history to find the last person who modified a line of code. This is rarely what you want. More often what you want is the original author of a line. Here's the solution: `git log -p -M --follow --stat -- path/to/your/file`

```console
# To find the last person who modified line 10 in the file.
foo@bar:~$ git blame -L10,+1 path/to/your/file --incremental
# To see the entire history of changes and find the original author of the line.
foo@bar:~$ log -p -M --follow --stat -- path/to/your/file
```

## 9. `gitk`

**Problem:**

You made some changes and want to review the stashed data before committing, to ensure that you haven't added any unnecessary or unwanted files.

**Solution:** 

Using `gitk`, you can view the stashed data and see the changes that you have made, which can help you identify any unwanted or unnecessary files and remove them before committing your changes.

![GITK EXAMPLE](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o0z7cneunwona56znroh.png)



```console
foo@bar:~$ gitk
```


## 10. `git merge origin/main`

**Problem:**

You have a Pull Request which is already approved and you're ready to merge it. But you want to update your branch with the latest version of `main` to make sure that your changes won't break anything in the new version. You can use the following command `git pull origin main --rebase`, but this command requres a `force` push and you'll lose you approvals. 

**Solution:** 

Update your branch using `git merge origin/main`.

```console
foo@bar:~$ git fetch origin
foo@bar:~$ git merge origin/main
```

## Conclusion

In conclusion, mastering Git requires a solid understanding of the various commands available and how to use them effectively. The 10 essential commands covered in this article are a good starting point to simplify your daily routine.

1. <del> `git commit --amend --no-edit` </del>
2. <del> `git cherry-pick oldest_commit^..newest_commit` </del>
3. <del> `git stash` </del>
4. <del> `git branch -m old_name new_name` </del>
5. <del> `git clean -fd` </del>
6. <del> `git-bisect` </del>
7. <del> `git fetch origin your_teammate_branch` </del>
8. <del> `git blame --incremental` </del>
9. <del> `gitk` </del>
10. <del> `git merge origin/main` </del>

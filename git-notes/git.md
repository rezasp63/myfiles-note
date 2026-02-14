# GIT NOTES

## Introduction:
### What is Git?

Git is a distributed version control system (VCS) used to track changes in source code and collaborate with others.

*In simple terms, Git helps you:*

* Keep a history of your code

* Work on features without breaking the main code

* Collaborate with multiple people safely

* Roll back to earlier versions if something goes wrong

Git was created by Linus Torvalds in 2005 for Linux kernel development.

### How to install GIT :

**Debian/Ubuntu**

For the latest stable version for your release of Debian/Ubuntu
```bash
apt-get install git
```

For Ubuntu, this PPA provides the latest stable upstream Git version

```bash
add-apt-repository ppa:git-core/ppa
apt update
apt install git
```
### Prepartion for Git:

#### first step:
check instalation

```bash
git --version
```
#### second step:

**Set your name and email and DefualtBranch name (must do once)**
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase "true"
```
Check:
```bash
git config --global --list
```
View all active config (from all levels)
```bash
git config --list
```
**Optional:**
Common Useful Configurations
Default editor
```bash
git config --global core.editor "vim"
```
Enable colored output
```bash
git config --global color.ui auto
```
Default branch name
```bash
git config --global init.defaultBranch main
```

## Getting and Creating Projects:
1. initiate git repository:


git-init - Create an empty Git repository or reinitialize an existing one

```bash
cd /home/user/my_project
git init 
```
2. Clone repository from Github:

 git-clone - Clone a repository into a new directory

 ```bash
 git clone https://github.com/libgit2/libgit2
 ```
## Git Architectures:

![Git Workflow Digram](workflow.jpg)

**Workflow Steps**
> [!IMPORTANT]
> 1. Local Repository Setup
>    - A developer clones a remote repository (e.g., from GitHub, > GitLab).
>    - This creates three areas locally:
>- Working Directory — where you modify files.
> ```bash
>  vim test.txt
>  touch tets.txt
> ```
> > [NOTE]
>  In this step file move **not staged** status (tracked and not stage):
> For comback file to working directory:
> ```bash
>  git restore test.txt # unchanged file back to working dirctoryfor more edit
> ```
> For move file to **staged** status:
> ```bash
>  git add test.txt 
>  git restore --staged <file> # file back to not stage status
> ```
>- Staging Area (Index) — where you prepare commits.
>```bash
>  git commit -m "message"
>```
> **how to turn back file after comit**
> 1. git reset
>Purpose:
>
>Moves the current branch’s HEAD backward to a previous commit. Depending on the mode, it can affect the staging area and > working directory.
>
>Modes:
>
> **--soft** → Moves HEAD only; keeps changes staged.
> 
> **--mixed (default)** → Moves HEAD and clears the staging area, keeps local changes.
> 
> **--hard** → Moves HEAD and deletes all changes (⚠ irreversible).
> 
>Examples:
>```bash
>git reset --soft HEAD~1   # Undo last commit, keep changes staged
>git reset --hard HEAD~3   # Undo last 3 commits and delete changes
> ```
>- Local Repository (.git) — where commits are stored.
>2. Commit Changes
>    - Files are staged and committed using:
>3. Push (Local → Remote)
>    - Transfers commits from local repository to remote repository:
>```bash
>   git push origin main
>```
>4. Fetch (Remote → Local metadata)
>    - Downloads info about new commits from remote but doesn’t merge:
>```bash
>   git fetch origin
>```
>5. Pull (Remote → Local full sync)
>    - Fetch + Merge together:
>```bash
>   git pull origin main
>```
>6. Branch merges and updates keep repositories synchronized across multiple collaborators.
>
>


 ***Diagram Explanation***
Here’s what the Git data transfer workflow diagram includes:

```
  +-----------------------+
   |      Remote Repo      |
   |   (GitHub / GitLab)   |
   +----------▲------------+
              |
         git push / pull
              |
   +----------▼------------+
   |     Local Repository  |
   |     (.git directory)  |
   +----------▲------------+
              |
       git commit / merge
              |
   +----------▼------------+
   |   Staging Area (Index) |
   +----------▲------------+
              |
           git add
              |
   +----------▼------------+
   |   Working Directory   |
   | (Source Code, Files)  |
   +-----------------------+
```

## Recording Changes to the Repository:
### Checking the Status of Your Files
```bash
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean

$ echo 'My Project' > README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```
### Tracking New Files:

```bash
$ git add CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```
Short Status
```bash
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```
To see what you’ve changed but not yet staged, type git diff with no other arguments:
```bash
$ git diff
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
index 8ebb991..643e24f 100644
--- a/CONTRIBUTING.md
+++ b/CONTRIBUTING.md
@@ -65,7 +65,8 @@ branch directly, things can get messy.
 Please include a nice description of your changes when you submit your PR;
 if we have to read the whole diff to figure out why you're contributing
 in the first place, you're less likely to get feedback and have your change
-merged in.
+merged in. Also, split your changes into comprehensive chunks if your patch is
+longer than a dozen lines.
```
***Ignoring Files***

Often, you’ll have a class of files that you don’t want Git to automatically add or even show you as being untracked. These are generally automatically generated files such as log files or files produced by your build system. In such cases, you can create a file listing patterns to match them named .gitignore. Here is an example .gitignore file:

```bash
$ vim .gitignore

# ignore all .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in any directory named build
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory and any of its subdirectories
doc/**/*.pdf
```
**Removing Files**
```bash
$ rm PROJECTS.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    PROJECTS.md

no changes added to commit (use "git add" and/or "git commit -a")
```
OR

```bash
git rm <file>
git rm -f <file>
```

## Viewing the Commit History
View Commit History:

```bash
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    Remove unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    Initial commit
```
 -p or --patch, which shows the difference (the patch output) introduced in each commit

 ```bash
 $ git log -p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
 spec = Gem::Specification.new do |s|
     s.platform  =   Gem::Platform::RUBY
     s.name      =   "simplegit"
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
     s.author    =   "Scott Chacon"
     s.email     =   "schacon@gee-mail.com"
     s.summary   =   "A simple gem for using Git in Ruby code."

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    Remove unnecessary test

diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
     end

 end
-
-if $0 == __FILE__
-  git = SimpleGit.new
-  puts git.show
-end
```
other usefull:
```bash
git log --oneline
git log --oneline --graph
```
## Working with Remotes:
**List remote:**
```bash
$ git clone https://github.com/schacon/ticgit
Cloning into 'ticgit'...
remote: Reusing existing pack: 1857, done.
remote: Total 1857 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1857/1857), 374.35 KiB | 268.00 KiB/s, done.
Resolving deltas: 100% (772/772), done.
Checking connectivity... done.
$ cd ticgit
$ git remote
origin
```
more detailes:
```bash
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```

**Adding Remote Repositories:**
```bash
git remote add <shortname> <url>

$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)

Now you can use the string pb on the command line instead of the whole URL. For example, if you want to fetch all the information that Paul has but that you don’t yet have in your repository, you can run git fetch pb

$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch]      master     -> pb/master
 * [new branch]      ticgit     -> pb/ticgit
```

> IMPORTANT NOTE
> 
>
>From Git version 2.27 onward, git pull will give a warning if the pull.rebase variable is not set. Git will keep warning you until you set the variable.
>
>If you want the default behavior of Git (fast-forward if possible, else create a merge commit): git config --global pull.rebase "false"
>
>If you want to rebase when pulling: git config --global pull.rebase "true"













---
layout: post
title:  "Discovery Exercise on Git"
date:   2017-09-30 01:00:00
categories: 
tags: git scm
---

<H1>Discovery Exercise on Git for Beginners</H1>

There are many online trainings for git
<ul>
<li> <A href="https://www.google.fr/search?q=git+training">https://www.google.fr/search?q=git+training</A> </li>
<li> <A href="https://try.github.io/">https://try.github.io/</A> </li>
<li> <A href="https://github.com/praqma-training/gitkatas">https://github.com/praqma-training/gitkatas</A> </li>
</ul>

and many docs
<ul>
<li> <A href="http://arnaud.nauwynck.free.fr/CoursIUT/CoursIUT-IntroGIT.pdf">http://arnaud.nauwynck.free.fr/CoursIUT/CoursIUT-IntroGIT.pdf</A> </li>
<li> <A href="http://en.wikipedia.org/wiki/Git">http://en.wikipedia.org/wiki/Git</A> </li>
<li> <A href="http://git-scm.com/">http://git-scm.com/</A> </li>
<li> <A href="https://github.com/">https://github.com/</A> </li>
<li> <A href="https://github.s3.amazonaws.com/media/progit.en.pdf">https://github.s3.amazonaws.com/media/progit.en.pdf</A> </li>
<li> <A href="http://git-scm.com/book/fr">http://git-scm.com/book/fr</A> </li>
<li> <A href="http://ndpsoftware.com/git-cheatsheet.html">http://ndpsoftware.com/git-cheatsheet.html</A> interactive doc for Workspace/Index/Repo</li>
<li> <A href="http://onlywei.github.io/explain-git-with-d3/">http://onlywei.github.io/explain-git-with-d3/</A> </li>
<li> <A href="https://marklodato.github.io/visual-git-guide/index-en.html">https://marklodato.github.io/visual-git-guide/index-en.html</A> interactive command drawings</li>
<li> <A href="https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf">https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf</A> </li>
</ul>





<H2>Question 1 : Discovering local repositories</H2>

Open a shell terminal, create a new directory, change dir into it

{%highlight shell%}
$ mkdir tp-git
$ cd tp-git
{%endhighlight%}

Verify that git command does not detect any local repository
{%highlight shell%}
$ git status
fatal: Not a git repository (or any of the parent directories): .git
{%endhighlight%}
 

Init a new local git repository 
{%highlight shell%}
$ git init
Initialized empty Git repository in /home/arnaud/tp-git/.git/
{%endhighlight%}

Check git repository status:
{%highlight shell%}
$ git status
Initialized empty Git repository in /home/arnaud/tp-git/.git/
{%endhighlight%}

Explore the (hidden) content of the directory:
{%highlight shell%}
$ ls

$ ls -a
.  ..  .git

$ find .
.
./.git
./.git/config
./.git/info
./.git/info/exclude
./.git/objects
./.git/objects/info
./.git/objects/pack
./.git/HEAD
./.git/description
./.git/hooks
./.git/hooks/pre-applypatch.sample
./.git/hooks/pre-commit.sample
./.git/hooks/applypatch-msg.sample
./.git/hooks/post-update.sample
./.git/hooks/update.sample
./.git/hooks/prepare-commit-msg.sample
./.git/hooks/pre-push.sample
./.git/hooks/commit-msg.sample
./.git/hooks/pre-rebase.sample
./.git/branches
./.git/refs
./.git/refs/heads
./.git/refs/tags
{%endhighlight %}

Read the .git/config file  ... nothing special yet
{%highlight shell%}
$ cat ./.git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
{%endhighlight%}


Configure your git global settings

{%highlight shell%}
git config --global user.name <<YourFirstname Lastname>>
git config --global user.email <<yourfirstname.lastname@gmail.com>>
{%endhighlight %}


Add Some basic Alias
{%highlight shell%}
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lol "log --graph --decorate --pretty=oneline --abbrev-commit --all"
git config --global alias.oneline "log --pretty=oneline --abbrev-commit --graph"
{%endhighlight %}


Example of alias and global setting son my account
{%highlight shell%}
$ git config --global -l

user.email=arnaud.nauwynck@gmail.com
user.name=Arnaud Nauwynck
push.default=simple
alias.st=status
alias.df=diff
alias.co=checkout
alias.ci=commit
alias.br=branch
alias.who=shortlog -sne
alias.oneline=log --pretty=oneline --abbrev-commit --graph
alias.lol=log --graph --decorate --pretty=oneline --abbrev-commit --all
alias.changes=diff --name-status
alias.dic=diff --cached
alias.diffstat=diff --stat
alias.lc=!git oneline ORIG_HEAD.. --stat --no-merges
alias.addm=!git-ls-files -m -z | xargs -0 git-add && git status
alias.addu=!git-ls-files -o --exclude-standard -z | xargs -0 git-add && git status
alias.rmm=!git ls-files -d -z | xargs -0 git-rm && git status
alias.mate=!git-ls-files -m -z | xargs -0 mate
alias.mateall=!git-ls-files -m -o --exclude-standard -z | xargs -0 mate
alias.undo=git reset --soft HEAD^
{%endhighlight %}

These settings can also be viewed/edited directly in text file ~/.gitconfig:

{%highlight shell%}
$ cat ~/.gitconfig
[user]
	email = arnaud.nauwynck@gmail.com
	name = Arnaud Nauwynck
[push]
	default = simple
[alias]
    st = status
    df = diff
    co = checkout
    ci = commit
    br = branch
    who = shortlog -sne
    oneline = log --pretty=oneline --abbrev-commit --graph
    lol = log --graph --decorate --pretty=oneline --abbrev-commit --all
    changes = diff --name-status
    dic = diff --cached
    diffstat = diff --stat
    lc = !git oneline ORIG_HEAD.. --stat --no-merges
    addm = !git-ls-files -m -z | xargs -0 git-add && git status
    addu = !git-ls-files -o --exclude-standard -z | xargs -0 git-add && git status
    rmm = !git ls-files -d -z | xargs -0 git-rm && git status
    mate = !git-ls-files -m -z | xargs -0 mate
    mateall = !git-ls-files -m -o --exclude-standard -z | xargs -0 mate
    undo = git reset --soft HEAD^
{%endhighlight %}




<H1>Question 2 - discovering clone remote repositories</H1>

To compare with a remote repository clone, open another shell terminal, 
{%highlight shell%}
$ git clone https://github.com/praqma-training/gitkatas
$ cd gitkatas
{%endhighlight%}

Read the .git/config file  ... 
{%highlight shell%}
$ cat ./.git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = https://github.com/praqma-training/gitkatas
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
{%endhighlight%}


Listing all the "remote" can also be done using command line:
{%highlight shell%}
$ git remote show
origin

$ git remote show origin
* remote origin
  Fetch URL: https://github.com/praqma-training/gitkatas
  Push  URL: https://github.com/praqma-training/gitkatas
  HEAD branch: master
  Remote branches:
    figaw-patch-1 tracked
    master        tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)

{%endhighlight%}



<H1>Question 3 - git add, git commit</H1>

Save some files, add them to git, and commit them
{%highlight shell%}
$ echo "test" > file1.txt
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	file1.txt

nothing added to commit but untracked files present (use "git add" to track)
{%endhighlight%}

Add your file content to git:
{%highlight shell%}
$ git add file1.txt
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   file1.txt


{%endhighlight%}

Commit your files
{%highlight shell%}
$ git commit -m "initial commit"
[master (root-commit) 865fda9] initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 file1.txt


$ git status
On branch master
nothing to commit, working directory clean
{%endhighlight%}


Modify again your file

{%highlight shell%}
echo " -version2" >> file1.txt 
$ git st
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   file1.txt

no changes added to commit (use "git add" and/or "git commit -a")
{%endhighlight%}



<H1>Question 4 - diff, browse history, undo local modification, compare revision, undo commit</H1>


add more commits:
{%highlight shell%}
 for i in 1 2 3 4 ; do echo $i >> file1.txt; git add file1.txt; git ci -m "add text $i"; done
[master 5e35789] add text 1
 1 file changed, 2 insertions(+)
[master c54c9c7] add text 2
 1 file changed, 1 insertion(+)
[master 8cb4301] add text 3
 1 file changed, 1 insertion(+)
[master f17cf73] add text 4
 1 file changed, 1 insertion(+)
{%endhighlight%}


{%highlight shell%}
$ git log
commit f17cf73e873543e32c740dfb2702bc041644bda4
Author: Arnaud Nauwynck <arnaud.nauwynck@gmail.com>
Date:   Sat Sep 30 05:57:06 2017 +0200

    add text 4

commit 8cb430127ae075b7c0ac15c22e1ed1be41f30a31
Author: Arnaud Nauwynck <arnaud.nauwynck@gmail.com>
Date:   Sat Sep 30 05:57:06 2017 +0200

    add text 3

commit c54c9c7635e9d729f9664b0180d0463631242ca9
Author: Arnaud Nauwynck <arnaud.nauwynck@gmail.com>
Date:   Sat Sep 30 05:57:06 2017 +0200

    add text 2

commit 5e3578944d01b6562d585a305f9d1fcc5daae56c
Author: Arnaud Nauwynck <arnaud.nauwynck@gmail.com>
Date:   Sat Sep 30 05:57:06 2017 +0200

    add text 1

commit 865fda9ed4860fb47e196d6fa68a9afd4a0b81ff
Author: Arnaud Nauwynck <arnaud.nauwynck@gmail.com>
Date:   Sat Sep 30 05:26:44 2017 +0200

    initial commit
{%endhighlight%}

See also

{%highlight shell%}
$ git lol 
* f17cf73 (HEAD -> master) add text 4
* 8cb4301 add text 3
* c54c9c7 add text 2
* 5e35789 add text 1
* 865fda9 initial commit

$ git oneline  # idem.. (no branches yet)
{%endhighlight%}


Do a local un-committed modif:

{%highlight shell%}
$ echo "temporary-change" >> file1.txt
$ git status

$ git diff 
diff --git a/file1.txt b/file1.txt
index e685709..bf1f340 100644
--- a/file1.txt
+++ b/file1.txt
@@ -4,3 +4,4 @@ test
 2
 3
 4
+temporary-change
{%endhighlight%}


Undo local uncommitted changes:
{%highlight shell%}
$ git checkout HEAD -- file1.txt
$ git status
{%endhighlight%}


Re-checkout file content from previous commit:
{%highlight shell%}
$ git co c54c9c7 -- file1.txt
{%endhighlight %}

compare with (equivalent?):
{%highlight shell%}
$ git co HEAD^^ -- file1.txt
$ git co HEAD~~ -- file1.txt
$ git co HEAD~{2} -- file1.txt
$ git co master^^ -- file1.txt
{%endhighlight %}


Amend a commit to change it (change content files, or change comment)
{%highlight shell%}
$ git commit --amend -m "changed comment"
{%endhighlight %}

Undo a commit  (dangerous rewrite...)
{%highlight shell%}
$ git reset HEAD^
$ git lol
* 8cb4301 (HEAD -> master) add text 3
* c54c9c7 add text 2
* 5e35789 add text 1
* 865fda9 initial commit

$ git reset HEAD^
$ git lol
* c54c9c7 (HEAD -> master) add text 2
* 5e35789 add text 1
* 865fda9 initial commit
{%endhighlight %}



<H1>Question 5 - branch, merge</H1>

Create a local new branch, swicth to this new branch

{%highlight shell%}
$ git branch
* master

$ git branch branch1
$ git branch
  branch1
* master

$ git co branch1 
Switched to branch 'branch1'
$ git branch
* branch1
  master
{%endhighlight%}

Do some modification in branch1:
{%highlight shell%}
$ echo "modif on branch1" >> file1.txt 
$ echo "new file on branch1" > file2.txt
$ git add .
$ git ci -m "modif file1.txt and added file2.txt on branch1"
[branch1 2d25a1d] modif file1.txt and added file2.txt on branch1
 2 files changed, 2 insertions(+)
 create mode 100644 file2.txt

$ git lol 
* 2d25a1d (HEAD -> branch1) modif file1.txt and added file2.txt on branch1
* c54c9c7 (master) add text 2
* 5e35789 add text 1
* 865fda9 initial commit
{%endhighlight%}

Now switch back to branch master, check no modifications from branch1 are present

{%highlight shell%}
$ git co master
Switched to branch 'master'

$ git st
On branch master
nothing to commit, working directory clean
$ ls
file1.txt
$ cat file1.txt 
{%endhighlight%}

Then merge changes from branch1 to this branch (master)
{%highlight shell%}
$ git merge branch1 
Updating c54c9c7..2d25a1d
Fast-forward
 file1.txt | 1 +
 file2.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 file2.txt

$ ls
$ cat file1.txt 
{%endhighlight%}


Reapeat Same question... but do simultaneously changes on "branch1" and "master" before merging (on separate file names, so not causing conflicts)

before merge:
{%highlight shell%}
$ git lol
* f9d1431 (HEAD -> branch1) change branch1
| * b82a66d (master) change master
|/  
* 2d25a1d modif file1.txt and added file2.txt on branch1
{%endhighlight%}

after merge:
{%highlight shell%}
$ git lol
*   2571f53 (HEAD -> master) Merge branch 'branch1'
|\  
| * f9d1431 (branch1) change branch1
* | b82a66d change master
|/  
* 2d25a1d modif file1.txt and added file2.txt on branch1
{%endhighlight%}



<H1>Question 6 - Create your own Github account, and repository</H1>

go to <A href="https://github.com">https://github.com</A>
Create an account, then create a repository "csid-exercises"

clone on your local PC your new github repository:

{%highlight shell%}
$ cd cours
$ clone https://github.com/<<your-account>>/csid-exercises
{%endhighlight%}
 

<H1>Question 7 - Create a github private key</H1>

Go in <A href="https://github.com/settings/profile">https://github.com settings menu</A> 
<BR/>
Then go in sub-section for <A href="https://github.com/settings/keys">SSH and GPG Keys</A>  
<BR/>

{%highlight shell%}
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
{%endhighlight%}

Then add it to your github account (add public key part... keep private key secret)

There is a doc at bottom of page: 
Learn how to <A href="https://help.github.com/articles/generating-an-ssh-key/">generate SSH keys</A>

Notice you could retype your password when pushing commits to github if you don't have your private key on a PC.


<H1>Question 8 - Commit and Push to Github</H1>

Commit and push files to your new "csid-exercises" github repository

Save the commands you typed for these exercises as a text file "csid-exercises/tp-git/tp.txt"

Also add a short description markdown file for this repo: "csid-exercises/README.md"

{%highlight shell%}
$ git add README.md
$ git add tp-git/
$ git commit -m "initial commit"
$ git push
{%endhighlight %}


Setup a second local clone of the same remote repository.
Try pull/fetch/push ... 


{%highlight shell%}
$ cd ~/cours
$ git clone https://github.com/<<your-account>>/csid-exercises  csid-exercises2
$ cd csid-exercises2
$ git pull 
$ echo ".." >> ftp-git/file1.txt; git add .; git ci -m "update"; git push  

.. 
$ cd ~/cours/csid-exercises
$ git pull 
{%endhighlight %}



<H1>Question 9 - discovering Gitkatas</H1>


{%highlight shell%}
$ git clone https://github.com/praqma-training/gitkatas
$ cd gitkatas
$ ls
$ cd basic-commits
$ ls
$ gedit README.md&
$ . ./setup.sh
{%endhighlight%}


<H1>Question 10 - testing merge conflict, using Gitkatas</H1>

example of a gitkatas exercise on resolving conflicts:

{%highlight shell%}
$ cd gitkatas/merge-conflict
$ gedit README.md &
$ . ./setup.sh
$ cd exercise
{%endhighlight%}

You have to merge branch "merge-conflict-branch1" to branch "master", but there is a conflict on file...

You must solve it to get this result:
{%highlight shell%}
arnaud@arn:~/gitkatas/merge-conflict/exercise$ git lol 
*   f68e64f (HEAD -> master) merged
|\  
| * 71cae00 (merge-conflict-branch1) add relevant fact
* | 0fcabd5 add indispensable truth!
|/  
* ed4face Add content to greeting.txt
* 806d0b8 Add file greeting.txt
{%endhighlight%}


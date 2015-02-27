### what is gitsh?

`gitsh` attempts to turn tricky git-level operations into understandable file-system operations,
by adopting the conceit that a git repository is actually many parallel universes of data laid
down in the same location, and that `git` is an imperfect portal and time machine for manipulating
those universes and their interrelationships.

Not too surprisingly, `gitsh` thinks it can be these things better, by masquerading as a shell.

### what can I do with gitsh?

Currently, the things that gitsh does easier than git:

 1. Showing files on branches

```shell
gitsh$ cat branch/file
$ git show branch:file
```

 2. Diffing files between branches

```shell
gitsh$ diff branch1/file branch2/file
bash$ diff -u <(git show branch1:file) <(git show branch2:file)
```

 3. Show current branch

```shell
gitsh$ pwd
bash$ git branch | grep ^\*
bash$ git rev-parse --abbrev-ref HEAD
```

|topic|`git` command|`gitsh` equivalent|`gitsh` notes|
|-----|-------------|------------------|-------------|
|**working with branches**||||
|listing branches|git branch|`ls`|- branch folding not implemented yet|
||||- `/bin/ls` incorporation not implemented yet|
|making branches|`git checkout -b foo-name`|`mkdir foo-name`|- can't specify branch base yet|
||||- currently leaves you on the new branch, contrary to /bin/mkdir|
|removing branches|`git branch -D foo-name`|`rmdir foo-name`|- `rmdir` deliberately useless without a magic env setting|
|checkout a branch|`git checkout foo-name`|`cd foo-name`||
|list current branch|`git rev-parse --abbrev-ref HEAD`|`pwd`||
|**working with files between repos||||
|show file on a branch|`git show branch:file`|`cat branch/file`|- doesn't use pager|
|diff specific files between branches|`diff -u <(git show branch1/file) <(git show branch2/file)`|`diff branch1/file branch2/file`|- assumes you want a unified diff|
|**setting up your environment**||||
|list aliases|-|`alias`||
|creating aliases|`git config --global alias.cd checkout`|`alias c=checkout`||
|removing aliases|-|`unalias c`||
|show current environment|-|`env`|shows internal environment variables|
|set environment variable|-|`export GITSH_DEBUG=1`||
|show command history|`history`|`history`||
|**debugging**||||
|show last command status|`echo $?`|`echo $?`||
|echo an arbitrary string|`echo foo`|`echo foo`||
|examine the current shell|-|`examine`||
|restart with debugging|-|`restart --debug`||
|**scripting**||||
|run a known command|`git ...`|`gitsh --command ...`||

### What's next

 1. I'm not happy that I don't have proper readline support, so that'll be the next push before adding/fixing commands.

 2. I'd like to get the current commands to interact with the shell equivalents, so gitsh starts to feel more like a shell. For example:

```shell
1 gitsh$ pwd
  master
2 gitsh$ ls
  * master
    topic/
    spike/
3 gitsh$ cd topic<tab>
  topic/ticket-1-add-support-for-wombats
  topic/ticket-2-add-support-for-penguins
  topic/defect-7-spay-all-mammals
4 gitsh$ cd topic/def<tab>
  Switched to branch 'topic/defect-7-spay-all-mammals'
5 gitsh$ ls -alF
  # ... directory listing shown
```

And perhaps at step 2 above, what I should really be seeing is just the output of /bin/ls, and maybe I should have these syntaxes available to me:

```shell
1 gitsh$ ls
  # file listing only
2 gitsh$ ls -al
  # formatted file listing
2 gitsh$ ls ++gitsh -al
  # branch listing
  # formatted file listing
3 gitsh$ ls --gitsh
  # branch listing only
```

Mostly, I'd like the same now-intuitive shell commands that I have at my disposal to be easily extended to include data from my git repo. I'm just not sure what "intuitive" means in the context of a combined git+shell.

 3. Making inter-branch activities work in terms of `mv` and `cp` seems pretty straightforward, these seem like `merge` and `cherry-pick` operations.

  4. Time-based commands are still an unexplored avenue. Expressing a rebase in terms of intuitive filesystem operations is not immediately apparent to me. So far the clearest idea I have of expressing something simple (such as, "Let's step backward in this branch's history,") involves a interactable curses status bar that can be used to move forward and backward through a repo. And you might drop marks on files, directories, SHAs, etc. to allow moving things around in time easily.

A concrete example might be useful. This happened a few days ago, which is what got me writing `gitsh` in the first place. Jaimie had a large topic branch checked out; unfortunately our development of a particular feature is set up such that we're maintaining a long-standing topic branch as a secondary `master`. So the correct workflow is:

```shell
1. pull the mega topic branch
2. checkout a new branch from HEAD of mega topic
3. do work, commit, push
4. issue a pull request between branch at 2 and mega topic.
5. code review
6. merge the pull request
```

Unfortunately, what Jaimie had done was to skip step 2, and stopped at step 3 before pushing. My fix was:

```shell
1. git pull --rebase
2. git checkout -b temp, to save a pointer to this version of this local mega-branch
3. git checkout topic/mega-branch
4. git reset --hard origin/topic/mega-branch
5. git checkout -b topic/mega-branch/branch-jaimie-wanted
6. cherry pick commits from temp (I didn't want an extra merge from temp)
7. git push
```

Now we could fall into the workflow at step 4 above.

What it seems like we did at step 2, when creating `temp`, and then at step 6, is that we performed a symlinking operation: except with time being the entity bridged rather than the filesystem.

If this whole workflow were not involving git, but were involving the filesystem, then the concrete steps would have been much simpler to understand and express.

So I dunno. But it's an experiment.

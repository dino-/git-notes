# Git Basics


## Examining changed files

    $ git status
    $ git diff
    $ git diff --cached  # Include staged items
    $ git diff --staged  # Include staged items, synonym for --cached


## Getting changes from a remote

Normally it's a simple `git pull origin master` or whatever branch you want. If
the upstream was set, `git pull` will suffice.

To pull from another non-server repo on another system on the network, while in
the local repo directory:

    $ git pull user@host:/path/to/repo

This is useful in cases where changes were made by a user other than the one
who can push back to a server. Or collaborating with people and sharing work
without going through the server.


### Sending some (but not all) commits somewhere in a bundle

In the source repo

    $ git bundle create ../foo.bundle src-branchname

This will get all unpushed commits in this branch. Now, send the file
`foo.bundle` somewhere.

In the destination repo, you can verify this looks ok to git before proceeding

    $ git bundle verify ../foo.bundle

And then pull it much like any other remote

    $ git pull ../foo.bundle dest-branchname[:src-branchname]

The bundle can be used to create a new clone too, which is useful if you need
to understand more about what's in it

    $ git clone foo.bundle new-dir-name


## Committing changes

In git, it's a two-step process of adding changes to the index and then committing

    $ git add -p   # Interactive prompting, like darcs

And then don't forget the commit

    $ git commit

In this form, an implicit add happens in git to add everything to the index

    $ git commit -a

You can also specify the short desc on the command line, but I personally dislike doing it this way

    $ git commit -a -m MSG


### Correcting something left out of a commit

    $ git commit --amend

ONLY works in git if amending *last* commit

Also, as with ANY source control, NEVER do this to a commit that has been pushed anywhere. Very bad!


## Viewing the commit log

    $ git log
    $ git log --name-only
    $ git log --stat           # Show details about individual files
    $ git log -1               # Show only the last commit
    $ git log --max-count=1    # Same as `git log -1`

A useful branch graph view

    $ git log --oneline --graph --branches

That one is so useful, add an alias for it to your `~/.gitconfig` alias block

    [alias]
      ... other aliases you may have ...
      lg = log --oneline --graph --branches

Add `--all` to include the stash refs

    $ git log --oneline --graph --branches --all
    $ git lg --all


### More git log variations

    $ git log --pretty=oneline
    $ git log --pretty=format:"%h %ad | %s%d [%an]" --date=iso

Can put this last one in .gitconfig [alias] block

    [alias]
      ... other aliases you may have ...
      hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --date=iso

Additional switches also work to see more things

    $ git hist --branches --all

List all authors to have committed to this repo

    $ git log --all --format='%cN <%cE>' | sort -u


### git show

View a file in a different branch without changing branches

    $ git show BRANCH:FILE

View a file at a different ref

    $ git show REF:FILE

examples

    $ git show HEAD^:foo/bar/baz    # Prior commit
    $ git show 3d8a55f:foo/bar/baz  # Specific commit


## Restoring working copy files to last committed state

**WARNING** This section is sketchy, needs more research

    $ git reset --soft   # Preserves working copy changes
    $ git reset          # Also preserves working copy, --soft is default
    $ git reset --hard   # Destroys working copy changes
    $ git checkout FILE  # I think also destroys working copy changes

Don't confuse this with `git revert` which is for committing a
prior version, to undo something more serious.

A lot of this reset/checkout business I find confusing in Git. Often
you can use the `git status` output to coach you on what to do to
undo something that's in-progress (restore working copy, un-stage
something, etc..)

Sometimes files end up staged but you want ALL of them to be unstaged (and you
DO NOT WANT TO LOSE THE CHANGES), AND there are too many to list individually.
To do this, simply:

    $ git reset

If your local branch has diverged from the remote and you want to reset local
to the remote:

    $ git switch branchname
    $ git reset --hard someremote/branchname


## Reverting

To revert multiple commits as a unit, do them separately like this

    $ git revert --no-commit D
    $ git revert --no-commit C
    $ git revert --no-commit B
    $ git commit -m "the commit message for all of them"


## Going back to a prior state of the repo

**WARNING** This section is sketchy, needs more research

    $ git reset --hard ID
    $ git reset --hard TAG

After that last one, master branch now points to TAG, but even
after this, later commits are still in the repo. Not really sure
how this works.


## Removing untracked files

To recursively remove all untracked files and directories

    $ git clean -df


## Working with remotes

    $ git remote -v               # Show all remotes


## Tagging

To make a tag, use -a or you get an unannotated tag. This is more
like the tag we understand from other RCS systems.

    $ git tag -a TAG  # opens an editor for MSG
    $ git tag -a TAG -m 'MSG'

List all tags

    $ git tag
    $ git tag -n  # Including the message

Show the commit that tag TAG is associated with

    $ git show TAG

You can then look for this commit in git log output

    $ git log

Point to a tag to build that rev

    $ git checkout TAG

You MUST push tags separately from other changes:

    $ git push
    $ git push --tags

Beware, no tag means no reference, so things can be garbage collected
at some point (what does this mean?) So, the procedure for really
removing from master:

    $ git tag oops
    $ git reset --hard EARLIER_TAG

Then, if you do want the "oops" tagged commit to be GCd away:

    $ git tag -d oops

Removing a tag

    $ git tag -d release01                 # The local tag
    $ git push origin --delete release01   # The remote tag

If the tags you have in your local clone get very out-of-date with what's on
the remote, clean them up like this:

    $ git tag -l | xargs git tag -d
    $ git fetch --tags

An alternative way to do this if there aren't very many tags

    $ git tag -d $(git tag -l) && git fetch -t


## Git Branching


### Making branches

Create a new local branch

    $ git checkout -b NEWBRANCH

or using the branch command

    $ git branch NEWBRANCH
    $ git checkout NEWBRANCH

**WARNING:** Never, EVER name a branch the same name as a tag. Turns
out, branches and tags are really almost the same kind of thing
(a ref I think)

List branches

    $ git branch      # local branches only
    $ git branch -r   # remote branches only
    $ git branch -a   # all local and remote
    $ git branch -a --sort=committerdate
        # Ordered by most recent commit in the branch

Switching to another (possibly remote) branch

    $ git checkout BRANCH

Push a local branch to the remote

    $ git push -u/--set-upstream origin LOCAL_BRANCH

Delete a branch completely, from both local and remote, do both of these things

    $ git branch -d LOCAL_BRANCH
    $ git push origin --delete REMOTE_BRANCH

Then, if you have clones with stale remote branches in their list, you may have to prune them:

    $ git fetch -p/--prune origin

View which remote branch each of the local branches are tracking

    $ git branch -vv


### Creating an orphan branch

These instructions are incomplete, need to try them out some more.
Also, see [this](https://docwhat.org/git-tip-empty-branch/)

Clone a copy that will be where we create the new orphan dist branch

    $ git clone REPO

In that new directory, create a new orphan branch

    $ cd PROJECT
    $ git checkout --orphan BRANCHNAME

And then remove everything (yes, I'm serious) from THE NEW CLONE. Be careful of your location

    $ cd PROJECT  # if necessary
    $ rm -rf ./*
    $ git add -A
    $ git add .gitignore

Commit this removal of everything.

    $ git commit

Clone a copy for working from the master branch

    $ cd ..
    $ git clone REPO PROJECT-master

Install the deployment files into the orphan

    $ cd PROJECT-master
    $ ./util/install.hs ... -p ../PROJECT

In the orphan again, add/commit those new files. The new clone

    $ cd ../PROJECT
    $ git add -A
    $ git commit

Push that local-only branch to the remote for the first time

    $ git push --set-upstream origin BRANCHNAME


### Renaming the default branch

First, change the local repository

    $ git branch -m master main
    $ git push -u origin main

Now remove the `master` branch on the remote

    $ git push origin --delete master

If this fails for github with a message like

     ! [remote rejected] master (refusing to delete the current branch: refs/heads/master)
    error: failed to push some refs to 'foo.com:bar-/baz.git

You will first need to change the default branch. In the repository web page:
Settings -> Branches -> Default branch section. Click the two-arrows icon to
the right of the current default branch and select the branch to be the new
default. You can then delete the other branch from here or issue the above
command again on the local system.


### pulling TO another branch

Suppose you have a branch `feature` that was made from `main`. You have uncommitted changes in `feature` but you want to bring in new changes to `main` by the team. It goes like this:

    $ git stash push
    $ git switch main
    $ git pull
    $ git switch feature
    $ git stash pop

This is of course a pain in the ass. But it can be done in one shot and without
disrupting `feature` with fetch. Again, on branch `feature` with the
uncommitted changes):

    $ git fetch origin main:main

FIXME May still need a rebase in here somewhere, need to test this

    $ git pull origin main:main --rebase


### Recovering branches and stashes from a damaged repo

I've had a a nix + cabal Haskell repo get into a state where I can't `cabal
build` any longer. And once this happened with corruption to git commit hooks
so that commits were no longer possible. Whatever the reason, suppose you can
no longer use the clone you have and must clone again into a new directory. The
problem is, what about local-only branches and stashes in the broken clone?
Here are some tips for moving them over.

Make your new clone

    $ git clone ... newclone
    $ cd newclone
    $ git add remote broken ../oldclone  # relative dirs are allowed here
    $ git fetch broken

You can now switch to the branches in broken to use them in newclone

    $ git switch branch-in-broken

Or, if there's a conflict with branch names

    $ git checkout --track broken/branch-in-broken

To recover stashes from the broken repo, you can use the tedious process of
patch files.

    $ cd ../oldclone
    $ git stash show -p stash@{..} > patchfile
    $ cd ../newclone
    $ git switch somebranch
    $ git apply ../oldclone/patchfile
    $ git stash push -m '...'


## Merging

### Aborting a merge with conflicts:

    $ git merge --abort

or, if your git is old

    $ git reset --merge

### dry-run

    $ git merge --no-commit --no-ff


## Git on the Server


### Get a solo git repo ready for sharing on your own server

There are many ways to set up shared repos on your own server. The way I've
been doing it is this:

- Create a user `git` whose default group is also `git` on the system. This
  user will own the entire filesystem tree directly above the repositories.
- Any users on the local system where this is happening will need to be added
  to the `git` group
- For users not on the local system, you'll want to add public ssh keys to the
  `git` user's `authorized_keys` file

Now, to create a new repository from another system

    $ cd NEWREPO
    $ ssh git@server git init --bare --shared=group /repos/path/NEWREPO.git
    $ ssh git@server chown -R git:git /repos/path/NEWREPO.git
    $ git remote add origin git@server:/repos/path/NEWREPO.git
    $ git push -u origin master

Not sure if this is enough to get all tags and things, probably not


### Copy a git repo from one server to another

This will get EVERYTHING, all branches, all tags, refs.

    $ git clone --mirror git@OLDserver:/repos/path/OLDproject.git
    $ cd OLDproject.git
    $ ssh git@NEWserver git init --bare /repos/path/NEWproject.git
    $ git remote add new git@NEWserver:/repos/path/NEWproject.git
    $ git push --mirror new
    $ cd .. && rm -rf oldproject.git


### Make a backup of a repo and keep it up-to-date

First time

    $ git clone --mirror git@server:/repos/path/project.git

To update

    $ cd project.git
    $ git remote update

To update without changing the current directory

    $ git --git-dir project.git remote update

Also useful, export the entire repo as a single file. This can be done with any
clone, not just a mirror clone.

    $ cd project.git
    $ git bundle create ../project.bundle --all


## Git Tools


### Stashing

Apply a single file from a stash. The FILENAME working will be overwritten by
what's in the stash.

    $ git checkout [<STASHID>] -- <FILENAME>
    $ git checkout stash@{0} -- <FILENAME>

To list the stashes with the date they were made

    $ git stash list --date=local


### Rewriting History


Had a situation like this:

    a - b - c (master)
     \
      d - e (br1)

We want to merge branch br1's commits into master but that branch
is missing commit b. This will default to creating a merge commit
if we simply `checkout master; git merge br1` Nobody like this,
apparently.

One way to fix it is to rebase in br1:

    $ git checkout br1
    $ git pull --rebase origin master

You'll then have this:

    a - b - c (master)
     \
      b - c - d - e (br1)

And then this will create a clean fast-forward merge:

    $ git checkout master
    $ git merge br1


### rev-list

    $ git rev-list HEAD --count   # Count of commits (I think just the checked-out branch)


### Removing all record that a file ever happened

    $ git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch PATH/TO/YOUR/FILE' --prune-empty --tag-name-filter cat -- --all

May want to add the file to `.gitignore` now. When you're sure everything is cool, do this:

    $ git push origin --force --tags

And, as with all rebase-y tasks, be careful if other people have used the repo.


### cherry pick

Cherry picking a range inclusive. If you want to cherry pick all commits
between an older commit and a newer one and include the starting one, it's not
obvious. Need the `^` like this:

    $ git cherry-pick A^..B


## Customizing Git


### Lines with trailing whitespace

Git by default gets upset about lines with trailing whitespace

To make this go away, you can make this one-time change to your config:

    $ git config --global core.whitespace '-trailing-space'


## Exotic, advanced stuff you probably won't need to do often


### Convert a darcs repository to git one-time

    $ cd drepo
    $ git init ../grepo
    $ darcs convert export | (cd ../grepo && git fast-import)
    $ cd ../grepo
    $ git checkout

May have to fix executable scripts or files after this.


### Tracking an empty directory with git:

For git, it's apparently still the 1990s. It doesn't track
directories at all, and so empty directories are a pain in the ass
just as with CVS.

To track and preserve an empty directory, you have to fake it with
a .gitignore file like this:

Make your directory, let's call it foo

Make a file foo/.gitignore containing this:

    # Ignore everything in this directory
    *
    # Except this file
    !.gitignore

Commit that foo/.gitignore and you're all set


### Changing the author info on an entire branch or repo

This will rewrite every commit, keeping everything including the date/time
stamps and only changing the author info. They will all have new hashes, so
beware doing this to branches that have been pushed.

For all commits up to and including the root

    $ git rebase --root --exec 'git commit --amend --no-edit --author "First Last <foo@bar.baz>"'

Or from some point onwards

    $ git rebase <after-this-commit> --exec 'git commit --amend --no-edit --author "First Last <foo@bar.baz>"'


### Grafting older changes onto a newer repo.

**WARNING** This section is VERY out-of-date, needs research/rewrite

Somebody didn't clone or fast-import! You've got this situation:

    dev/
       proj-orig/     # This has all the original work on proj
       proj-new/      # Somebody copied proj-orig without any source control info and started committing

Now you want to graft the earlier, 'orphaned' history onto these later commits in proj-new. Making a single repository with the combined, whole history.

So, how to fix this?

Before you begin, make sure all team members have committed and pushed anything they want to keep. They will be losing their cloned dirs as a result of this. Let's begin:

Let's cd to a dir above your repo dir, in its parent, dev/ say

Make a backup, just because we're paranoid

    $ git clone proj-new backup

Make the clone we'll fix in

    $ git clone proj-new proj-fix

Let's assume that the newer patches are in branch 'master'

Make an empty branch to bring in the older patches

    $ cd proj-fix
    $ git symbolic-ref HEAD refs/heads/older
    $ rm .git/index
    $ git clean -fdx

   ([This symbolic-ref business from this page](http://book.git-scm.com/5_creating_new_empty_branches.html))

Pull patches into this empty branch from your earlier repo

    $ git pull proj-orig

Get the commit id of the older head up, we'll need it shortly

    $ git log -1

Switch to the master branch

    $ git checkout master

Perform the graft of older onto newer

    $ git filter-branch --parent-filter 'sed "s/^\$/-p <hex commit id from above>/"' HEAD

The master branch should now look right, with the older patches before the newer ones.

This work can now be pushed back to the remote, must be forced:

    $ git push -f

Instruct all team members to discard their cloned dirs of this repo and clone
again. The history has been rewritten. This is very important, **do not blow
off communicating this to the team!**


## About this document

Author: Dino Morelli <dino@ui3.info>  
Date: 2023-Dec-21

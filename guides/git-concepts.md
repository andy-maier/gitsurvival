Git Concepts
============

Locations and data flow for some basic Git subcommands
------------------------------------------------------

    work directory    staging area     local repository     remote repository

         <-----------------------------------  <----- clone -------

         <----------- checkout --------------

         ----- add, rm --->

                             --- commit ---->

                                               ------- push ------>

         <-----------------------------------  <------ pull -------

                                               <------ fetch ------

                                            <--->
                                            merge

                                            <--->
                                            rebase

Note that most commands shown in the diagram above work on branches, so
there is a lot that is not shown in the diagram, but nevertheless it
gives a first idea about how Git operates.

The local repository and the staging area are stored in the ".git"
subdirectory under the work directory.

Most of the time, the work directory is associated with a branch from the
local repository; if so then that branch is called the *current branch*.

The HEAD label in the local repository points to the last commit that the
working directory is associated with. Usually that is the node that is the
tip of the current branch.

Commits
-------

A commit holds informations about the differences in the files, the date and
author and references the previous commit. Each commit has a unique identifier
called *commit id*.

In Git commands where commit ids can be specified, it is usually sufficient to
specify its first 7 characters. The Git command will fail if that is not
uniqe throughout the repository. **[TBD: Verify whether Git commands will fail
if the short commit ID is not unique throughout the repository.]**

In the commit tree of a repository, a commit is also called a *node*, and
its preceding commit is called its "parent node".

Branches
--------

A *branch* is a label that points to a commit.

Because each commit knows its preceding commit, the term "branch" is
sometimes also used as a term for a chain/line/history of commits.

In most cases, Git documentation uses the term "branch" for the label.
For example, "deleting a branch" via `git branch -d` deletes the branch label,
but not the commit that branch points to.

Commits that are not referenced by anything (branches, labels or their
child commits), are subject to Git garbage collection. So a chain of commits
whose tip is referenced by a branch will go away at some point, if that branch
(label) is deleted.

One tends to imagine such a line of commits as always having a "base".
However, that is a fragile notion, because Git does not remember on which
commit a new branch was based on, when it was created.

For example, walking the current tree structure from the branch tip back to
the next ancestor that has more than one child does not necessarily lead one
to the original base: New branches may have been based upon some commit
in between the original base and the current tip of the branch in question,
or the chain of commits that existed when a branch was started at its
original base ceases to exist for some reason.

See these discussions for more details on the "base" of a branch:
* [stackoverflow: Where does a Git branch start and what is its length?](http://stackoverflow.com/questions/17581026/where-does-a-git-branch-start-and-what-is-its-length)
* [stackoverflow: Finding a branch point with Git?](http://stackoverflow.com/questions/1527234/finding-a-branch-point-with-git)

In any case, the repository does not know where a branch *once started off*.
It just knows the *current tree structure* of commits.

If a change is committed on the current branch, the branch label is moved
to the new commit. Tags are also labels that point to commits, but tags will
not be moved foward by commits, even if the tag references the tip of a chain
of commits.

Branch names, like any labels, can have slash characters in their name. You
can use that to organize your local branches into groups (e.g. `ideas/xxx`).

To list the branches in the local repository:

    $ git branch

This lists only the so called *local branches*. More on remote branches will
be discussed in section [Remote branches](Remote branches), below.

Understanding the commit tree
-----------------------------

An easy way to look at the commits is the commit log. It is important to
understand that the commit log is always shown only for a particular branch.

This command lists the commits for the current (=checked out) branch:

    $ git log

This command lists the commits for a particular branch in the local repo,
without a need to change what is currently checked out:

    $ git log {branch}

This command lists the chain of commits between of the chain l
The tree structure of the local repo can be shown with a number of more
elaborated commands. For example, 


Remotes
-------

A *remote* is a definition on the local repository that specifies a remote
repository. A remote has a name (e.g. "origin") and specifies the URL of a
remote repository.

Note that Git repository URLs do not always start with "http" or "https";
they may even be local directory paths. For reference, see the
[GIT URLs section of git-clone](https://www.kernel.org/pub/software/scm/git/docs/git-clone.html#URLS).

So the term "remote" does not imply that the repository is on a different
host; it just means that it is not the current local repository.

The standard remote named "origin" specifies where the local repository was
cloned from, and is normally used as the default remote for most interactions
with remote repositories.

The default remote for a remote-tracking branch "{branch}" can be set via the
"branch.{branch}.remote" configuration variable. For example, by default,
the "branch.master.remote" configuration variable is set to "origin", which
causes the remote branch "origin/master" to be pushed to the (local branch)
"master" in the remote repo identified by the "origin" remote.

A local repository can have more than one remote defined. In scenarios with
forked repos, or with Gerrit, this is actually a typical case.

To display the remotes of the local repository:

    $ git remote -v
    origin  {remote-repo-url} (fetch)
    origin  {remote-repo-url} (push)

To add a remote to the local repository:

    $ git remote add {remote} {remote-url}

To remove a remote from the local repository:

    $ git remote remove {remote}

Removing a remote automatically also removes any configuration variables that
use that remote in their name or their value
(e.g. "remote.{remote}.\*", "branch.{branch}.remote={remote}",
and the corresponding other "branch.{branch}.\*" variables).

Adding a remote (e.g. the remote that was just removed), does not
automatically create all of these variables again. This normally only plays
a role when changing the "origin" remote, but `git push` will then complain
about a missing upstream definition and show the right command to re-establish
these variables.

Some examples of repository URLs ({remote-url}):

- SSH-based URL for GitHub. Most remote Git sites allow a public key to be
  uploaded in order to avoid having to enter userid and password on every
  command that works with the remote repository. That only works with
  SSH-based URLs:

      ssh://git@github.com/{user-id}/{project}.git
      ssh://git@github.com/{org-id}/{project}.git

  or:

      git@github.com/{user-id}/{project}.git
      git@github.com/{org-id}/{project}.git

- SSH-based URL for a privately hosted repository, again with public key
  stored:

      ssh://git@inovadevelopment.com:8026/home/git/cmpi_headers_2_1_work

  or:

      git@inovadevelopment.com:8026/home/git/cmpi_headers_2_1_work

- file system based URLs

      /data/mygits/work/my.git

Remote branches
---------------

A *remote branch* is a branch in a (local) repository that is maintained as
an exact copy of a (local) branch in a remote repository.

To list remote branches in the local repository:

    $ git branch -r

To list both local branches and remote branches in the local repository:

    $ git branch -a

Note that in most Git versions, the second command lists the remote branches
with an additional `remotes/` prefix. That is just the way this command
displays the remote branches; the `remotes/` prefix is not part of the remote
branch name.

Remote branches are never to be modified by commits; they exist as a
kind of cache of the corresponding branch in the remote repository, so that
complex operations such as merging or rebasing can be performed within the
local repository without having to access the remote repository.

The set of (local) branches in a remote repository is listed in the output of:

    $ git remote show {remote}

One point of confusion with Git is that it uses the syntax `{remote}/{branch}`
mostly for remote branches in the local repository, but sometimes also
as a shorthand for the upstream branch `{branch}` in remote repository
`{remote}`.

Upstream branches
-----------------

A local branch can have an *upstream branch* defined for it. The upstream
branch is also called a *tracking branch* for the local branch, or a
*remote-tracking branch*.

The purpose of an upstream branch is for Git to automatically know which
branch in which remote repository the local branch is connected with.
This allows Git to push to and fetch from the remote repository without
specifying the upstream branch on each Git command. Also, Git uses the
upstream branch as a comparison basis, for example when `git status` shows
the current branch as being ahead and behind commits on its upstream branch.

The upstream branch, being in a remote repository, will cause that branch
to be present in the local repository as a remote branch.

The upstream branch does not necessarily have to have the same name as the
local branch. The remote branch always has the same name as the upstream
branch in the remote repo (with an added `{remote}/`).

Here is an example for a local topic branch `topic1` that is named `mytopic1`
in the remote repo, as well as the data flow for some important Git
commands between them:

    local repository                                   origin
                                                   (remote repository)

    topic1           ---------- push -----------+
    (local           <--------- pull ---------+ |
     branch)         <--+                     | |
                        |                     | |
                        | rebase/merge        | |
                        |                     | +-->   mytopic1
    origin/mytopic1  ---+                     +-----   (upstream
    (remote branch)  <--------- fetch --------------    branch)

Fetch and merge (or fetch and rebase, if configured that way) is also combined
in the `git pull` command.

The upstream branch of a local branch is authoritatively defined in config
variables of the local repository:

    $ git config -l |grep "^branch\."
    branch.topic1.remote=origin
    branch.topic1.merge=refs/heads/mytopic1
    branch.master.remote=origin
    branch.master.merge=refs/heads/master

In this local repository, two upstream branches are defined:

- branch `mytopic1` on remote repository `origin`
- branch `master` on remote repository `origin`

These upstream branches have corresponding remote branches in the local
repository:

    $ git branch -a
      topic1                   # has upstream branch (see config variables)
      topic2                   # does not have an upstream branch
    * master                   # has upstream branch (see config variables)
      remotes/origin/mytopic1  # remote branch for branch mytopic1 on remote repo origin
      remotes/origin/master    # remote branch for branch master on remote repo origin

Given that they may be named differently, the relationship between upstream
branches and local branches cannot always be determined from their names alone.
The following command provides useful information about a remote repository,
showing the relationships between local and upstream branches.

In its output section "Remote branch", the command shows all branches in the
remote repository along with a status whether they are tracked and if so their
update status.

In the output sections "Local ...", the command shows how the upstream branches
are connected with the local repository, for each data flow direction:

    $ git remote show {remote}

or, for the example above:

    $ git remote show origin
    * remote origin
      Fetch URL: {origin-repo-url}
      Push  URL: {origin-repo-url}
      HEAD branch: master
      Remote branch:
        master tracked
        theirtopic3 new (next fetch will store in remotes/origin)
        topic1 tracked
      Local branch configured for 'git pull':
        master merges with remote master
        topic1 merges with remote mytopic1
      Local ref configured for 'git push':
        master pushes to master (up to date)
        topic1 pushes to mytopic1 (up to date)

If anything needs to be fixed in the remote-tracking metadata, this can be
done by changing the corresponding config variables; they are the
authoritative source for the remote-tracking information. Such changes are
best done by editing the repo's config file:

    $ git config -e

The following command is another way to display the tracking information
for a particular remote repository:

    $ git fetch {remote} -v --dry-run

or, for the example above:

    $ git fetch origin -v --dry-run
    From {origin-repo-url}
     = [up to date]      master -> origin/master
     = [up to date]      topic1 -> origin/mytopic1

Branch `topic2` is not listed, because it does not have an upstream branch
defined in this example.
Branch `theirtopic3` is not listed, because it does not have a local branch
in the local repository.

The remote branches in the local repository that can be updated for a particular
remote to match the remote repository, with this command:

    $ git fetch [{remote}]

Normally, the default remote is "origin" (but this can be configured to be
differently).

The current branch in the local repository can then be updated from these
remote branches, with:

    $ git rebase

Differences between local and remote repository
-----------------------------------------------

The local repository is a complete repository, with branches, tags, history,
etc.

However, the set of commits (and hence, the node structure) in the local
repository may (and usually will) differ from the remote repository.

For example, branches may just exist locally and may never be pushed to the
remote repository.

Or the remote repository may be worked by multiple people, so it will
contain branches you are not interested in, and that are never being fetched
into your local repository.

Each updating of either the local or remote repository is best thought of
as an operation on a branch; it is far away from being a sync operation between
the repositories (e.g. such as "svn update").

Even within corresponding branches, the chain of commits may be different
between local repo and remote repo, for example when other work is merged
into that branch of the remote repo.

Pushing to a remote repository
------------------------------

The following commands push commits from the local repository to a remote
repository. When that happens, they are not just copied over there, but merged
into a particular branch. That can lead to merge conflicts.

* Create a new upstream branch on a particular remote repository and push
  the current branch to that upstream branch, merging it there:

      $ git push -u {remote} HEAD:{branch}

  x

* Push the current branch to its upstream branch, merging it there:

      $ git push

  Note that this does not push any tags.

* To push tags in the local repo to a particular remote repo:

      $ git push --tags {remote}

  This pushes all changes on tags, i.e. additions and deletions.

* To delete a branch in a remote repository:

      $ git push {remote} :{branch}

  This essentially pushes "no branch" to that branch in the remote repo,
  resulting in a deletion of the branch label.


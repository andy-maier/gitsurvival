Git Tips and Tricks
===================

Git help
--------

To display the most common Git subcommands:

    $ git

To display a complete list of all Git subcommands available in your
installation:

    $ git help -a

To display a list of Git concepts for which help is available in your
installation:

    $ git help -g

To invoke the man page for a specific Git subcommand:

    $ git help {subcommand}

A great help for understanding Git man pages is the Git glossary:

    $ git help glossary

Links:

* [Everyday Git](ftp://www.kernel.org/pub/software/scm/git/docs/v2.1.2/everyday.html)

Setting up a local repo
-----------------------

To create a local copy of a remote repository, issue:

    $ git clone ssh://git@github.com/{project}.git

which creates the repository in ./{project}.git, or issue:

    $ git clone ssh://git@github.com/{project}.git {workdir}

specifying the new work directory {workdir}.

Depending on the project, configuration variables may be needed to make the
work easier.

To display the configuration variables of the local repository:

    $ git config -l

Status
------

Display the status of the work directory against the local repository:

    $ git status

Or in a more brief variant:

    $ git status -s

The first column indicates the status in the current branch of the local
repository, and the second column indicates the status in the work directory.

There is no way to display the status of the local repository against
the remote repository.

List differences
----------------

List differences (even when committed) between work directory and the "master"
branch of the local repository:

    $ git diff --name-status master

This will ignore any untracked files (they can be shown with `git status`).

This should show only the file changes that are supposed to go into git
review.

"Closing" a branch
------------------

Sometimes, branches in the local repository are no longer needed, either
because they are done and their changes have been pushed, or because the work
is abandoned. Mercurial has a concept of closing a branch. Git has no exact
equivalent to that, but a similar effect can be achieved using this
"archiving" approach.

The approach is to create a tag on the branch tip, and to remove the branch
label from that node. This causes the branch to no longer be listed in
`git branch`, but still allows to find and/or restore it via the tag, should
it ever be needed again.

To "archive" (i.e. "close") a branch:

    $ git tag archive/{branch} {branch}
    $ git branch -d {branch}

Using "archive/" in the tag name is purely a convention and not a requirement.

If needed, the branch can be restored later by checking out the tag, and
recreating the branch:

    $ git checkout archive/{branch}
    $ git checkout -b {new-branch}

The second checkout command creates a new branch for the content of the work
directory, and then checks it out again. This second checkout does not change
the work directory content (because it had been updated by the first checkout
already), but is still needed to mark the checkout as being from the new
branch.

The new branch name "{new-branch}" can be the same as the old name
"{branch}", if so desired.

If the branch has previously been published to a remote repository, the
following commands archive the branch on the remote repository as well:

    $ git branch -dr {remote}/{branch}   # remove branch label in local repo
    $ git push --prune {remote}          # propagate any branch removals to remote repo
    $ git push --tags {remote}           # propagate new tags to remote repo

Renaming a branch
-----------------

Renaming a remote branch in a local repo:

    $ git branch -m {remote}/{old-branch} {remote}/{new-branch}

**TBD: For remote-tracking branches, is this change propagated to the remote repo?
[CO: I would never do that, a remote branch is REMOTE and could be checked out by someone else.
what happens if someone has set up {old-branch) as upstream branch for his local branch? ]
[AM: How is the workflow then to rename a branch in a remote repo, and get that change
to arrive at the remote branch in the local repo?]**

Renaming a (local) branch in a remote repo:

    $ git push {remote} {remote}/{old-branch}:refs/heads/{new-branch} :{old-branch}

This actually creates a new branch and deletes the old branch

Show the node tree of the local repository
------------------------------------------

      $ git log --graph --pretty=oneline --abbrev-commit

  The value for the --pretty option controls how much is shown, use "short" or
  "medium" for adding things like author and date.

Show the files that were part of a commit
-----------------------------------------

      $ git show --name-only {commit-id}

      or
   
      $ git show --name-status {commit-id}
       
  Instead of {commit-id}, a branch name can be specified to mean the latest
  commit on that branch. If {commit-id} is nt specified, it defaults to HEAD.

Remove a tag in the remote repository
-------------------------------------

      $ git tag -d {tag}
      $ git push {remote} :refs/tags/{tag}

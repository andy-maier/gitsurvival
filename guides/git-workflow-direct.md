Git workflow when working directly with the original repo
=========================================================

**TODO: This still needs to be rescoped to direct access (it originally was a
copy of the github-fork version).**

Principle
---------

There is a local repo whose origin is the remote repo it has been cloned from.
Work happens on topic branches which get pushed to the remote repo, are
reviewed there and finally merged into master.

So there are two repos involved:

                     original repo (on GitHub, remote name: "origin")
    local repo

The flow of changes is:

    push topic branch: local repo ---> original repo (to publish a contribution)
    merge topic branch into master: on original repo (after review)
    fetch master: original repo ---> local repo (to move our basis forward)

Setting up
----------

1. Clone the original repo locally:

       $ git clone {github-original-repo-url} {workdir}
       $ cd {workdir}

Creating a topic branch
-----------------------

New work starts with creating a topic branch in the local repo. This does not
necessarily have to happen before the first changes to files, but should be
done as soon as it is clear that some changes are needed. Prior local
changes to files are not overwritten by this:

    $ git fetch origin          # Update all remote-tracking branches of
                                # local repo that are connected to remote "origin"

    $ git checkout -b {new-branch} origin/master
                                # Create a new branch based on "origin/master",
                                # set its upstream branch to "origin/master",
                                # and check it out.

    $ git branch -a
    * {new-branch}
      master
      remotes/origin/HEAD -> origin/master
      remotes/origin/master

Note that the new local branch is based on a remote branch. This is perfectly
allowed and avoids additional "overhead" branches.

Propagate changes from original repo to local repo
--------------------------------------------------

This is performed regularly even when not publishing any branches (e.g. every
morning), but most importantly before publishing any branches.

This assumes that the work area is clean (i.e. committed).

1. Update the remote branches from the original repo:

       $ git fetch origin

   This propagates any missing commits from the "master" branch in the
   original repo to the remote-tracking branch "origin/master" in the local
   repo. It also updates any other remote-tracking branches (i.e. our topic
   branches), but there should not be any changes on them.

2. Rebase any local changes in the currently checked out branch to latest
   level of "origin/master"

       $ git rebase

   It is important to use "rebase" instead of "merge" (or even "pull"),
   because we want to base our changes on the most recent changes in the
   remote-tracking branch. Rebase makes sure that first the commits from the
   remote-tracking branch are applied, and on top of that the local commits.
   Merge would order the commits of both sides in their time order, so they
   may end up intermingled.

   **[TBD: I think I had a case where this did not update the current branch.
   The suspicion is that that branch was not a tracking branch. Verify.]**

AM: default for rebase is to rebase against the upstream branch

3. If the "rebase" reports conflicts, resolve them by editing the
   corresponding files, commit the result (to the checked out branch), and
   resume the rebase (the command for resuming is being displayed in the
   original "rebase").

   This is easily done by calling:
   
       $ git mergetool
       
   This will open the preferred merge tool (set up in the Git configuration)
   for each file to be changed, one after the other and when the files are
   saved in that tool and the tool is closed, the file is automatically added
   to the staging area (i.e. ready for commit).

   Once you are finished with this "merging" you can call:
   
       $ git rebase --continue
       
   to resume rebase and Git will commit your changes with the commit message
   that lead to the "conflict".
   
   You can leave the "rebase" mode anytime by:
   
       $ git rebase --abort
       
   which will reset the changes to the state before the rebase started.

Working on a branch
-------------------

This is a cycle of working and committing to the current branch {new-branch}.

Commit often and early. It is not helpful trying to have only large commits
that can be used unchanged for the merge requests, because having more
granular commits provides the basis for cherry-picking (between branches of
the local repo) should that be needed, and for restructuring changes that
end up being unrelated, into separate branches and merge requests.

Publishing a branch to the original repo
----------------------------------------

1. Make sure the work area is clean (committed) and the branch to be published
   ({new-branch}) is checked out.

2. Propagate changes from original repo to local repo, as described further
   up, in order to reduce the chance for conflicts in the merge request that
   is being created:

       $ git fetch origin
       $ git rebase

       # If conflicts are reported: resolve, commit, resume the rebase

3. Optional: Squash the multiple commits of the branch into one commit:

       $ git rebase -i

       # In the editor that comes up, change the "pick" commands of the second
       # commit and of any following commits to "squash".

       # The resulting commit message can be edited in a second invocation of
       # the editor.

   The commit log for branch {new-branch} now shows only one commit on top
   of the last commit from "origin/master", with a commit message as edited.

       $ git log      # The checked out branch is shown by default

   The squashing results in a change to the local repo only, the remote
   repo is changed only afterwards when the branch is pushed.

   The single commits that previously existed in the branch, are now gone. So
   use squashing only the single commits are not expected to be used anymore.

4. Publish the branch to the original repo:

       $ git push origin HEAD:{new-branch}

   This creates branch {new-branch} in the original repo, and makes branch
   {new-branch} in the local repo a remote-tracking branch.

5. Create a merge request for the branch using the GitHub web interface.

Cleaning up after a branch has been merged or finally rejected
--------------------------------------------------------------

Once the owner of the original repo has merged the merge request (with or
without further changes), the local repo should be
cleaned up to indicate that the branch has been processed. In this
workflow, this is done by deleting the branch in the local repo.

The branch remains unchanged in the remote repo, in order not to
break any work derived from that branch.

1. Remove the branch label in the local repo:

       $ git branch -d {branch}

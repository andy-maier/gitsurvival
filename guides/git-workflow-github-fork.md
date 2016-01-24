Git workflow with GitHub forks
==============================

Principle
---------

The intention is to contribute to a GitHub repo to which one does not have
write access. The contributions are done via "Merge Requests" (aka
"Pull Requests") on a fork of the original repo, which is also on GitHub.

So there are three repos involved:

                     original repo (on GitHub, remote name: "origin")
    local repo
                     forked repo (on GitHub, remote name: "myfork")

The flow of changes is:

    local repo ---> forked repo (to publish contributions)
    forked repo ---> original repo (to request merging of a work unit)
    original repo ---> local repo (to get our own merged work units and updates by others)

All work is done on the local repo, using topic branches.

The forked repo is used only to hold those topic branches that become merge
requests, using the same branch names as in the local repo. If the merge
requests are accepted into the original repo at some point, their branches
are archived in the local and forked repo. The `master` branch in the forked
repo is not maintained and ages out.

The original repo's maintainers merge the merge requests from the forked repo
(and other work) into the `master` branch of the original repo, from where it
is propagated into the `origin/master` remote-tracking branch of the local
repo.

Setting up
----------

1. Clone the original repo locally:

       $ git clone {github-original-repo-url} {workdir}
       $ cd {workdir}

2. Fork the GitHub repo using the GitHub web interface. It is important to
   create the forked repo this way, instead of pushing the local repo up into
   a new repo, because that establishes a relationship between these repos on
   GitHub.

3. Create a remote for the forked repo (in the local repo):

       $ git remote add myfork {github-forked-repo-url}

4. Show the remotes and branches (just to check):

       $ git remote -v
       myfork {github-forked-repo-url} (fetch)
       myfork {github-forked-repo-url} (push)
       origin {github-original-repo-url} (fetch)
       origin {github-original-repo-url} (push)

       $ git branch -a
       * master
         remotes/origin/HEAD -> origin/master
         remotes/origin/master

       $ git config -l |grep "^branch\."
       branch.master.remote=origin
       branch.master.merge=refs/heads/master

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
   repo.

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

Publishing a branch to the forked repo
--------------------------------------

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

4. Publish the branch to the forked repo:

       $ git push myfork HEAD:{new-branch}

   This creates branch {new-branch} in the forked repo, and makes branch
   {new-branch} in the local repo a remote-tracking branch.

5. Create a merge request for the branch using the GitHub web interface.

Cleaning up after a branch has been merged or finally rejected
--------------------------------------------------------------

Once the owner of the original repo has merged the merge request (with or
without further changes), the local repo and the forked repo should be
cleaned up to indicate that the branch has been processed. In this
workflow, this is done by archiving the branch in the remote repo (via a tag)
and by deleting it in the local repo.

Note that this removes the branch in the remote repo, so anything that
references that branch now is dangling. This could for example be a local
clone or a fork of someone else you are not aware of. The assumption though is
that your GitHub repo fork is exclusively worked by you, particularly as far
as the topic branches are concerned.

1. Tag the branch in the local repo:

       $ git tag {tag} {branch}

   where for merged branches, `{tag}` is:

       archive/done_{branch}
   
   and for rejected branches, `{tag}` is:

       archive/rejected_{branch}

2. Propagate the tag to the forked repo:

       $ git push --tags myfork

3. Remove the branch label in the local repo:

       $ git branch -d {branch}

4. Remove the branch label in the forked repo:

       $ git push {remote} :{branch}

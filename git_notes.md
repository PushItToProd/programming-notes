# Git notes

## Philosophy

* a project's Git history should provide useful context to its development
* there should be one trunk branch that is _the_ history of the project
* commits should form meaningful records in the project history
  * provide clear, descriptive commit messages
  * minimize extraneous commits on the master branch by rebasing dev branches
    before merge to squash non-functional commits and clarify commit messages

## Keeping Git history clean

* `git amend` - really handy for fixing the last commit when I invariably forget
  to add a file or decide my commit name isn't clear enough

* `git rebase -i` - allows editing, squashing, and removing older commits.
  really useful for when I'm hacking around and introduce a lot of intermediate
  commits that don't provide useful context in the Git history

* `git merge --squash` - squashes all merge changes into a single commit. useful
  to avoid cluttering the git history

* `git pull --rebase` - pulls upstream changes down without introducing a merge
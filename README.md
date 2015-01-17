# Detoul

Ramon Snir (c) 2015 [MIT License]

A declarative tool for creating integration branches in git

### Why?

I found that during development of a version, people wish to test if their code works with other team members' changes. My team members often had issues with the integration branches, merging the integration branches into their feature branches (and then forcing us to start slowly rebasing the feature branches, to take out commits that were meant for a future version but still needed testing).

### How?

(further explanation will come in the future, and of course with new features)

Create your release specification branch:
```sh
detoul init
detoul create release-february master
```

Release specifications are placed in the `detoul-spec` branch and are named by the file name of the specification file. Let's say we have the following content for the committed `release-february` file in `detoul-spec`:
```
checkout master
take Task-1
take Task-3
take-squash Task-4 add dropdowns support
pick 40abfe3
```

Then we could test that it can be built without conflicts:
```sh
$ detoul test release-february
Checking out 'master'...
Taking 'Task-1-improve-buttons'...
Taking 'Task-2-fix-tables'...
Taking 'Task-3-add-dropdowns' (squashed)...
Picking '40abfe3'...
Integration branch succesfully created.
```

This means the branch is good, and we can then push our changes to `detoul-spec` to our remote repository and redeploy the release.

To actually make the branch (to test it or to use it), we can run `detoul make release-february`.

If your deployment procedured isn't configured to make the latest `detoul` branch, then you can make it and push it in a single command (`detoul push release-february`) and then deploy the branch `release-february`.

### Installation

1. `git clone https://github.com/ramonsnir/detoul`
2. Bring `detoul` into the path either by: editing your `~/.bashrc` adding `export PATH="$PATH:<path-to-cloned-repo>"`; or by symlinking it from `/usr/local/bin`.
3. Add to your `~/.bashrc` a line like `. <path-to-cloned-repo>/bash-completion/detoul` (for auto-completion support).

I plan to create (and maintain) packages for ArchLinux/Debian/RHEL and an installer for Windows.

### Requirements

Until `detoul` has a good test suite, this list should be taken with a grain of salt.

##### Linux

* a Linux-compatible operating system with a Bourne shell (bash)
* the perl5 interpreter
* git 1.7+ (developed with git 2.2.1, but also tested with a Centos 6.5 machine with git 1.7.1)

##### Cygwin

A Windows machine with Cygwin bash could use `detoul` too (note that perl is available as a Cygwin package). The colorized output in Cygwin has less colors (as it supports fewer colors than most terminal emulators), but that shouldn't bother users too much. If it does, then it is probably time for you to find out that Cygwin comes with Mintty (which is a pretty good terminal emulator - much better than the Windows command prompt) and can also work with xterm (which is available as a Cygwin package).

##### MinGW

Please see the issue https://github.com/ramonsnir/detoul/issues/21.

### detoul help

```
detoul: a declarative tool for creating integration branches in git
Usage:
    detoul init
    detoul specs-push
    detoul specs-pull
    detoul create <release> [<base_branch>]
    detoul (test|make) <release>
    detoul add-to <release> <branch> [--squash] [--merge] [--message "<message>"]
    detoul cat (<release>|<archived-release>)
    detoul edit <release> [--amend]
    detoul push <release>
    detoul archive <release>
    detoul unarchive <archived-release>
    detoul help
```

### rebase vs merge

The default `take` and `take-squash` actions behave a lot like `git-rebase` and so will not play nicely with all merge-detection methods (so Github and Bitbucket will not automatically mark relevant pull requests as merged).

If you don't like this behavior, you can instead use `take-merge` (or `detoul add-to <release> <branch> --merge`) which will keep the source branch an ancestor of the target branch.

### Using `git-rerere` with `detoul`

Clearly unless all your feature branches are always completely disjoint, then you will often have merge conflicts between branches. `detoul` employs `git-rerere` to **re**use **re**corded **re**solutions of conflicts. If you try to `detoul make` a release and `detoul` encounters a conflict, it will exit with a failure but will allow you to manually resolve the merge conflict. If you configured your `git` to use `git-rerere` (by calling `git config rerere.enabled true`), then it will automatically record your conflict resolution. The next time you run `detoul make` for the release, `detoul` will take your recorded resolutions and use them to resolve the encountered conflict. `detoul` will then save the `rr-cache` (recorded resolutions cache) inside the `detoul-spec` branch so that everyone that wishes to build the release could reuse your recorded resolutions automatically.

Note that while your resolutions will not be recorded without enabled `git-rerere`, `detoul` **will** automatically use recorded resolutions even without enabling `git-rerere` enabled in your local repository (of course, assuming someone recorded the conflict resolution before).

### Auto-completion gone wrong

After a couple of dozen releases, the release name auto-completion starts to be less useful, as there are too many irrelevant releases. If it bothers you, you can "archive" old releases using `detoul archive`. Archived releases won't be suggested by the auto-completion, and will not be built by mistake. They can still be browsed using `detoul cat` (which will also continue to auto-complete on them), and can be revived at any time using `detoul unarchive`.

### Why in bash?

I wrote this on my own time, and wanted to have fun. It shouldn't take more than an hour to translate detoul into any other language, once I choose to which one.

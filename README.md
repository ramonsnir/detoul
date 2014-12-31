# Detoul

Ramon Snir (c) 2014 [MIT License]

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

* a Linux-compatible operating system with a Bourne shell (bash)
* the perl5 interpreter
* git 1.7+ (developed with git 2.2.1, but also tested with a Centos 6.5 machine with git 1.7.1)

A Windows machine with Cygwin bash (MINGW not tested) could use `detoul` too (note that perl is available as a Cygwin package). The colorized output in Cygwin has less colors (as it supports fewer colors than most terminal emulators), but that shouldn't bother users too much. If it does, then it is probably time for you to find out that Cygwin comes with Mintty (which is a pretty good terminal emulator - much better than the Windows command prompt) and can also work with xterm (which is available as a Cygwin package).

### detoul help

```
detoul: a declarative tool for creating integration branches in git
Usage:
    detoul init
    detoul specs-push
    detoul specs-pull
    detoul create <release> [<base_branch>]
    detoul (test|make) <release>
    detoul add-to <release> <branch> [--rebase] [--squash|--fixup]
    detoul cat <release>
    detoul edit <release> [--amend]
    detoul push <release>
    detoul help
```

### rebase vs merge

The default `take` and `take-squash` actions behave a lot like `git-merge` and will play nicely with all merge-detection methods (so Github and Bitbucket will automatically mark relevant pull requests as merged). The difference between the two is in the first-parent commit ancestry, but the whole branch is still accessible through the second parent of the last commit (or only commit if using `take-squash`).

If you don't like this behavior, you can instead use `rebase` and `rebase-squash` which leave no trace of the original branch.

### Why in bash?

I wrote this on my own time, and wanted to have fun. It shouldn't take more than an hour to translate detoul into any other language, once I choose to which one.
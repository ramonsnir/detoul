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

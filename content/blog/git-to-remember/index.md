---
title: Git to Remember
date: 2020-10-28T20:00
---

Some useful Git commands, some of them for regular use but most you just need every once in a while. Here is where I remember them.

## Checkout a file from another branch

```shell
git checkout master ./path-to-file
```

Passing a file path to the checkout command checks out only that file from the branch specified. This is mostly useful to clean up messes.

## Delete merged branches locally

```shell
git fetch --prune
```

Useful when doing some housekeeping.

## Checkout the previously checked out branch

```shell
git checkout -
```

Quickly navigate between branches.

## Connect a local branch to a remote branch

```shell
git branch --set-upstream-to=origin/foo foo
```

This becomes necessary from time to time. You can also use `-u=` instead of `--set-upstream-to=`.

## Change the remote

```shell
git remote set-url <name> <new-url>
```

Necessary after migrating a repo, i. e. to a new host or if a fork upstream has changed.

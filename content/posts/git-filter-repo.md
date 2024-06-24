---
title: 'Using git filter-repo to clean up Git history'
date: 2024-01-22T10:34:29+01:00
tags: ['git', 'how-to']
---

As I was preparing an open source release of a tool, which was previously only developed internally I was looking into how to remove internal files (e.g. pipeline configurations) from the repository and additionally I wanted to clean up the commit history, as the commit authors contained internal information (e.g. usernames).

After a bit of searching the interwebs I stumbled over [`git filter-repo`](https://github.com/newren/git-filter-repo), which seems to be the solution of choice for this task nowadays.

## git filter-repo usage

First thing you need to be aware of is that by default `git filter-repo` refuses to work on repositories which haven't been freshly pulled.
This is is a security mechanism, which is meant to protect you from shooting yourself in the foot.
So the first step of the process is creating a fresh clone of your repository:

```shell
git clone https://example.com/repo.git
```

### Removing files from Git history

Compared to the complex combination of commands, which would be required to achieve the same result with `git filter-branch`, this is a stupid simple task to achieve with `git filter-repo`.
Here's an example how to remove a `Jenkinsfile` from the repository:

```shell
git filter-repo --invert-paths --path Jenkinsfile
```

The command also works the same if you want to remove whole repositories!

### Cleaning up commit authors

Because of the way the Git client was set up for the developers and some contributions being done directly in Bitbucket, some commits contained the username as the e-mail address of the commit id.
At first I thought I would have to write a script to loop over all commits and fix each authors, but luckily `git filter-repo` had my back again!
All I had to do was create a `.mailmap` file, which contained the information how to map commit author e-mail adresses to commit author names and run `git filter-repo --use-mailmap`.

```shell
# Will add John Doe as the commiter name for all commits made with the e-mail john.doe2@example.com
John Doe <john.doe2@example.com>
# Will add John Doe as the commiter name and john.doe2@example.com as the commit e-mail for all commits with the e-mail u123456@example.com
John Doe <john.doe2@example.com> <u123456@example.com>
```

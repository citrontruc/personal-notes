# Github

## Table of content

- [Github](#github)
  - [Table of content](#table-of-content)
  - [Create tags](#create-tags)
  - [Basic commands](#basic-commands)
  - [Git bisect](#git-bisect)

## Create tags

git tag v1.0.20
git push origin tag v1.0.20

## Basic commands

git init
git remote add origin \<repo-url>
git fetch
git checkout main

---

**git status** pour voir quels sont les changements que l'on pousse et que l'on tient prêt.

**git restore** pour laisser de côté tous les changements; Plus récent que git reset et à privilégier pour les cas où on veut juste retourner en arrière.

git commit --amend -m "Added HTML tags to hello.html"
Si besoin de rajouter des informations sur un commit qui a déjà été écrit.

**git revert** et ajouter ensuite le nom du commit afin de retourner en arrière.

**git switch and git checkout**. switch is newer and only for switching. git checkout is also for restoring branches.

**git reset** + name of branch to indicate which branch to reset to.

**git branch** + name of branch to create branch.

**git checkout -b \<name of new branch>** : lets you create a new branche and switch to it.

## Git bisect

Helps you with debugging. You mark commits as good and commmits as bad and we want to identify when did the code became bad.

In order to start the bisect process write the following command. We mark the current commit as being bad.

```sh
git bisect start
git bisect bad
```

We then identify a good commit and mark it as such:

```sh
git bisect good <good_commit_hash>
```

You can then write your tests in a middle commit:

```sh
./run-tests.sh
```

And mark as bad or good the commit in order to identify which of the commits broke the code.

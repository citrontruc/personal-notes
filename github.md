git tag v1.0.20
git push origin tag v1.0.20

---

git init
git remote add origin <repo-url>
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

**git checkout -b <name of new branch>** : lets you create a new branche and switch to it.

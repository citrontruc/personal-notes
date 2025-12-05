# Django

## Setup

// Créer une app dans django
uv run manage.py startapp music

// Lancer le serveur de test django
uv run manage.py runserver
exemple : uv run musicarchive/manage.py runserver

// Créer un nouveau projet django
uv run django-admin startproject musicarchive

python manage.py createsuperuser

## Organisation du code

Organisation en dossiers :
- Views : contient le code des pages et comment réagir lorsque l'utilisateur prend des actions.
- templates : code html des pages à afficher.
- models : indique les objets à manipuler.
forms : indique comment l'utilisateur rentre des informations.

## Bases de données

python manage.py makemigrations
python manage.py migrate
python manage.py shell

Important : besoin de spécifier les directories de tous les éléments. Notamment, préciser où se trouvent les tests avec pytest.ini et aussi préciser où se trouvent les tests.
Pour les migrations, inclure dans les fichiers init les modèles à inclure.

lancer la commande "uv run manage.py" affiche toutes les commandes qu'il est possible d'utiliser avec manage.py.

---

uv run manage.py createsuperuser
Manière dont les password sont stockés dans django : <algorithm>$<iterations>$<salt>$<hash>

ATTENTION : quand on supprime la db sqllite, on supprime aussi les données de superuser.

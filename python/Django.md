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

## Chemin d'une requête

1) Serveur web (gunicorn).
2) MiddleWare (traite toutes les requêtes entrantes et sortantes).
3) Routing (on cherche l'endpoint concerné par la requête).
4) Code caché derrière le endpoint.
5) Si modification d'une page, template de page.
6) HttpResponse ou JsonResponse.
7) Middleware
8) Serveur.

## Bases de données

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py shell
```

Important : besoin de spécifier les directories de tous les éléments. Notamment, préciser où se trouvent les tests avec pytest.ini et aussi préciser où se trouvent les tests.
Pour les migrations, inclure dans les fichiers init les modèles à inclure.

lancer la commande "uv run manage.py" affiche toutes les commandes qu'il est possible d'utiliser avec manage.py.

---

uv run manage.py createsuperuser
Manière dont les password sont stockés dans django : <algorithm>$<iterations>$<salt>$<hash>

ATTENTION : quand on supprime la db sqllite, on supprime aussi les données de superuser.

---

La connexion à une base de données se fait par l'utilisation d'un fichier settings.py.

## ORM

Start by defining your data models:

```python
class Album(models.Model):
    title = models.CharField(max_length = 30)
    artist = models.CharField(max_length = 30)
    genre = models.CharField(max_length = 30)

    def __str__(self):
        return self.title

class Song(models.Model):
    name = models.CharField(max_length = 100)
    album = models.ForeignKey(Album, on_delete = models.CASCADE)

    def __str__(self):
        return self.name
```

At all time, you can access the django ORM with the command (note: there is a simple python shell but the shell_plus command is more powerful and lets you give arguments).

```bash
python manage.py shell_plus --print-sql
```

Some example requests:

```python
# create an entity
e = Employee(first_name='John',last_name='Doe')
e.save()

# Get all values
Employee.objects.all()

# Get specific values & modify them
e = Employee.objects.get(id=1)
e.last_name = 'Smith'
e.save()

# Filters
Employee.objects.filter(first_name='Jane')

# Delete values
e.delete()
```

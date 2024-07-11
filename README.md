# gestion_projet

``` 
myproject/         # Répertoire principal du projet Django
├── myproject/     # Répertoire contenant les paramètres globaux du projet
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── env/           # Environnement virtuel (à créer avec `python -m venv env`)
├── manage.py      # Script de gestion Django
├── projets/       # Application "projets"
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py   # Modèles de l'application "projets"
│   ├── serializers.py  # Sérialiseurs de l'application "projets"
│   ├── tests.py
│   ├── urls.py     # URLs de l'application "projets"
│   └── views.py    # Vues de l'application "projets"
├── taches/        # Application "taches"
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py   # Modèles de l'application "taches"
│   ├── serializers.py  # Sérialiseurs de l'application "taches"
│   ├── tests.py
│   ├── urls.py     # URLs de l'application "taches"
│   └── views.py    # Vues de l'application "taches"
└── equipes/       # Application "equipes"
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py   # Modèles de l'application "equipes"
    ├── serializers.py  # Sérialiseurs de l'application "equipes"
    ├── tests.py
    ├── urls.py     # URLs de l'application "equipes"
    └── views.py    # Vues de l'application "equipes"

```

1. **Créer un environnement virtuel :**

   Ouvrez l'invite de commande (cmd) ou PowerShell et exécutez la commande suivante pour créer un environnement virtuel nommé `env` :

   ```bash
   python -m venv env
   ```

2. **Activer l'environnement virtuel :**

   Pour activer l'environnement virtuel sur Windows, utilisez la commande suivante :

   ```bash
   env\Scripts\activate
   ```
### Étapes complètes de configuration

1. **Créer et activer l'environnement virtuel :**
   ```bash
   python -m venv env
   env\Scripts\activate
   ```

2. **Installer les dépendances :**
   ```bash
   pip install django djangorestframework mysqlclient
   ```

3. **Créer le projet Django et les applications :**

   Créez le projet Django :
   ```bash
   django-admin startproject myproject
   cd myproject
   ```

   Créez les applications :
   ```bash
   python manage.py startapp projets
   python manage.py startapp taches
   python manage.py startapp equipes
   ```

4. **Configurer `settings.py` pour MySQL et les applications :**

   Ouvrez `myproject/settings.py` et configurez les paramètres suivants :

   ```python
   import os

   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.mysql',
           'NAME': 'django_db',
           'USER': 'django_user',
           'PASSWORD': 'django_password',
           'HOST': 'localhost',
           'PORT': '3306',
       }
   }

   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'rest_framework',
       'projets',
       'taches',
       'equipes',
   ]

   # Other settings...
   ```

5. **Configurer les URLs principales :**

   Ouvrez `myproject/urls.py` et configurez les URLs pour les applications :

   ```python
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('api/projets/', include('projets.urls')),
       path('api/taches/', include('taches.urls')),
       path('api/equipes/', include('equipes.urls')),
   ]
   ```

### Application `projets`

#### `projets/models.py`

```python
from django.db import models

class Client(models.Model):
    nom = models.CharField(max_length=255)
    adresse = models.CharField(max_length=255)
    numero_telephone = models.CharField(max_length=20)
    personne_contact = models.CharField(max_length=255)

    def __str__(self):
        return self.nom

class Projet(models.Model):
    nom = models.CharField(max_length=255)
    description = models.TextField()
    date_debut = models.DateField()
    date_fin_prevue = models.DateField()
    budget = models.DecimalField(max_digits=10, decimal_places=2)
    clients = models.ManyToManyField(Client, through='ProjetClient')

    def __str__(self):
        return self.nom

class ProjetClient(models.Model):
    projet = models.ForeignKey(Projet, on_delete=models.CASCADE)
    client = models.ForeignKey(Client, on_delete=models.CASCADE)

    class Meta:
        unique_together = ('projet', 'client')
```

#### `projets/serializers.py`

```python
from rest_framework import serializers
from .models import Client, Projet, ProjetClient

class ClientSerializer(serializers.ModelSerializer):
    class Meta:
        model = Client
        fields = '__all__'

class ProjetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Projet
        fields = '__all__'

class ProjetClientSerializer(serializers.ModelSerializer):
    class Meta:
        model = ProjetClient
        fields = '__all__'
```

#### `projets/views.py`

```python
from rest_framework import viewsets
from .models import Client, Projet, ProjetClient
from .serializers import ClientSerializer, ProjetSerializer, ProjetClientSerializer

class ClientViewSet(viewsets.ModelViewSet):
    queryset = Client.objects.all()
    serializer_class = ClientSerializer

class ProjetViewSet(viewsets.ModelViewSet):
    queryset = Projet.objects.all()
    serializer_class = ProjetSerializer

class ProjetClientViewSet(viewsets.ModelViewSet):
    queryset = ProjetClient.objects.all()
    serializer_class = ProjetClientSerializer
```

#### `projets/urls.py`

```python
from rest_framework.routers import DefaultRouter
from .views import ClientViewSet, ProjetViewSet, ProjetClientViewSet

router = DefaultRouter()
router.register(r'clients', ClientViewSet)
router.register(r'projets', ProjetViewSet)
router.register(r'projetclients', ProjetClientViewSet)

urlpatterns = router.urls
```

### Application `taches`

#### `taches/models.py`

```python
from django.db import models
from projets.models import Projet

class Tache(models.Model):
    PRIORITE_CHOICES = [
        ('haute', 'Haute'),
        ('moyenne', 'Moyenne'),
        ('basse', 'Basse'),
    ]

    STATUT_CHOICES = [
        ('en cours', 'En cours'),
        ('terminee', 'Terminée'),
    ]

    projet = models.ForeignKey(Projet, on_delete=models.CASCADE)
    nom = models.CharField(max_length=255)
    description = models.TextField()
    date_debut = models.DateField()
    date_fin_prevue = models.DateField()
    priorite = models.CharField(max_length=10, choices=PRIORITE_CHOICES)
    statut = models.CharField(max_length=20, choices=STATUT_CHOICES)

    def __str__(self):
        return self.nom
```

#### `taches/serializers.py`

```python
from rest_framework import serializers
from .models import Tache

class TacheSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tache
        fields = '__all__'
```

#### `taches/views.py`

```python
from rest_framework import viewsets
from .models import Tache
from .serializers import TacheSerializer

class TacheViewSet(viewsets.ModelViewSet):
    queryset = Tache.objects.all()
    serializer_class = TacheSerializer
```

#### `taches/urls.py`

```python
from rest_framework.routers import DefaultRouter
from .views import TacheViewSet

router = DefaultRouter()
router.register(r'taches', TacheViewSet)

urlpatterns = router.urls
```

### Application `equipes`

#### `equipes/models.py`

```python
from django.db import models

class Equipe(models.Model):
    nom = models.CharField(max_length=255)
    description = models.TextField()
    nombre_max_membres = models.IntegerField()

    def __str__(self):
        return self.nom

class MembreEquipe(models.Model):
    equipe = models.ForeignKey(Equipe, on_delete=models.CASCADE)
    nom = models.CharField(max_length=255)
    role = models.CharField(max_length=50)
    email = models.EmailField()

    def __str__(self):
        return self.nom
```

#### `equipes/serializers.py`

```python
from rest_framework import serializers
from .models import Equipe, MembreEquipe

class EquipeSerializer(serializers.ModelSerializer):
    class Meta:
        model = Equipe
        fields = '__all__'

class MembreEquipeSerializer(serializers.ModelSerializer):
    class Meta:
        model = MembreEquipe
        fields = '__all__'
```

#### `equipes/views.py`

```python
from rest_framework import viewsets
from .models import Equipe, MembreEquipe
from .serializers import EquipeSerializer, MembreEquipeSerializer

class EquipeViewSet(viewsets.ModelViewSet):
    queryset = Equipe.objects.all()
    serializer_class = EquipeSerializer

class MembreEquipeViewSet(viewsets.ModelViewSet):
    queryset = MembreEquipe.objects.all()
    serializer_class = MembreEquipeSerializer
```

#### `equipes/urls.py`

```python
from rest_framework.routers import DefaultRouter
from .views import EquipeViewSet, MembreEquipeViewSet

router = DefaultRouter()
router.register(r'equipes', EquipeViewSet)
router.register(r'membresequipes', MembreEquipeViewSet)

urlpatterns = router.urls
```

### Étapes supplémentaires

1. **Créer et appliquer les migrations :**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Créer un superutilisateur pour accéder à l'admin Django :**
   ```bash
   python manage.py createsuperuser
   ```

3. **Démarrer le serveur de développement Django :**
   ```bash
   python manage.py runserver
   ```

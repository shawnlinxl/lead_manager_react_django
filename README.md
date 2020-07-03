# Lead Manager: React + Django REST framework tutorial

[Tutorial](https://youtube.com/watch?v=Uyei2iDA4Hs) from Traversy Media on YouTube.

## Workflow

### Setting up Django project

- Install basic dependencies for backend:

  ```sh
  pip install django djangorestframework django-rest-knox
  ```

  - djangorestframework: for creating REST api
  - django-rest-know: for authentication

- Create project

  ```sh
  djangoo-admin startproject leadmanager
  ```

- Create app

  ```sh
  cd leadmanager
  python manage.py startapp leads
  ```

- Add installed apps to project settings

  ```py
  INSTALLED_APPS = [
    'leads',
    'rest_framework'
  ]
  ```

### Setting up app data models

- Create model: go to `/leads/models`

  ```py
  class Lead(models.Model):
      name = models.CharField(max_length=100)
      email = models.EmailField(max_length=100, unique=True)
      message = models.CharField(max_length=500, blank=True) 
      created_at = models.DateTimeField(auto_now_add=True)
  ```

  - **unique=True** restricts the field to be unique
  - **blank=True** marks the field optional
  - **auto_now_add=True** adds time automatically when entry is created

- Create migrations for the model we've just created

  ```sh
  python manage.py makemigrations leads
  python manage.py migrate
  ```

  - **makemigration** will create a file called `0001_inital.py` for our model in the `leads/migrations` folder
  - running **migrate** will create database tables for our model

### Setting up API

- Create serializer

  Under leads, create a file called `serializers.py`.

  ```py
  from rest_framework import serializers
  from leads.models import Lead

  # Lead Serializer
  class LeadSerializer(serializers.ModelSerializer):
      class Meta:
          model = Lead
          fields = "__all__"
  ```

- Create API

  Under leads, create a file called `api.py`.

  ```py
  from leads.models import Lead
  from rest_framework import viewsets, permissions
  from .serializers import LeadSerializer

  # Lead Viewset
  class LeadViewSet(viewsets.ModelViewSet):
      queryset = Lead.objects.all()
      permission_classes = [permissions.AllowAny]
      serializer_class = LeadSerializer
  ```

  ViewSets allows you to create a full CRUD API without the need to specify everything.

- Create URLs

  - In `leadmanager/urls.py`, add

    ```py
    urlpatterns = [
        path('', include('leads.urls'))
    ]
    ```

  - In leads, create `urls.py`.
  
    ```py
    from rest_framework import routers
    from .api import LeadViewSet

    router = routers.DefaultRouter()
    router.register("api/leads", LeadViewSet, "leads")

    urlpatterns = router.urls
    ```

  At this point, we should have a basic CRUD API ready to run.

- Run Server
  
  ```sh
  python manage.py runserver
  ```

### Creating the frontend project structure

- Create an App for the frontend
  
  ```sh
  python manage.py startapp frontend
  ```

- Make directory for frontend code
  
  ```sh
  mkdir -p ./frontend/src/components
  mkdir -p ./frontend/{static,templates}/frontend
  ```

  - `src/components` to store all components, and Redux things for React application
  - `templates` to handle `index.html`
  - `static` will be the compiled JavaScript: use webpack to compile source into main.js.

### Installing tools needed for the frontend

- Move to root folder (same level as Pipfile)
- Initialize npm project
  
  ```sh
  npm init -y
  ```

  `-y` is used to skip answering questions, and use defaults
- Install dev dependency

  ```sh
  npm i -D webpack webpack-cli
  ```
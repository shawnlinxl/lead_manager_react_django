# Lead Manager: React + Django REST framework tutorial

[Tutorial](https://youtube.com/watch?v=Uyei2iDA4Hs) from Traversy Media on YouTube.

## Setting up Django Backend

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

## Setting up React Redux frontend

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

  - Install webpack

    ```sh
    npm i -D webpack webpack-cli
    ```

  - Install babel

    ```sh
    npm i -D @babel/core babel-loader @babel/preset-env @babel/preset-react babel-plugin-transform-class-properties
    ```

  - Install react

    ```sh
    npm i react react-dom prop-types
    ```

  - Create .babelrc in root to be able to use the presets, and add the following to the file

    ```json
    {
      "presets": ["@babel/preset-env", "@babel/preset-react"],
      "plugins": ["transform-class-properties"]
    }
    ```

  - Create webpack.config.js in root to load babel loader

    ```javascript
    module.exports = {
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: "babel-loader",
            },
          },
        ],
      },
    };
    ```

  - Add scripts to package.json

    ```json
    "scripts": {
      "dev": "webpack --mode development --watch ./leadmanager/frontend/src/index.js --output ./leadmanager/frontend/static/frontend/main.js",
      "build": "webpack --mode production ./leadmanager/frontend/src/index.js --output ./leadmanager/frontend/static/frontend/main.js"
    }
    ```

    scripts can be run via `npm run build` and `npm run dev`.

### Create React source code

- Under src, create `index.js`

  ```javascript
  import App from "./components/App";
  ```

- Under create App.js under src/components/app

  ```javascript
  import React, { Component } from "react";
  import ReactDOM from "react-dom";

  class App extends Component {
    render() {
      return <h1>React App</h1>;
    }
  }

  ReactDOM.render(<App />, document.getElementById("app"));
  ```

- Create index.html under templates/frontend

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://bootswatch.com/4/cosmo/bootstrap.min.css">
    <title>Lead Manager</title>
  </head>
  <body>
    <div id="app"></div>
    {% load static %}
    <script src="{% static "frontend/main.js" %}"></script>
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/js/bootstrap.min.js" integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI" crossorigin="anonymous"></script>
  </body>
  </html>
  ```

  `<script src="{% static "frontend/main.js" %}"></script>` is the Django syntax for templates.
  Note in here, we've included bootstrap and corresponding bootstrap javascripts.

### Adding frontend to Django

- Add app to django project settings

  ```py
  INSTALLED_APPS = [
      "frontend",
  ]
  ```

- Add app template to frontend/views.py, so Django can display it

  ```py
  from django.shortcuts import render

  def index(request):
      return render(request, "frontend/index.html")
  ```

- Add url.

  - In frontend, create urls.py.

  ```py
  from django.urls import path
  from . import views

  urlpatterns = [path("", views.index)]
  ```

  - In leadmanager/urls.py, add the url

  ```py
  urlpatterns = [
    path("", include("frontend.urls")),
    path("", include("leads.urls"))
  ]
  ```

### Now test the app is up and running

- run npm build

  ```sh
  npm run dev
  ```

- go to localhost:8000. You should see the react app.

## Create the app

### Build the React app

- Create src/layout/Header.js
- Add header to App.js and render it
- Create src/leads/Dashboard.js to contain Form.js and Leads.js
- Add dashboard to App.js

### Implement Redux

- Install redux

  ```sh
  npm i redux react-redux redux-thunk redux-devtools-extension
  ```

  - redux-thunk: middleware for async requests

- Create store.js under src

- Create reducers folder, and an index.js file which is the meeting space for all reducers we are going to have

- Import provider and store in App.js: note how provider wraps around all other components in here

  ```javascript
  class App extends Component {
    render() {
      return (
        <Provider store={store}>
          <Fragment>
            <Header />
            <div className="container">
              <Dashboard />
            </div>
          </Fragment>
        </Provider>
      );
    }
  }
  ```

### Setup Leads Reducer

- Create leads.js reducer under reducers
- Create types.js under actions to organize things
- Create leads.js under actions
- install axios

  ```sh
  npm i axios
  ```

- Connect Leads component with redux
- Add "dispatch" in asynchronous actions

## Setup Authentication

### Implement in backend

- in leads, models.py, import default django user model
- add owner field to leads
- change permission settings in api.py

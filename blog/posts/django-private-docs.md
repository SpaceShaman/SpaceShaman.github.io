---
authors:
    - SpaceShaman
slug: django-private-docs
date: 2023-12-15
comments: true
categories:
    - Python
    - Django
---

# Createing private documentation with Django and MkDocs

In this article, I will show you how to create private documentation that can only be accessed by users with the appropriate permissions. We will use Django: [Django](https://www.djangoproject.com/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

<!-- more -->

## Dependencies

### Material for MkDocs

[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) is an extension for [MkDocs](https://www.mkdocs.org/), that adds new features and changes the look of the documentation (The page you are currently reading is written using [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)). In this article, we will use [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), but if you want to use the regular [MkDocs](https://www.mkdocs.org/), it should not be a problem.

### Django

[Django](https://www.djangoproject.com/) is a popular framework for creating web applications.

## Preparing the Django Project

The first step is to create a Django project and an application to display the documentation.

1. Install Django and Material for MkDocs

    ``` bash
    pip install django mkdocs-material
    ```

2. Create a Django project

    ``` bash
    django-admin startproject private_docs
    ```

3. Create a Django application to display the documentation

    ``` bash
    cd private_docs
    django-admin startapp doc_viewer
    ```

4. Add the application to the `settings.py` file

    ``` python linenums="1" hl_lines="4"
    # private_docs/settings.py
    INSTALLED_APPS = [
        ...
        'doc_viewer',
    ]
    ```

5. Add the path to the `assets` folder of the documentation to the static files of Django (without this, images will not be displayed)

    ``` python linenums="1" hl_lines="4"
    # private_docs/settings.py
    STATIC_ROOT = os.path.join(BASE_DIR, ".static")
    STATIC_URL = '/static/'
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, ".docs/assets"),
    ]
    ```

6. Add the path to the view that displays the documentation to the `urls.py` file

    ``` python linenums="1" hl_lines="6"
    # private_docs/urls.py
    from django.urls import re_path
    from doc_viewer.views import docs_view

    urlpatterns = [
        re_path(r"^docs/(?P<path>.*)$", docs_view, name="docs"),
    ]
    ```

## Creating the MkDocs Documentation

The next step is to create the documentation. To do this, create a file mkdocs.yml in the main project directory and add the following code to it:

``` yaml linenums="1"
# mkdocs.yml
site_name: Private Docs
use_directory_urls: false # This is important to correctly display the documentation via Django
site_dir: .docs # This is a directory where the documentation will be generated
theme:
    name: material # If you want to use the default MkDocs theme, remove this line
```

Then create a directory `docs`, in which we will place the documentation files written in Markdown. You can also create a file `docs/index.md` which will be the main page of the documentation.

## Creating the Django View to Display the Documentation

The next step is to create the Django view to display the documentation. To do this, open the file `doc_viewer/views.py` and add the following code to it:

``` python linenums="1"
# doc_viewer/views.py
import os
from django.conf import settings
from django.contrib.auth.decorators import permission_required
from django.http import HttpResponse, HttpResponseNotFound

@permission_required("permission_name") # Replace permission_name with the name of the permission that will be required to view the documentation
import os

from django.conf import settings
from django.contrib.auth.decorators import permission_required
from django.http import HttpResponse, HttpResponseNotFound


@permission_required(
    "doc_viewer.view_docs"
)  # Replace permission_name with the name of the permission that will be required to view the documentation
def docs_view(request, path) -> HttpResponseNotFound | HttpResponse:
    mkdocs_path = os.path.join(settings.BASE_DIR, ".docs")
    # Check if the documentation exists
    if not os.path.exists(mkdocs_path):
        return HttpResponseNotFound("Documentation does not exist.")

    # If the path is empty, display the home page
    if path == "":
        path = "index.html"

    # Check if the file for the given path exists in the documentation
    file_path = os.path.join(mkdocs_path, path)
    if not os.path.isfile(file_path):
        return HttpResponseNotFound("Page does not exist.")

    # Reading the contents of the file
    with open(file_path, "r", encoding="utf-8") as f:
        content = f.read()

    # If the file is an HTML file, replace relative paths with absolute paths
    if path.endswith(".html"):
        url = request._current_scheme_host
        content = content.replace('"assets/', f'"{url}/static/')
        content = content.replace('"../assets/', f'"{url}/static/')
        content = content.replace('"../../assets/', f'"{url}/static/')
        content = content.replace('"img/', f'"{url}/static/img/')
        content = content.replace('"../img/', f'"{url}/static/img/')
        content = content.replace('"../../img/', f'"{url}/static/img/')

    # Return the contents of the file
    return HttpResponse(content)
```

## Running the Project

Congratulations! Your project is now ready. All you need to do is generate the documentation and start the Django server. To do this, follow these steps:

1. Generate the documentation

    ``` bash
    mkdocs build
    ```

2. Run migrations and collectstatic

    ``` bash
    python manage.py migrate
    python manage.py collectstatic
    ```

3. Create a superuser

    ``` bash
    python manage.py createsuperuser
    ```

4. Run the Django server

    ``` bash
    python manage.py runserver
    ```

5. Go to the admin panel [http://localhost:8000/admin/](http://localhost:8000/admin/) and log in with the superuser account you created in step 3.

6. You're done! Now you can go to the page [http://localhost:8000/docs/](http://localhost:8000/docs/) and see your documentation.

## Summary

In this article, I showed you how to create private documentation using Django and Material for MkDocs. If you have any questions or comments, please post them in the comments section below. If you want to stay up-to-date with my articles, add my blog to your favorite RSS feeds. See you in the next article!

## Final Project

You can find the final project on [GitHub](https://github.com/SpaceShaman/django-private-docs).

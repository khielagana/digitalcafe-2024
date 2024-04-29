# Your first page

2024-04-28

We will now write your very first webpage with Django. There is a chance you have already done the "Intro to HTML" assignment. If you have not, or if your course doesn't have this assignment, in short: browsers render text formatted as Hypertext Markup Language (HTML). You can serve HTML as a file, or you can also serve it as a string from a web framework like Django.

## Creating your webpage

There is a little more Django boilerplate we have to work through. Run this command to create a new "app." Django apps are sub-modules that Django uses to structure projects. Since we are only interested in making one web app, we will simply have a single Django app called "core."

```bash
python manage.py startapp core
```

This will create a new app beside the package directory called `core/`. If you have followed the naming conventions so far, the full filepath of `core/` will be `digitalcaferoot/digitalcafe/core/`.

The `core/` directory will contain these files:

```
core
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py
```

We are primarily interested in the `core/views.py` file. Open it in your text editor.

Django routes HTTP requests to Python functions called "views." Views, being functions, can execute code. Ultimately, they must return something that the client can use, which usually means HTML.

In `core/views.py`, write the following code. Replace any code that may have been added by default.

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello world!")
```

This function, `index`, is a view. We will now wire up this view to execute when you visit the server's root page, `/`.

Inside the `core/` directory, create a new file `core/urls.py`. Inside this file, write the following code:

```python
from django.urls import path

from . import views # This . package just means "the current package; we are importing the sister file "views.py"

urlpatterns = [
    path("", views.index, name="index")
]
```

We are not quite done. Go to the package directory (i.e., `digitalcaferoot/digitalcafe/digitalcafe/`) and modify `digitalcaferoot/digitalcafe/digitalcafe/urls.py` with the following code:

```python
from django.contrib import admin
from django.urls import path
from django.urls import include # Add this line

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("core.urls")), # Add this line
]
```

Yes, again, this is confusing. Every time you add a new view, you will need to register it with your app's `urls.py`, and every time you add a new app, you will need to register it with the package directory's `urls.py`.

Now your first view is ready. Run the server again and navigate to `http://localhost:8000/`. You should now see "Hello world!" rendered in your browser.

Remember that what you wrote in `core/urls.py` is text that can render HTML. If you want to try writing HTML instead of plain text, you can change the string from `"Hello world!"` to `"<h1>Hello world!</h1>"`.

## Takeaways

I personally like to follow this workflow when adding a new view.

- First, write the function in your app's `views.py`.
- Then, register the function in your app's `urls.py`.
- Then, if necessary, register the whole app in your larger Django project's `urls.py`.

You may find out later on that Django supports something called "class-based views." We advise you to ignore this feature. Functions are much simpler and much easier to understand.

## Checkpoint

Please screenshot your browser at `http://localhost:8000/`, displaying "Hello world!" as an _HTML header 1._

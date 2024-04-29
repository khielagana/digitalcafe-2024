# Hello Django

2024-04-28

In this section of the Digital Cafe walkthrough, we will set up your Django project.

The rest of this walkthrough assumes that you are familiar with your terminal or are at least capable of learning it. We will also be using UNIX file path conventions: directories and files will be separated by forward slashes (e.g., `digitalcaferoot/digitalcafe/manage.py`). Mac OS follows UNIX file path conventions. If you are on Windows, forward slashes are replaced by backward slashes (e.g., `digitalcaferoot\digitalcafe\manage.py`).

## Django development approach

The original version of Digital Cafe was written using Flask, which is a Python web microframework. This version of Digital Cafe is written using Django, which is a much heavier framework.

If you come across the Flask version of Digital Cafe, you may notice that it is structured as many quick iterations. This was only possible (and desirable) because Flask is not opinionated about how to handle web app development. In the Django version of Digital Cafe, we need to take things a little more slowly and thoroughly. In a similar, inverted way, this is only possible and desirable because Django already has the answers for building web apps.

For example, the Flask version of Digital Cafe starts by storing the database in a dictionary before iterating to MongoDB. We do not do this in Django because we only have one option for our database: a relational database. Yes, there are different flavors of relational database (SQLite vs. PostgreSQL, for instance), but the choice is made for us to use a relational database.

Being railroaded like this has its benefits. It makes it less mentally taxing to build features if you know that there's only one real way to do it.

## Structure of a Django project

A Django project is composed of multiple layers of directories, which each have multiple files.

The directory you created in the previous section, `digitalcaferoot`, is what we will call the "true root" of the entire Django project. It is the umbrella directory that contains every other file related to your Django project. Your true root will contain your Python virtual environment, and it is where you will start your Django editing sessions.

We will now create a Django project. Run this command in your terminal window. Make sure your virtual environment is active.

```bash
django-admin startproject digitalcafe
```

This will create a new directory called `digitalcafe`. We will call this directory the "project root."

You should now see this file structure, starting from the project root:

```
digitalcafe
├── digitalcafe
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

The top-level `digitalcafe` directory is the project root. The second-level `digitalcafe` is a _separate_ directory that we will call the "package directory." It contains more management files.

Let us briefly recap:

- There is a folder `digitalcaferoot` that is the "true root" of your entire Django project. It contains your Python virtual environment. When starting a coding session, you will usually start here and activate your virtual environment.
- There is a folder `digitalcaferoot/digitalcafe/` that is the "project root" of your Django project. It was created when you ran `django-admin startproject digitalcafe`. You will spend most of your terminal time here because it hosts `manage.py`, which is the file Django uses for command-line tasks.
- There is a folder `digitalcaferoot/digitalcafe/digitalcafe/` that is the "package directory" of your Django project.

Yes, it's confusing.

Thankfully, that's all we need to do to start your very first Django project. Navigate into the project root (i.e., `digitalcaferoot/digitalcafe/`) and run this command:

```bash
python manage.py runserver
```

Your terminal should display messages that it is now running a Django development server.

Open your web browser and navigate to the URL `http://localhost:8000`. This URL points to your local server; it can't be accessed by anyone else. You should now see a welcome page for Django, saying "The install worked successfully! Congratulations!"

## Checkpoint

Please take a screenshot of your browser, pointed at `http://localhost:8000`.

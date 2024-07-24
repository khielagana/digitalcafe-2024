# The admin site

2024-04-28

In the previous section, we discussed how to design the database. The main issue with this is that anyone who wants to edit data would still need to know how to code. Most people in the world (and very likely your workplace) will not be technical, and they will also have a need to edit data in the backend.

Django provides an "admin" app by default to help your non-technical administrators change the database by themselves. We will set it up in this section.

## `createsuperuser`

Run this command to create a user account with administrator privileges. This type of account is typically called a "superuser" among software professionals.

```bash
python manage.py createsuperuser
```

You will be prompted for a username, an email, and a password. We will assume that you chose `cafeadmin` as your username.

Start the server and navigate to `http://localhost:8000/admin`. Here, you will be able to log in as the superuser you just created. Note, though, that your Product model from your `core` app will not be available. To make it available on the admin site, you will need to register the model in `core/admin.py`.

```python
from django.contrib import admin

from .models import Product

# Register your models here.

admin.site.register(Product)
```

Now, your admin site should make Product available to CRUD through a graphical interface.

Make a special note that the admin site's User model is also available for apps other than the admin app to use. In other words, you can simply use the registration/login functions freely provided by Django in `core`. We will make use of this later in the walkthrough. This is how Django apps are expected to manage users.

## Checkpoint

Screenshot yourself logged into your superuser account on the admin app.

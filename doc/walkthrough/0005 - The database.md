# The database

2024-04-28

We now come to one of the best parts of Django. As a framework, Django is incredibly good at managing a proper _database_ in which you can store your data. In this section, we will migrate our product data from a hardcoded list within the `index` view function to a proper database table in SQLite.

## Setting up the database

We haven't set up our database yet. Before we do anything related to the database, run this command.

```bash
python manage.py migrate
```

This tells Django to "migrate", or update, your database to a new state. Running this command for the first time tells Django to set up the database to its default state. The default migrations usually come from Django's built-in `admin` app, which we will use later.

You might see a new file, `digitalcaferoot/digitalcafe/db.sqlite3`. This is the SQLite database file that will store all of our app's data going forward. SQLite, of course, is not the only database engine we can use, but it is by far the easiest to set up, and it is more than capable of handling anything that a small website like Digital Cafe can throw at it.

## Making your own models

Let's now make our own database tables. We will start by making a table to hold data about our products. This means we need to create a model called `Product`, which will have at least two fields: a text field for the `name`, and a number field for the `price`.

Open `core/models.py` and add this code:

```python
from django.db import models

# Create your models here.

class Product(models.Model):
    name = models.CharField(max_length=50)
    price = models.IntegerField()
```

Django models are represented as Python classes. When we eventually turn these models into database tables, each model will get its own database table.

We can now generate the migrations needed to transform our database to the state we prescribe in the `core/models.py` file by running this command.

```bash
python manage.py makemigrations core
```

Now that our migration files have been generated in `core/migrations/`, we can actually run the migrations by again running:

```bash
python manage.py migrate
```

The overall workflow of adding models is quite nice in Django. First, change `models.py`, then makemigrations, then migrate. Django helpfully figures out what changed between the last time you migrated, so you are free to specify what _end state_ you want your models to be in rather than be forced to write exactly how you want the database to change.

## Interacting with your models

Django models inherit from `models.Model`. This inheritance does a lot of heavy lifting. It gives Django models methods that make them very useful for create-read-update-delete (CRUD) operations.

We will test out our models in a Django shell. Open the Django shell with this command:

```bash
python manage.py shell
```

Import your models into the Django shell.

```python
>>> from core.models import Product
```

Now, you can perform CRUD on Products.

```python
>>> p = Product(name="Americano", price=110)
>>> p.save()
>>> p = Product(name="Cappuccino", price=130)
>>> p.save()
>>> Product.objects.all()
<QuerySet [<Product: Product object (1)>, <Product: Product object (2)>]>
>>> americano = Product.objects.get(name="Americano")
>>> americano.price
110
```

If you want to change the way a Product represents itself in code, you can provide an implementation for its built-in `__str__` method:

```python
class Product(models.Model):
    name = models.CharField(max_length=50)
    price = models.IntegerField()
    def __str__(self):
        return f'{self.name}'
```

Now, if you open a new shell, you can see that Product is more human-friendly in the logs.

```python
>>> from core.models import Product
>>> Product.objects.all()
<QuerySet [<Product: Americano>, <Product: Cappuccino>]>
```

The takeaway here is that you can import models to be used in your views, too. This is how views can interact with your database. To make models available to views, import them like this:

```python
# In `core/views.py`
from .models import Product
```

## Checkpoint

Let's extend what you've learned in this section.

1. Add a new product, Espresso, whose price is PHP 100, to the database using the Django shell.
2. In your view function `index`, source your product data from the database instead of a hardcoded list.

Screenshot the Django shell, and screenshot your new code in `index`.

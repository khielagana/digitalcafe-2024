# HTML

2024-04-28

You've seen how to write a basic view. Now, we will write a more complicated view to list the products available on our website.

## The view as a function of data

Let's say that the Digital Cafe has only just opened. They only have two products: Americano, which is sold at PHP 110, and Cappuccino, which is sold at PHP 140.

You could easily write this as bare HTML, but one of the points of using a web framework is that the data that you must render will very likely change. It is generally unwise to write the data directly into your view like that. Most people choose to store their data somewhere then dynamically render the HTML based on the data. That's why we see the view as a _function_ of data: the view changes depending on the data.

## Registering your app

Before we continue, we should register our `core` app with the larger Django package. Navigate to `digitalcafe/settings.py` and add this line to the `INSTALLED_APPS` list:

```python
INSTALLED_APPS = [
    'core.apps.CoreConfig', # Add this line
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

This will tell Django that `core` is an active app. This step is necessary for the next section.

## Templates

Let's modify the `index` view you made in the previous section to display the products instead of a hello world message.

```python
def index(request):
    products = [
        {"name": "Americano", "price": 110},
        {"name": "Cappuccino", "price": 140},
    ]
    output = str(products)
    return HttpResponse(output)
```

We now express our products first as data, then render them as a string. If you visit your index page now, you will see the products list, as a string, displayed on the web page. Of course, this is not what we intend: we want to format the products nicely. We could just format an HTML string directly in the view, but more typically, Django projects use something called "templates."

Make a new directory under `core/` called `templates/`. Under `templates/`, make another directory called `core/`. (This is another rough edge of getting started with Django; it's related to namespacing templates.) Under `core/templates/core/`, make a file called `index.html`. This is where we will write our template.

Add this code to `index.html`.

```html
{% if product_data %}
    <ul>
        {% for product_record in product_data %}
            <li>{{ product_record.name }} - PHP {{ product_record.price }}</li>
        {% endfor %}
    </ul>
{% endif %}
```

This may look strange if you have written HTML before. The curly braces here are part of Django's templating language. A `{% %}` indicates a control structure like if or for. A `{{ }}` indicates a variable.

Other than the strange syntax, it should be transparent what we are aiming to do here. We want to loop over a list called `product_data`. Each element in `product_data` will be bound to the variable `product_record`, which is supposed to have two properties `name` and `price`.

Return to `core/views.py` and change the file contents to the following code.

```python
from django.http import HttpResponse
from django.template import loader

def index(request):
    # Load the template
    template = loader.get_template("core/index.html")
    products = [
        {"name": "Americano", "price": 110},
        {"name": "Cappuccino", "price": 140},
    ]
    context = {
        "product_data": products
    }
    return HttpResponse(template.render(context, request))
```

We can make our products list from the view function available to the HTML template by passing in a dictionary called `context`. (Note that even though you usually access dictionary properties with a square bracket accessor, you need to access properties with a dot accessor in Django templates.)

Visit `http://localhost:8000` again to see the products rendered as an unordered HTML list. This is much better.

## Checkpoint

Let's check your ability to extend what you've learned. Implement two changes to what we built in this section:

1. Change the HTML output. Instead of an unordered list, make an HTML table with two columns: product name and product price. There should be a header row with these two columns. Each row of table data should be about one product from the `products` list in our view function.
2. Add a new product, Espresso, which is sold at PHP 100.

Screenshot the new page on your browser.

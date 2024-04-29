# Viewing products

2024-04-29

We now have the foundations we need to quickly begin developing features for Digital Cafe. Our next task is to let users click on a product on the main list and view a product detail page.

These next few pages will go fast. Our aim is to demonstrate that once you understand Django's basics, you can't help but move this fast.

## Routing

We effectively want to have two main "pages."

- First, at `/`, a page to display the list of all products. We already have this page.
- Second, at `/product/<product_id>`, a page to display the details of a specific product. Note that there is a variable part of this path, `product_id`. Django's routing helps us capture such variables.

When designing a website, you usually allow navigation between pages using _links_. The index page should link to each of the products, and the product pages should link back to the index.

## Make a new view

We need to make a new view to handle the product page. We will follow the workflow for adding such a view.

Define a new function `product_detail` in your `core/views.py` file. Unlike `index`, it has to take an argument `product_id` due to the variable in the route that we plan to attach to it. For now, we will simply return the `product_id` variable to test if it works.

```python
def product_detail(request, product_id):
    return HttpResponse(str(product_id))
```

Register your new view with `core/urls.py`.

```python
urlpatterns = [
    path("", views.index, name="index"),
    path("product/<int:product_id>", views.product_detail, name="product_detail"),
]
```

Note two things. First, the `name` parameter on each of these paths will be used as an identifier that we can use later in special helper functions. Second, in the first argument defining the path, enclose your variable and its data type in angle brackets.

Start the server. Navigate to `http://localhost:8000/product/1`. You should see the value "1" displayed on your web browser. This shows that your `product_detail` view function correctly captured the variable `product_id` as specified in `core/urls.py`.

## Use the database

Of course, when we specify a product ID, we are not actually interested in the product ID itself but rather the product row that shares the ID. Our next task is to display the _name_ of the product instead of just the ID.

We can now use the Product model we set up in a previous section. If you have not already done so, import the Product model in `core/views.py`.

```python
from .models import Product
```

Now, you can use the Product model in your view function. Let's simply render the name first without any HTML formatting to prove that we can do this.

```python
def product_detail(request, product_id):
    p = Product.objects.get(id=product_id)
    return HttpResponse(str(p.name))
```

When you visit `http://localhost:8000/product/1`, it should now render the name of the Product that has that ID (which is most likely "Americano").

## Link to the product pages

We are supposed to be able to directly visit the product pages from the index page. Revisit `core/templates/core/index.html`. If you have not already changed your `index` view function to use the database instead of the hardcoded list, replace it now with this code:

```python
def index(request):
    # Load the template
    template = loader.get_template("core/index.html")
    products = Product.objects.all()
    context = {
        "product_data": products
    }
    return HttpResponse(template.render(context, request))
```

We now have access to the product ID. We can use that to create "anchor" tags in HTML, which are more commonly known to users as links. Let's assume that you have converted the HTML in `index.html` to render a table instead of an unordered list.

```html
{% if product_data %}
    <table>
        <tr>
            <th>Name</th>
            <th>Price</th>
        </tr>
        {% for product_record in product_data %}
            <tr>
                <td>{{ product_record.name }}</td>
                <td>{{ product_record.price }}</td>
            </tr>
        {% endfor %}
    </table>
{% endif %}
```

Inside this for loop, we can actually wrap the name inside an anchor tag as such.

```html
{% if product_data %}
    <table>
        <tr>
            <th>Name</th>
            <th>Price</th>
        </tr>
        {% for product_record in product_data %}
            <tr>
                <td><a href="/product/{{ product_record.id }}">{{ product_record.name }}</a></td>
                <td>{{ product_record.price }}</td>
            </tr>
        {% endfor %}
    </table>
{% endif %}
```

It should be clear now how templating works. Anchor tags require you to specify a URL to which they point, and you can dynamically create those URLs using the data that you pass into a Django template.

Render your index page again. You should now see that the names of your products are hyperlinks. If you click on them, they should take you to the appropriate product page.

## Using HTML for the product page

Now, we want to convert the product page from simply rendering the name of the product into a proper HTML page. We will follow the workflow for adding a new HTML template:

1. Create a new file, `core/templates/core/product_detail.html`.
2. We will pass in an instance of the Product model as context. Add HTML code that renders the product name and the product price.
3. Add an anchor tag that takes the user back to the main products page.

Here is an example of what you might have in your `product_detail.html` file.

```html
<a href="/">Back to products</a>
{% if product %}
    <p>Product name: {{ product.name }}</p>
    <p>Product price: {{ product.price }}</p>
{% endif %}
```

Convert your `product_detail` view to render this template.

```python
def product_detail(request, product_id):
    template = loader.get_template("core/product_detail.html")
    p = Product.objects.get(id=product_id)
    context = {
        "product": p
    }
    return HttpResponse(template.render(context, request))
```

This should have felt similar to what you did in `index`, and more importantly, it should have felt _correct_. Reload your browser page and play around with the website. Congratulations, you have added a new feature to your Django web app.

Do you remember the admin site from the previous section? If you add a Product to the database through that site, it will actually render on your main site. This is because you are now rendering the view as a function of what's in your database. This is how web apps become dynamic.

## Checkpoint

Take two screenshots. The first screenshot should be of the index page (the page where all products are listed). The second screenshot should be of the Espresso page, assuming that you added Espresso to your products table in a previous section.

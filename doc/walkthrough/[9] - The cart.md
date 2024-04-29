# The cart

2024-04-29

This is where all our Django experience now comes together. In this section, we will add the ability for users to add products to their cart. This feature will require us to touch almost every part of Django so far, from views to templates to models to routing. However, the hope is that it is crystal clear how Django helps you implement the feature.

## Start with the model

Django becomes easier if you think of _everything_ as subordinate to the model. We should thus start here.

We only have two models right now: Product, which we made ourselves, and User, which we got out-of-the-box from Django. Pause to consider how you would model a shopping cart. Since this is a walkthrough, we will prescribe the model ourselves, but you should begin practicing data modeling now.

There will be one new model: CartItem. Each row of CartItem will represent one "line item" in the cart. A "line item" is a common concept in retail: it is composed of a product ID and a quantity. Together, many line items might make up a single transaction or, in this case, a single cart.

```python
...
from django.contrib.auth.models import User
...
class CartItem(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=False)
    product = models.ForeignKey(Product, on_delete=models.CASCADE, null=False)
    quantity = models.IntegerField()
    def __str__(self):
        return f'{self.quantity} of {self.product} (User: {self.user.username})'
```

For now, we will assume that each User only has one Cart, but that the Cart may contain many CartItems. If you have taken a basic data modeling class, you should understand this intuitively. If you haven't taken a basic data modeling class, try [this](https://www.youtube.com/watch?v=xsg9BDiwiJE) video.

A CartItem is very much related to both the User who put the items in their cart and the Product that the user put into their cart. A CartItem should thus have _references_ to these other two tables. This is what _foreign keys_ are for.

We will now register our model. Remember these steps:

- Make migrations
- Migrate
- Add to admin site (if desired)

Since this is a walkthrough, here are the commands needed to execute these steps.

```bash
python manage.py makemigrations core
python manage.py migrate
```

Then, in `core/admin.py`, add this code.

```python
from .models import CartItem
admin.site.register(CartItem)
```

Now, superusers can use the admin site to manage CartItems. Of course, we want users to be able to manage their own CartItems. We will tackle this in the next subsection.

## The user interface

How might a user add items to their cart? Never mind how a user might _want_ to do this; you can iterate on that later. For now, our only goal is to allow it at all.

It will be most straightforward for both us and the user to put another _form_ on each product detail page that lets a user specify a quantity to add to their cart. When the user clicks the form's submit button, the corresponding product and quantity should be added to their cart, and they should be redirected to the main index page.

We can achieve this in a similar way to how we built the login form. First, we can add a form to the product detail HTML template.

Change `core/templates/core/product_detail.html` to the following code.

```html
<h1>Product details</h1>
<a href="/">Back to products</a>
{% if product %}
    <p>Product name: {{ product.name }}</p>
    <p>Product price: {{ product.price }}</p>
{% endif %}

<h2>Add to cart</h2>
<form method="POST" action="#">
    {% csrf_token %}
    <input type="hidden" name="product_id" value="{{ product.id }}">
    <label for="quantity">Quantity</label>
    <input type="number" name="quantity" value="1">
    <input type="submit" value="Add to cart">
</form>
```

This form pattern should look very familiar. Note a little trick here. Since we want to add a CartItem, we will need both the quantity of the item to add and the ID of the item itself. We source the quantity from user input, but what about the product ID? We can add that ourselves by adding a "hidden" input field to our form with a pre-set value for the product ID. Now, the product ID will also be passed to the view when the form is submitted.

Of course, instead of sending its POST request to the login page, this form sends its post request to the `product_detail` view. We should now update this view to handle both GET requests for the normal page and POST requests for form submissions separately.

Update your code in the `product_detail` view to the following.

```python
@login_required
def product_detail(request, product_id):
    if request.method == 'GET':
        template = loader.get_template("core/product_detail.html")
        p = Product.objects.get(id=product_id)
        context = {
            "product": p
        }
        return HttpResponse(template.render(context, request))
    elif request.method == 'POST':
        submitted_quantity = request.POST['quantity']
        submitted_product_id = request.POST['product_id']
        product = Product.objects.get(id=submitted_product_id)
        user = request.user
        cart_item = CartItem(user=user, product=product, quantity=submitted_quantity)
        cart_item.save()
        return redirect('index')
```

This code is brittle, but it works on the "happy path," which is when the user does what you originally expected them to do. There are a few edge cases that will break this code. For example, if a user adds multiple CartItems with the same product, the CartItems will not automatically get merged. You will have to handle that logic in the `product_detail` view if you want to rectify it. For the purposes of this walkthrough, we will ignore that edge case.

Note that when you are making a CartItem, the `user` and the `product` are supposed to be the objects themselves. That is why we have to fetch a product using the product ID first.

Let's add one last feature. When a user is redirected to the index, there should be a message telling them that they have added a CartItem to their cart.

Add this code somewhere in your `core/templates/core/index.html` file.

```html
{% if messages %}
    <ul class="messages">
        {% for message in messages %}
            <li{% if messages.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}
```

Then, right after `cart_item.save()` in your `product_detail` file, add this code:

```python
messages.add_message(
    request,
    messages.INFO,
    f'Added {submitted_quantity} of {product.name} to your cart'
)
```

Users of your app should now be able to add items to their cart.

## Checkpoint

Take two screenshots:

- First, a screenshot of the Americano product detail screen.
- Second, the index screen _after_ adding an Americano to your cart.

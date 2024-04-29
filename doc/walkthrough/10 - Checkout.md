# Checkout

2024-04-29

What good is a cart if you can't check it out?

This section will put the final touches on the basic version of Digital Cafe. It won't look pretty, and it won't handle errors well, but if the users stay on the happy path, the app will work.

## More models

What happens when you check out a cart? It turns into a transaction. A user may have many transactions. Each transaction may have many line items.

We now need to add these two new models, Transaction and LineItem, to our `core` app. The process should feel very familiar.

```python
class Transaction(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=False)
    created_at = models.DateTimeField()

class LineItem(models.Model):
    transaction = models.ForeignKey(Transaction, on_delete=models.CASCADE, null=False)
    product = models.ForeignKey(Product, on_delete=models.CASCADE, null=False)
    quantity = models.IntegerField()
```

Remember, now you need to make migrations, migrate, and then register the models with the admin site if necessary.

## A checkout view

We will prescribe that the user will initiate checkout by going to a `checkout` route and view. The template will display a summary of all their CartItems and ask them if they want to check out. When the user checks out, the view will convert all the user's CartItems into LineItems and store these records as a Transaction.

It should feel like we're moving fast. It should also feel like this makes sense. Django's abstractions help you move fast while still making sense.

Make a new template, `core/templates/core/checkout.html`. We anticipate that we will pass in a list of CartItems and also the user, so we will make use of both of these items in our template.

```html
<h1>Checkout</h1>
<a href="/">Back to products</a>
<p>Hi, {{ user.username }}!</p>

{% if cart_items %}
    <table>
        <tr>
            <th>Product name</th>
            <th>Quantity</th>
        </tr>
        {% for cart_item in cart_items %}
            <tr>
                <td>{{ cart_item.product.name }}</td>
                <td>{{ cart_item.quantity }}</td>
            </tr>
        {% endfor %}
    </table>
    <form method="POST" action="#">
        {% csrf_token %}
        <input type="submit" value="Checkout">
    </form>
{% else %}
    <p>No cart items.</p>
{% endif %}
```

Make a new view in `core/views.py`, `checkout`, to handle this. For now, let's just render the template; we will get back to the POST handling later.

```python
@login_required
def checkout(request):
    template = loader.get_template("core/checkout.html")
    cart_items = CartItem.objects.filter(user=request.user)
    context = {
        'cart_items': list(cart_items),
    }
    return HttpResponse(template.render(context, request))
```

Register this view with `core/urls.py`.

```python
path("checkout", views.checkout, name="checkout"),
```

Finally, add a link to this route to your `index` template.

```html
<a href="/checkout">Go to check out</a>
```

You can now visit your checkout page. Everything should now feel like it's falling into place.

Let's now implement the actual checkout. Import the Python `datetime` module and your new models.

```python
import datetime as dt
from .models import Transaction
from .models import LineItem
```

Now, when the user checks out, convert all their CartItems to LineItems wrapped in a Transaction.

```python
@login_required
def checkout(request):
    if request.method == 'GET':
        template = loader.get_template("core/checkout.html")
        cart_items = CartItem.objects.filter(user=request.user)
        context = {
            'cart_items': list(cart_items),
        }
        return HttpResponse(template.render(context, request))
    elif request.method == 'POST':
        cart_items = CartItem.objects.filter(user=request.user)
        # Create new transaction
        created_at = dt.datetime.now(tz=dt.timezone.utc)
        transaction = Transaction(user=request.user, created_at=created_at)
        transaction.save()
        for cart_item in cart_items:
            line_item = LineItem(
                transaction=transaction,
                product=cart_item.product,
                quantity=cart_item.quantity,
            )
            line_item.save()
            cart_item.delete()
        messages.add_message(request, messages.INFO, f'Thank you for your purchase!')
        return redirect('index')
```

Of course, a user may want to check their transaction history. This is another feature, but it is crystal clear how you would implement this.

1. Do we have the models we need? In this case, yes, Transaction and LineItem suffice.
2. Do we have the views we need? Not yet. We can create them.

We leave this last feature to you to complete on your own. It's the same process, again and again. Once you get used to it, you can go incredibly fast.

## Checkpoint

Take one screenshot of the transaction history page.

Congratulations -- if you have followed along, you have created Digital Cafe.

# Users

2024-04-29

We now arrive at a large problem in developing web apps. Most web apps (as opposed to web sites) require some sort of interaction from users. Most web apps also need to track which specific users performed which specific actions. Some web apps need to prevent users from taking actions if they are not logged in.

Django's answer to the question of users is to provide a User model by default. Django's User model comes with an authentication framework, so you will not need to design one from scratch. The User model is also extensible: if you need to add additional fields to a User that aren't included by default, Django recommends that you create a model specific to your app and then join it one-to-one with the User model.

By default, the Django user authentication method is a "native login." This means that Django users usually login directly via a username and password. It has become very common for web apps to offer "federated login" through providers like Google and Facebook. While Django does support adding these login methods, for simplicity, we will not attempt to extend the native login in this walkthrough.

## Protecting your existing pages

Let's do this backwards. The _first_ thing we will do is protect our two existing views, `index` and `product_detail`, against users who are not logged in.

Import the `login_requred` decorator from `django.contrib.auth.decorators`. A "decorator" is something that you add to the top of a function to modify its behavior. We will not be creating decorators in this walkthrough, but we will be using them.

```python
from django.contrib.auth.decorators import login_required
```

Then, on top of both of your views, add the decorator.

```python
@login_required
def index(request):
    ...

@login_required
def product_detail(request, product_id):
    ...
```

Try to reload the page. You should expect there to be an error. If you are not getting an error, try logging out of the admin site, which you may be logged into. Once you see an error, you may proceed.

## The login view

It naturally follows that your `core` app needs to be able to log users in. This is usually done by creating a view specifically for login. By default, Django's `login_required` decorator tries to redirect unauthenticated users to `accounts/login`, but you can customize this by changing `settings.LOGIN_URL` or by providing a `login_url` keyword argument to the decorator. We will leave this as is for now.

Django does provide built-in views specifically for authentication, but for the purposes of this walkthrough, we will create our own view. Create a new function in `core/views.py` called `login_view`. Do not call it `login` to avoid naming conflicts with functions we will import later.

```python
def login_view(request):
    ...
```

This is where we will introduce an important concept in web apps: forms. A "form" is an HTML element that allows users to send input to the server. Let's create our form now in the `login_view` view. Create a new file `core/templates/core/login_view.html` and add this HTML code.

```html
<h1>Login</h1>
{% if messages %}
    <ul class="messages">
        {% for message in messages %}
            <li{% if messages.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
        {% endfor %}
    </ul>
{% endif %}
<form method="POST" action="#">
    {% csrf_token %}
    <label for="username">Username</label>
    <input type="text" name="username">
    <label for="password">Password</label>
    <input type="password" name="password">
    <input type="submit" value="Login">
</form>
```

Note that this form has two new elements.

- The `{% if messages %}` block fetches what are usually called "flash" messages from Django. Flash messages are pieces of information that you might want to display to a user if some event (such as a failed login) occurs. You will see how this is used later.
- The `{% csrf_token %}` line here is a security feature that protects your route. It adds a secret, hidden field to the form that defends against the case where someone sends a POST request to your endpoint without actually trying to login.

Change the login view to render this new template.

```python
def login_view(request):
    template = loader.get_template("core/login_view.html")
    context = {}
    return HttpResponse(template.render(context, request))
```

Register the login view with `core/urls.py`.

```python
# Unfortunately, in this case, the trailing slash is REQUIRED.
path("accounts/login/", views.login_view, name="login_view"),
```

Now, when you try to visit any page while you are not logged in, you should get redirected to a login form.

Of course, our login form doesn't actually do anything right now. All it does when we press "Login" is send an HTTP POST request (as opposed to a normal HTTP GET request) to the `accounts/login/` route on our Django server. (The form's "method" property tells is to send a POST request, and the form's "action" property tells it to send to the same URL as the current URL.) We will need to modify our view to handle different types of requests.

Pause to consider the different types of requests that might hit `accounts/login/`. First, there is the normal GET request that comes when a user simply visits the site on their browser. In this case, we simply want to render the login HTML page. Second, there is the POST request that comes when a user fills out the login form and presses "Login." In this case, we want to check if their credentials are correct. If they are, we want to tell the Django server to log the user in, and we also want to redirect them to a page.

Import these new Django functions in `core/views.py`.

```python
from django.shortcuts import redirect
from django.contrib.auth import (
    login, logout, authenticate
)
from django.contrib import messages
```

Then, implement different codepaths for whether your view got a GET request or a POST request.

```python
def login_view(request):
    if request.method == 'GET':
        template = loader.get_template("core/login_view.html")
        context = {}
        return HttpResponse(template.render(context, request))
    elif request.method == 'POST':
        submitted_username = request.POST['username']
        submitted_password = request.POST['password']
        user_object = authenticate(
            username=submitted_username,
            password=submitted_password
        )
        if user_object is None:
            messages.add_message(request, messages.INFO, 'Invalid login.')
            return redirect(request.path_info)
        login(request, user_object)
        return redirect('index')
```

Note a few things here.

- If the request was a POST request, you can access the key-value pairs submitted with the POST request in the `request.POST` object. In this case, since the POST request is sent by a form, the key-value pairs in the `reuqest.POST` object will correspond to the form fields. That is why we can expect a `username` and `password` key: the form itself had fields named `username` and `password`.
- The `authenticate` method here is supposed to return a User object if the login is successful and `None` if the login failed. If there is no User object, we want to simply redirect the user back to the login page.
- If the login failed, we also use the `messages` function to load a new message to be displayed in the flash messages upon the page redirect. Try failing a login intentionally to see this in action.
- The last line of the function, `return redirect('index')`, tells Django to redirect to the view named `index`. You can see in `core/urls.py` that we named one of our paths `index`. That is the view we want to redirect to.

You can now try to log into your site using the superuser you created in a previous section. Since users are shared between the admin site and the main site, this should work. If you don't remember your admin credentials, you can run `python manage.py createsuperuser` to try again. You can also manage users directly via the admin site.

Once you have implemented login, you can use the user that is currently logged in in your views. Try to change the index view to greet the user by username. You can get the current user from the request object like so.

```python
@login_required
def index(request):
    # Load the template
    template = loader.get_template("core/index.html")
    products = Product.objects.all()
    context = {
        "user": request.user,
        "product_data": products,
    }
    return HttpResponse(template.render(context, request))
```

Then, add this line somewhere at the top of `core/templates/core/index.html`.

```html
<p>Hi, {{ user.username }}!</p>
```

## Checkpoint

Take two screenshots:

- First, a screenshot of a failed login.
- Second, a screenshot of the new index page, greeting your user by username.

# Digital Cafe

2024-04-27

Welcome to Digital Cafe. This is the first installment of an introduction to web application programming. Our goal in this walkthrough is to teach you the very basics of how a "web application," as opposed to a mere "web page," works.

You can think of a web application (henceforth "web app") as a web page that stores data. This sounds like a simple addition, but it isn't. Having to manage data is where almost all the complexity of modern web development comes from. Thankfully, special programming libraries called _frameworks_ are available to help us design and run such applications easily.

In this walkthrough, we will be building a simple e-commerce application for a company that sells coffee. Users should be able to browse products, add products to their cart, and check out their orders. There are a lot of requirements in that one sentence.

We will use the Django framework to build our web app. The repository you are in right now is a snapshot of the code we had after writing documents 0001 through 0010. Note that you might not be able to download and run this repository directly, because we have excluded a settings file from the git repository. It shouldn't matter: you are meant to build Digital Cafe from scratch in your own repository. Use this repo as a guide, not as something to copy in its entirety.

## Objectives

After completing this walkthrough, you should be able to:

1. Build a simple web app from scratch using Django,
2. Reason about how to add a new feature to a Django web app.

We have adapted the official Django tutorial flow around the Digital Cafe web app concept instead of the Polls web app concept they use on their website.

It is beyond the scope of Digital Cafe to teach you about how "single-page apps" work. Django is excellent at producing so-called "multi-page apps", which reload the web browser every time you visit a new page. There is nothing wrong with sticking to this simple model as a beginner.

## Prework

### Installations

Bear with us, there is some setup that you need to do before running a Django project.

#### Mac

Please install the following on your computer:

- Python 3.11

Consider using Homebrew to install Python 3.11. Homebrew is a "package manager" that lets you install software from your terminal.

If you have Homebrew, run this command:

```zsh
brew install python@3.11
```

Refresh your shell:

```zsh
exec $SHELL
```

Verify that Python 3.11 has been installed:

```zsh
python3.11 --version
```

If you get a response with Python's version set to 3.11.x, you may proceed.

Make an empty directory and set it as your new working directory. This new directory will serve as the "root" of your entire Django project. We will call this directory `digitalcaferoot` for this tutorial.

```zsh
mkdir digitalcaferoot && cd digitalcaferoot
```

Make a Python virtual environment. This is where we will install our Python libraries.

```zsh
python3.11 -m venv env
source env/bin/activate
```

Your terminal prompt should now have a `(env)` prepended to it. This means that your `python` command now points to the virtual environment. Verify this by checking where `python` points to:

```zsh
which python
```

If you get a response pointing to a file inside the `env/` directory, you may proceed.

This is the last piece of setup for now. Install the `django` library.

```zsh
pip install django
```

Once this finishes, check if Django was installed:

```zsh
django-admin
```

If you get a help message, you may proceed.

#### Windows

Please install the following on your computer. You may get this software from Python's official website.

- Python 3.11

**Pay attention to the installer**. When it prompts you to "add Python to your PATH", tick the checkbox. If you missed this, open the installer again, uninstall Python, then reinstall Python with the checkbox ticked.

Open a PowerShell terminal. Check if Python 3.11 was installed properly by running:

```powershell
python --version
```

If you get a response with Python's version set to 3.11.x, you may proceed.

Make an empty directory and set it as your new working directory. This new directory will serve as the "root" of your entire Django project. We will call this directory `digitalcaferoot` for this tutorial.

```powershell
New-Item digitalcaferoot -ItemType Directory
Set-Location digitalcaferoot
```

Make a Python virtual environment. This is where we will install our Python libraries.

```powershell
python -m venv env
```

Since we are on Windows, there is a chance that we will need to allow external scripts to run. Open a new PowerShell window **as administrator** and run the following command:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Re-open a normal PowerShell window and navigate back to your `digitalcaferoot` directory. Now, you can activate your Python virtual environment:

```powershell
.\env\Scripts\activate.ps1
```

Your terminal prompt should now have a `(env)` prepended to it. This means that your `python` command now points to the virtual environment. Verify this by checking where `python` points to:

```powershell
Get-Command python
```
If you get a response pointing to a file inside the `env/` directory, you may proceed.

This is the last piece of setup for now. Install the `django` library.

```powershell
pip install django
```

Once this finishes, check if Django was installed:

```powershell
django-admin
```

If you get a help message, you may proceed.

### Theory

This part is optional, but highly helpful. A web app needs four parts to function:

- Routing HTTP requests based on their content and path ("routing")
- Storing data in a database and making it accessible to the rest of the app ("models")
- Rendering HTML based on data that may vary ("views")
- The glue logic between models and views ("controllers")

Web frameworks like Django (the framework we're using now), Flask, Ruby on Rails, and more all use these four basic concepts.

I wrote about these in this series of articles:

- https://joeilagan.com/article/2024-itm-web-apps-1
- https://joeilagan.com/article/2024-itm-web-apps-2
- https://joeilagan.com/article/2024-itm-web-apps-3

## Checkpoint

Please take a screenshot of your Terminal/PowerShell window after running the `django-admin` command.

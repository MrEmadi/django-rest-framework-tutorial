**Django REST Framework Tutorial**

- **Reference: [here](https://www.django-rest-framework.org)**
- **OS: Windows**

# Setting up a new environment

Before we do anything else we'll **create a new virtual environment**, using **`venv`**. 
This will make sure our package configuration is kept nicely **isolated** from any other projects we're working on.
```shell
py -m venv env
```
```shell
# + CategoryInfo          : SecurityError: (:) [], PSSecurityException
# + FullyQualifiedErrorId : UnauthorizedAccess

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```
```shell
.\env\Scripts\activate
```
Now that we're inside a virtual environment, we can install **our package requirements**.
```shell
py -m pip install --upgrade pip
```
```shell
pip install django
```
```shell
pip install djangorestframework
```
```shell
pip install pygments
 
# We'll be using this for the code highlighting
```
```shell
# Check the installation of packages in the env:

pip freeze
```
Note: **To exit the virtual environment** at any time, just type **`deactivate`**.

# Getting started

To get started, let's **create a new project** to work with.
```shell
django-admin startproject tutorial

# tutorial: project name
```
```shell
cd tutorial
```
Once that's done we can create an app that we'll use **to create a simple Web API**.
```shell
py manage.py startapp snippets

# snippets: app name
```
We'll need to **add our new `snippets` app** and the `rest_framework` app to `INSTALLED_APPS`. 
Let's **edit the `tutorial/settings.py` file** (**See `my code`**).

# Creating a model to work with

For the purposes of this tutorial we're going to start by **creating a simple `Snippet` model** that is used to store code snippets. 
Go ahead and **edit the `snippets/models.py` file** (**See `my code`**).

We'll also need **to create an initial migration** for our snippet model, and **sync the database** for the first time.
```shell
py manage.py makemigrations snippets
```
```shell
py manage.py migrate snippets
```

# Creating a Serializer class

The first thing we need to get started on our Web API is to provide a way of **serializing and deserializing the snippet instances** into representations such as **json**. 
We can do this by declaring serializers that work very similar to Django's forms. 
**Create a file in the snippets directory named `serializers.py`** and add the codes (**See `my code`**).

The first part of the `serializer` class defines the fields that **get serialized/deserialized**. 
The `create()` and `update()` methods define how fully fledged instances are created or modified when calling `serializer.save()`

A `serializer` class is very similar to a Django Form class, and includes similar validation flags on the various fields, such as **required**, **max_length** and **default**.

The field flags can also **control how the serializer should be displayed in certain circumstances**, such as when rendering to HTML. 
The `{'base_template': 'textarea.html'}` flag is equivalent to using `widget=widgets.Textarea` on a Django Form class. 
This is particularly useful for controlling **how the browsable API should be displayed**, as we'll see later in the tutorial.

We can actually also save ourselves some time by using the `ModelSerializer` class, as we'll see later, but for now we'll keep our serializer definition explicit.

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
Let's **edit the `tutorial/settings.py` file** (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/216a7127aef8cf07f80ce494492e9a4745759487#diff-050a23520aa24d9e65854375089f1a01bfec8034d7ee08c99908f181f15d9905)**).

# Creating a model to work with

For the purposes of this tutorial we're going to start by **creating a simple `Snippet` model** that is used to store code snippets. 
Go ahead and **edit the `snippets/models.py` file** (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/10d7720f2603e0a128313d6c353b93c3b3acfedb#diff-f0f7ef9b28e5a8b154f81c098a64de0c40e7017ece5d6b071f62842e25271c0c)**).

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
**Create a file in the snippets directory named `serializers.py`** and add the codes ([**codes**](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/a300942b65b946cb14608b73f2c8390cc2c404af#diff-5c7b30f359b4fbd824cb0b4654a642b0ce23724f6e91c6b7d8e14f1c52148096)).

The first part of the `serializer` class defines the fields that **get serialized/deserialized**. 
The `create()` and `update()` methods define how fully fledged instances are created or modified when calling `serializer.save()`

A `serializer` class is very similar to a Django Form class, and includes similar validation flags on the various fields, such as **required**, **max_length** and **default**.

The field flags can also **control how the serializer should be displayed in certain circumstances**, such as when rendering to HTML. 
The `{'base_template': 'textarea.html'}` flag is equivalent to using `widget=widgets.Textarea` on a Django Form class. 
This is particularly useful for controlling **how the browsable API should be displayed**, as we'll see later in the tutorial.

We can actually also save ourselves some time by using the `ModelSerializer` class, as we'll see later, but for now we'll keep our serializer definition explicit.

# Working with Serializers

Before we go any further we'll familiarize ourselves with using our new Serializer class. Let's drop into the Django shell.
```shell
py manage.py shell
```
Okay, once we've got a few imports out of the way, let's create a couple of code snippets to work with.
```text
>>> from snippets.models import Snippet
>>> from snippets.serializers import SnippetSerializer
>>> from rest_framework.renderers import JSONRenderer
>>> from rest_framework.parsers import JSONParser
>>> snippet = Snippet(code='foo = "bar"\n')
>>> snippet.save()
>>> snippet = Snippet(code='print("hello, world")\n')
>>> snippet.save()                                    
>>> serializer = SnippetSerializer(snippet)
>>> serializer.data
{'id': 2, 'title': '', 'code': 'print("hello, world")\n', 'linenos': False, 'language': 'python', 'style': 'friendly'}
```
At this point we've translated the model instance into Python native datatypes. 
To finalize the serialization process we render the data into **`json`**.
```text
>>> content = JSONRenderer().render(serializer.data)
>>> content
b'{"id":2,"title":"","code":"print(\\"hello, world\\")\\n","linenos":false,"language":"python","style":"friendly"}'
```
**Deserialization** is similar. First we parse a stream into Python native datatypes, 
then we restore those native datatypes into a fully populated object instance.
```text
>>> import io
>>> stream = io.BytesIO(content)
>>> data = JSONParser().parse(stream)
>>> serializer = SnippetSerializer(data=data)
>>> serializer.is_valid()
True
>>> serializer.validated_data
{'title': '', 'code': 'print("hello, world")', 'linenos': False, 'language': 'python', 'style': 'friendly'}
>>> serializer.save()
<Snippet: Snippet object (3)>
```
Notice how similar the API is to working with forms. 
The similarity should become even more apparent when we start writing views that use our serializer.

We can also serialize query sets instead of model instances. 
To do so we simply add a `many=True` flag to the serializer arguments.
```text
>>> serializer = SnippetSerializer(Snippet.objects.all(), many=True)
>>> serializer.data
[{'id': 1, 'title': '', 'code': 'foo = "bar"\n', 'linenos': False, 'language': 'pytho
n', 'style': 'friendly'}, {'id': 2, 'title': '', 'code': 'print("hello, world")\n', '
linenos': False, 'language': 'python', 'style': 'friendly'}, {'id': 3, 'title': '', '
code': 'print("hello, world")', 'linenos': False, 'language': 'python', 'style': 'friendly'}]
```

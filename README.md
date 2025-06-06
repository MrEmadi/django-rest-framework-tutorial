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

# Using ModelSerializers

Our `SnippetSerializer` class is replicating a lot of information that's also contained in the `Snippet` model. 
It would be nice if we could keep our code a bit more concise.

In the same way that Django provides both `Form` classes and `ModelForm` classes, REST framework includes both `Serializer` classes, and `ModelSerializer` classes.

Let's look at refactoring our `serializer` using the `ModelSerializer` class. 
Open the file `snippets/serializers.py` again, and replace the `SnippetSerializer` class with the code (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/b1af180c81bda9c8ef1fd0bfcd97413b6c67bdf2#diff-5c7b30f359b4fbd824cb0b4654a642b0ce23724f6e91c6b7d8e14f1c52148096)**).

One nice property that serializers have is that **you can inspect all the fields in a serializer instance**, by printing its representation. 
Open the **Django shell with `py manage.py shell`**, then try the following:
```text
>>> from snippets.serializers import SnippetSerializer
>>> serializer = SnippetSerializer()
>>> print(repr(serializer))
SnippetSerializer():
    id = IntegerField(label='ID', read_only=True)
    title = CharField(allow_blank=True, max_length=100, required=False)
    code = CharField(style={'base_template': 'textarea.html'})
    linenos = BooleanField(required=False)
    language = ChoiceField(choices=[('abap', 'ABAP'), ('abnf', 'ABNF'), ('actionscrip
t', 'ActionScript'), ('actionscript3', 'ActionScript 3'), ('ada', 'Ada'), ('adl', 'AD
L'), ('agda', 'Agda'), ...], required=False)
    style = ChoiceField(choices=[('abap', 'abap'), ('algol', 'algol'), ('algol_nu', '
algol_nu'), ('arduino', 'arduino'), ...], required=False)
```
It's important to remember that `ModelSerializer` classes don't do anything particularly magical, they are simply **a shortcut for creating `serializer` classes**:
- An automatically determined set of fields.
- Simple default implementations for the `create()` and `update()` methods.

# Writing regular Django views using our Serializer

Let's see how we can write some API views using our new `Serializer` class. 
For the moment we won't use any of REST framework's other features, we'll just write the views as regular Django views.

Edit the `snippets/views.py` file, and add the code. 
The root of our API is going to be a view that supports listing all the existing snippets, or creating a new snippet.
Note that because we want to be able to POST to this view from clients that won't have a CSRF token we need to mark the view as `csrf_exempt`. 
This isn't something that you'd normally want to do, and REST framework views actually use more sensible behavior than this, but it'll do for our purposes right now.
We'll also need a view which corresponds to an individual snippet, and can be used to retrieve, update or delete the snippet (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/57c6694f67dd817f33f92152c318d2a545979b5f#diff-a4dc166130ff34f7334d219587a451162dbba6a006e8b1297d4e1c2b0c2f300c)**).

Finally, we need to wire these views up. Create the `snippets/urls.py` file (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/57c6694f67dd817f33f92152c318d2a545979b5f#diff-78398e5f071186ce247a6b02761caa27e9136a8c98834edff9b525148486bef1)**).

We also need to wire up the root urlconf, in the `tutorial/urls.py` file, to include our snippet app's URLs (**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/57c6694f67dd817f33f92152c318d2a545979b5f#diff-8ddbfb7c93b4b0e6d701ffc6a001259d6fa58ae81c1e2771d31c34c8e3a03c65)**).

It's worth noting that there are a couple of edge cases we're not dealing with properly at the moment. 
If we send malformed `json`, or if a request is made with a method that the view doesn't handle, then **we'll end up with a `500 server error` response**. 
Still, this'll do for now.

# Testing our first attempt at a Web API

Now we can **start up a sample server that serves our snippets**. Quit out of the shell:
```text
>>> quit()
```
Start up Django's development server:
```shell
py manage.py runserver

# Django version 5.2.2, using settings 'tutorial.settings'
# Starting development server at http://127.0.0.1:8000/
# Quit the server with CTRL-BREAK.
```
In another terminal window, we can test the server.
We can test our API using `curl` or `httpie`. 
`Httpie` is a user-friendly http client written in Python. 
Let's install that.
You can install `httpie` using `pip`:
```shell
pip install httpie
```
Finally, we can get a list of all the snippets:
```shell
http GET http://127.0.0.1:8000/snippets/ --unsorted

#HTTP/1.1 200 OK
#Date: Sat, 07 Jun 2025 09:31:08 GMT
#Server: WSGIServer/0.2 CPython/3.13.3
#Content-Type: application/json
#X-Frame-Options: DENY
#Content-Length: 354
#X-Content-Type-Options: nosniff
#Referrer-Policy: same-origin
#Cross-Origin-Opener-Policy: same-origin
# 
# [
#     {
#         "id": 1,
#         "title": "",
#         "code": "foo = \"bar\"\n",
#         "linenos": false,
#         "language": "python",
#         "style": "friendly"                                                          
#     },
#     {
#         "id": 2,
#         "title": "",
#         "code": "print(\"hello, world\")\n",
#         "linenos": false,
#         "language": "python",
#         "style": "friendly"                                                          
#     },
#     {
#         "id": 3,
#         "title": "",
#         "code": "print(\"hello, world\")",
#         "linenos": false,
#         "language": "python",
#         "style": "friendly"                                                          
#     }
# ]
```
Or we can get a particular snippet by referencing its `id`:
```shell
http GET http://127.0.0.1:8000/snippets/2/ --unsorted

# HTTP/1.1 200 OK
# Date: Sat, 07 Jun 2025 09:33:57 GMT
# Server: WSGIServer/0.2 CPython/3.13.3
# Content-Type: application/json
# X-Frame-Options: DENY
# Content-Length: 120
# X-Content-Type-Options: nosniff
# Referrer-Policy: same-origin
# Cross-Origin-Opener-Policy: same-origin
# 
# {
#     "id": 2,
#     "title": "",
#     "code": "print(\"hello, world\")\n",
#     "linenos": false,
#     "language": "python",
#     "style": "friendly"                                                              
# }
```
Similarly, you can have the same JSON displayed by visiting these URLs in a web browser.

# Request and Response objects

**REST framework** introduces a `Request` object that extends the regular `HttpRequest`,
and provides more flexible request parsing. 
The core functionality of the `Request` object is the `request.data` attribute,
which is similar to `request.POST`, but more useful for working with Web APIs.
- **`request.POST`**: Only handles form data.
  Only works for `POST` method.
- **`request.data`**: Handles arbitrary data.
  Works for `POST`, `PUT` and `PATCH` methods.

**REST framework** also introduces a `Response` object, which is a type of `TemplateResponse` that takes unrendered content and uses content negotiation to determine the correct content type to return to the client.
- **`return Response(data)`**: Renders to content type as requested by the client.

Using numeric HTTP status codes in your views doesn't always make for obvious reading,
and it's easy to not notice if you get an error code wrong.
**REST framework** provides more explicit identifiers for each status code,
such as `HTTP_400_BAD_REQUEST` in the `status` module.
It's a good idea to use these throughout rather than using numeric identifiers.

**REST framework** provides two wrappers you can use to write API views.
- The `@api_view` decorator for working with function-based views.
- The `APIView` class for working with class-based views.

These wrappers provide a few bits of functionality such as making sure you receive `Request` instances in your view,
  and adding context to `Response` objects so that content negotiation can be performed.

The wrappers also provide behavior such as returning `405 Method Not Allowed` responses when appropriate,
and handling any `ParseError` exceptions that occur when accessing `request.data` with malformed input.

# Pulling it all together

Okay, let's go ahead and start using these new components to refactor our views slightly.
Our instance view is an improvement over the previous example.
It's a little more concise, and the code now feels very similar to if we were working with the Forms API.
We're also using named status codes, which makes the response meanings more obvious.
Here is the view for an individual snippet, in the `views.py` module.
This should all feel very familiar—it is not a lot different from working with regular Django views.
Notice that we're no longer explicitly tying our requests or responses to a given content type.
`request.data` can handle incoming `json` requests, but it can also handle other formats.
Similarly, we're returning response objects with data,
but allowing REST framework to render the response into the correct content type for us
(**[codes](https://github.com/MrEmadi/django-rest-framework-tutorial/commit/d1a077f63e397314bc3bb2a57e66f7138aead1a4#diff-a4dc166130ff34f7334d219587a451162dbba6a006e8b1297d4e1c2b0c2f300c)**).

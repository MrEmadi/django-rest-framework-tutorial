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
Let's **edit the `tutorial/settings.py` file** (**my code**).

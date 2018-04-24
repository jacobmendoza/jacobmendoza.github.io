---
layout: post
title:  "Virtual environment for Python"
date:   2015-10-30 17:40:57 +0000
categories: TECHNOTES VENV PYTHON
---

# Setting a virtual environment to work with specific versions of the Python interpreter and packages
This assumes that you know the location of the interpreter (or it's in your path variable as in this case). Specifically for this example, we are using Python 3.4. The last parameter creates the environment in the same folder.
```
virtualenv -p python3.4 ./
```
Resulting in:
```
Running virtualenv with interpreter /usr/local/bin/python3.4
Using base prefix '/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4'
New python executable in ./bin/python3.4
Also creating executable in ./bin/python
Installing setuptools, pip...done.
```
## Activation
Now you need to activate the virtual environment:
```
source bin/activate
```
## Packages
Running now Python will result in using the specified version (3.4). To install dependencies you can use pip. All the operations will be made inside the venv:
```
pip install -U textblob
pip install beautifulsoup4
pip install Flask
```
## Deactivation
You can always leave the venv with the following shell function provided by it:
```
deactivate
```

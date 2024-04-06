---
title: "Post: When you encounter the error 'error: invalid command 'bdist_wheel' after creating a venv' in Python"
last_modified_at: 2021-05-19T23:59:32+0900
categories:
  - Blog
tags:
  - Python
  - venv
  - error
---

[Source](https://stackoverflow.com/a/44862371)

Often, after creating and activating a venv, one might encounter the following error.
```error: invalid command 'bdist_wheel'```

# tl;dr

```pip install wheel```
After doing so,
```python setup.py bdist_wheel```
It is said that it will work.

For me, just running ```pip install wheel``` works fine.

# Why?

Even if you install wheel in the global Python, when creating a venv without giving options, only a minimum number of packages are installed.

Right after creating the venv, if you run ```pip list```, it shows like below...
```
Package       Version
------------- -------
pip           20.0.2 
pkg-resources 0.0.0  
setuptools    44.0.0
```

As you can see, it is evident that wheel is not included.

If you want to inherit global packages and create a venv, you can initialize it with the ```--system-site-packages``` option like below.
```python -m venv {env_dir} --system-site-packages```
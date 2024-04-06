---
title: "Post: when a specific package installation is extremely slow on a VM or cloud instance"
last_modified_at: 2021-12-19T17:57:25+0900
categories:
  - Blog
tags:
  - Python
  - pip
---
# tl;dr
```pip install --upgrade pip```

# Because of the pip version...?

While working on development outside of regular tasks, I encountered an issue where installing firebase-admin via pip on the cloud instance I was hosting suddenly crashed the instance...

Naturally, I couldn't imagine there being a problem with a simple installation, and it was only after noticing the Github action runner was dead that I realized something was amiss...

Upon closer inspection, it turned out that the installation of grpcio, a dependency of firebase-admin, was particularly slow, and it would stop responding...

As a result of searching, I found a response to an issue similar to [this one](https://github.com/grpc/grpc/issues/22815#issuecomment-621984140).

It stated that the manylinux2010 wheel binaries for grpcio had been changed to require pip version 19 or above.

Somehow, when trying to deploy an old project to an instance, I ended up using an older version of python in the venv, and using an old version of pip led me to this problem.

Isn't it supposed to throw a warning or an error...?! Anyways, I'm not sure if it ends well without slowing down in a normal environment, but it seems to cause problems in a virtual machine with limited resources. My guess is that it crashes during build because it can't use the pre-built binary wheels, but I'm not sure about the exact underlying behavior.

Nonetheless, after upgrading the pip version, the installation which used to take so long that it even killed the instance, finished in just one second...
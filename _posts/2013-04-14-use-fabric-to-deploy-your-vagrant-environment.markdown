---
layout: post
title: "use fabric to deploy your vagrant environment"
date: 2013-04-14 23:33
comments: true
categories: 
---

Vagrant is a great tool that lets you create and destroy virtualbox vms.
I will use fabric to the deployment stack, I will do all the fine grained 
post-install and post-configure tasks with it.

Below is step by step guide:
<!--more -->
* 1. Install virtual box in your ubuntu
* 2. Install vagrant in your ubuntu
* 3. Install fabric in your ubuntu
* 4. Add default box in your system, it may take some minutes
```
> vagrant add box precise32 http://files.vagrantup.com/precise32.box
```

* 5. Start up your default vagrant
* 6. Touch fabfile.py in your current directory with below contents:
```python
from fabric.api import *
def vagrant():
    env.user='vagrant'
    env.hosts=['127.0.0.1:2222']
    result = local('vagrant ssh-config | grep IdentityFile', capture=True)
    env.key_filename = result.split()[1].trip('\"')
def uname():
    run('uname -a')
```

* 7. Input ```fab vagrant uname``` in  your terminal, then you will get some messages.


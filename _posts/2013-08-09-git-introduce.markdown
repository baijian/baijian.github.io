---
layout: post
title: "git-introduce"
date: 2013-08-09 12:00
comments: true
categories: 
toc: true
---

### what is git

Git is not subversion, git is a content-addressable filesystem which is a very powerful tool that you
can easily use as more than just a VCS.

As of Linus Torvalds's opnion Subversion has been the most pointless project ever started, 
If you like using CVS, you should in some kind of mental institution or somewhere else.

Now [github](https://github.com) has been the best and cool SCM system, it is very very popular,
and if your are a programer, you will be shamed of not having a github account.

<!-- more -->

### some features of git

* A project is a git repository and every client have all of the codes.

* Fast and simple local branch, you add a branch means just add a refs.

* Develop your project without network, as we know it is distributed.

### How to install?

[Install Guide](https://help.github.com/articles/set-up-git)

* CentOS

    $ yum install git

* Ubuntu

    $ sudo apt-get install git

* Mac

    $ brew install git

* Windows ?

    Oh, please, do not use windows any more ^-^

### How to config?

$ ssh-keygen -t rsa -C "xxx#xxmail.com"

$ git config --global user.name "xxx"

$ git config --global user.email "xxx#xxmail.com"

$ git config --global color.ui true

### Init your project

$ mkdir demo & cd demo

$ git init 

$ echo "test" > test.txt

$ git add test.txt

$ git commit -m "add test file"

### Push your code to the central server

$ git remote add origin git@xxserver.com:<user>/<repo>.git

$ git push origin master

### Update your project and modify

$ git pull origin master

$ git status

$ echo "test2" > test2.txt

$ git commit -m "add test2 file"

$ git push origin master

### Exclude some files and view the modification history

To Exclude files you do not want to stage just add a *.gitignore* file 
to your repository, and input *git log --graph* command to see modification 
history of  your repository.

### How to do teamwork with git?

Because of the low cost of adding a branch in git, so I choose branch in teamwork,
*master* branch is ready to release to the app server, and any other branch is the 
develop branch.

If you want to release a new function, just merge the branch to the *mater* branch after it 
passes the test.

### How git works?

Git have two types of commands, one is *plumbing commands* as we always use, and 
the other one is *porcelain commands* as we not always use.

You must want to know how contents are stored in git, first you should understand
objects in git. There are four types of objects, *tree*,*blob*,*commit*,
*tag*; for example how *blob* are stored in git database? 

The contents of blob objects have *content header* and *content body*, the contents 
will be compressed and write to a file which name is releated to the *SHA* hash 
of contents. The file is placed in *.git/objects* directory, and the sub-directory
name is the first two character of the *SHA* hash.Blew is the code of ruby.

```ruby
header = blob + ' ' + content.size + \0
body = content
new_content = header + body
sha = Digest::SHA1.hexdigest(new_content)
compressed = zlib::deflate(new_content)
path = ".git/objects/**/*****..."
File.open(path, 'w') {
    |f| f.write(compressed)
}
```

### The end

At last , recommend a website to learn git: [ProGit](http://git-scm.com/book)

Or if git-scm was killed by *GxFxW*, you can view from [Github-ProGit](https://github.com/progit/progit),
Ofcourse it is write in markdown and have almost all languages.


---
layout: post
title: "build-your-site-with-jekyll"
date: 2014-03-08 23:42:55
comments: true
categories: 
---

[Jekyll](http://jekyllrb.com) is a software which is managed by `rubygems`,you can
use `gem install jekyll` to install it. I will not guide you how to use `jekyll`, you
can view its offical site to learn to use it.

I will show you just three shell scripts which is writen by me to help you build and
deploy your static site. I recommend you to write your blog using `markdown` syntax
which is the best syntax I like.

Before you begin your tour of static site, maybe your should create a repository
on github, maybe name it as `${name}.github.io`(do not ask me why, you should know it).
Then you checkout to a `source` branch, which place all your site source codes, and from
now on you will always stay in branch `source`, next I will introduce for you my three
shell scripts, and you just place them in a new directory which is called `bin`, and then
add `bin` to jekyll exclude(just add `exclude: ['bin']` to your `_config.yml`).

### clean
The first script is the most simple, and it just help you clean some tmp directories and
files.

{% highlight bash %}
#!/bin/sh
binpath=$(cd `dirname $0`; pwd)
suc_mutex=0
fail_mutex=0
sitepath="${binpath}/../_site"
if [ -d $sitepath ]; then
    rm -fr $sitepath
    if [ $? -eq 0 ]; then
        suc_mutex=$[suc_mutex+1]
        echo "\033[32;40mDelete ${sitepath} Successful... \033[0m"
    else
        fail_mutex=$[fail_mutex+1]
        echo "\033[31;40mDelete ${sitepath} Fail! \033[0m"
    fi
fi
deploypath="${binpath}/../_deploy"
if [ -d $deploypath ]; then
    rm -fr $deploypath
    if [ $? -eq 0 ]; then
        suc_mutex=$[suc_mutex+1]
        echo "\033[32;40mDelete ${deploypath} Successful... \033[0m"
    else
        fail_mutex=$[fail_mutex+1]
        echo "\033[31;40mDelete ${deploypath} Fail! \033[0m"
    fi
fi

if [ $suc_mutex -eq 0 ] && [ $fail_mutex -eq 0 ]; then
    echo "\033[32;40mAlready clean! \033[0m"
elif [ $suc_mutex -gt 0 ] && [ $fail_mutex -eq 0 ]; then
    echo "\033[32;40mClean successful! \033[0m"
fi
{% endhighlight %}

###post

post script is to help you add a post of your site.

{% highlight bash %}
#!/bin/sh
binpath=$(cd `dirname $0`; pwd)
postpath="${binpath}/../_posts"
date=$(date +"%Y-%m-%d")
time=$(date +"%T")
postname="$date-$1.markdown"
filename=$postpath/$postname
touch $filename
echo "---
layout: post
title: \"$1\"
date: $date $time
comments: true
categories: 
---" > $filename
vim $filename
if [ $? -eq 0 ];then
    echo "\033[32;40mAdd post ${filename} Successful! \033[0m"
else
    echo "\033[31;40mTo add ${filename} Fail! \033[0m"
fi
{% endhighlight %}

### deploy

deploy script help your to build your site and push your site to your repository's master
branch, then you can view your site by `http://$[name].github.io`,and in alternative 
you can also publish your site to your own server.

{% highlight bash %}
#!/bin/sh
binpath=$(cd `dirname $0`; pwd)
deploypath="${binpath}/../_deploy/"
sitepath="${binpath}/../_site/"
cd ${binpath}/../
echo "\033[32;40mBuilding site... \033[0m"
jekyll build
echo "\033[32;40mDeploying site... \033[0m"
rm -fr _deploy
git clone -b master . _deploy/ --single-branch
cd _deploy
rm -fr ./*
cd ..
cp -r _site/* _deploy/
cd _deploy
git add .
git add -u
now=$(date +"%Y-%m-%d %H:%M:%S")
echo "\033[32;40mUpdating site... \033[0m"
git commit -m "Site update at $now"
echo "\033[32;40mPush site to github... \033[0m"
git push origin master --force
cd ..
git push origin master --force
echo "\033[32;40mPublish site to your server... \033[0m"
#rsync -avz --delete _site/ name@ip:/home/www/blog
echo "\033[32;40mComplete! \033[0m"
{% endhighlight %}

Have fun with them, and any requirements tell me below and I will satisfy you quickly.

### Attention

**I will update my manner to use `jekyll` in the near future
include the shell scripts above, so to see my [BlogRepo](https://github.com/baijian/baijian.github.io/tree/source) as the newest version**

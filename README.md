Blog
===

This repo is my blog repo using `jekyll` and some custom scripts to help
me post and build my blog. You can just clone the `source` branch of this repo to
your own repo and remove the exist posts in `_posts` directory
and imamges in `images` directory. And remember to modify `_config.yml`.
You can modify style by files in `_layout` and `css` to meet your own demands.
Any questions can post a issue to me~

### Guide

* Install rbenv to manage your ruby version

$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv

$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc

$ echo 'eval "$(rbenv init -)"' >> ~/.zshrc

* Install one version of ruby

$ rbenv install 1.9.3-p545

$ rbenv shell 1.9.3-p545

$ gem install bundle 

$ rbenv rehash

* Install dependences

$ script/bootstrap

After execute this script, `bundle install` will install dependences to vendor
directory,then you can execute `bundle exec jekyll build` to generate site.

* Use script to post, build and deploy blog site

$ script/post [hello-my-first-post]

$ script/deploy

Deploy script will push your blog site to [name].github.io, or you can modify
`script/deploy` to deploy your blog site to rsync to your own server.

$ script/clean

### TODO List


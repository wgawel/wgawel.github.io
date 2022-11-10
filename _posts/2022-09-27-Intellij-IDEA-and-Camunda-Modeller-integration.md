---
layout: post
title: "Intellij IDEA and Camunda Modeller integration"
categories: [IntelliJ, Camunda, tools]
---

I have a Github Pages site with the Jekyll engine. How to preview a post before push a commit to the Github repository? Follow the instruction below.

### Install Windows Subsystem for Linux (WSL).

1. Open `Command Prompt` as administrator.

2. Type following command and hit Enter:

        wsl --install

3. Restart Windows.


### Local Jekyll installation

In WSL terminal:

    sudo apt install ruby-dev
    sudo apt install ruby-bundler

Go to the directory with Jekyll:

    ls /mnt/c/projects/my_jekyll_site
    bundle install

Build your local Jekyll site:

    bundle exec jekyll serve

Or, with livereload:

    bundle exec jekyll serve  --livereload

Done! Your site will be available on http://127.0.0.1:4000/

[![Jekyll Windows Preview](/assets/image/screenshots/jekyll_windows_preview.png)](/assets/image/screenshots/jekyll_windows_preview.png)
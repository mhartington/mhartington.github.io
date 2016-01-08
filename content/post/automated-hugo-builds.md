+++
date = "2016-01-04T22:48:59-05:00"
draft = true
title = "Automated Hugo Builds"
+++

This is a static site built using [Hugo](https://gohugo.io). It's similar to Jekyll in a few ways, but written in Go, so it's super fast. Plus it's not ruby so thats always a plus. Hugo is pretty simple to setup, especially on OSX. You can just `brew install` it.

```bash
brew install hugo

# create a new site
hugo new site mySite
```

You can also run this on Linux or Windows, so another plus over Jekyll.


Posts are written in Markdown or HTML, what ever you prefer with some frontmatter that feels very familiar to Jekyll, so you should feel right at home.

A big selling point was that automated builds were dead simple to setup. They even have a section in their docs about setting it up for Github pages. Though, for their example, it assumes you're using this for a projects gh-pages branch, and not a `user.github.io` site. While the process is similar, it does have some important differences that you need to watch out for. Nothing against Hugo, but some thing that they could have made a bit clearer.


## Enabling builds

So first I'm going to assume you've dug into Hugo a bit and maybe built a simple site with a page or two. So let's push this to github!


 [![blog-repo](/img/blog-repo.png)](/img/blog-repo.png)

 One thing you may notice is that I've pushed this to a dev branch, and then made that the default branch for the repo. This is where the Hugo docs differ. Since this is a `user.github.io` site, we'll need to deploy the final output to master.

## Building on Push

Now that we have our code somewhere, we can use a automated build service to handle pushing the updates for us. For this we'll use [Wercker](http://wercker.com). Now this is similar to Travis or CircleCI, but it has a bunch of pre-built processors that you can use to make setup even easier.


Sign up for Wercker (don't worry, it's free) and create a new app from a github repo

[![new-app](https://gohugo.io/img/tutorials/wercker-add-app.png)](https://gohugo.io/img/tutorials/wercker-add-app.png)

_image courtesy of Hugo's offical docs_



Follow the steps and configure you access without SSH key

 [![blog-repo](https://gohugo.io/img/tutorials/wercker-access.png)](https://gohugo.io/img/tutorials/wercker-access.png)

_image courtesy of Hugo's offical docs_

This will give you a template `wercker.yml` file, which you can ignore for now. We'll be making our own later on.


Now that we have an app on Wercker, let's configure our build. Head over to github and grab an [access token](https://github.com/settings/tokens), the default setting will do fine. Copy the token somewhere like a blank text file.

Back on Wercker, navigate to your app and click the gear icon in the top


[![new-app](/img/app-settings.png)](/img/app-settings.png)

In the sidebar, click Targets and add a custom deploy method.



[![new-app](/img/custom-deploy.png)](/img/custom-deploy.png)

We'll just call this GithuPages for now, enable automatic builds, and set the target branch to build from to dev. Now in my case, dev is the name of the branch my site is developed on. It may be different for you so update the name to match what you used.


[![new-app](/img/gh-pages.png)](/img/gh-pages.png)


Click save and then add a new environment variable. This will be your github access token you just created.
We'll call it `GIT_TOKEN`, add the token, click save and we should be good.


[![new-app](/img/token.png)](/img/token.png)

## The `wercker.yml` file

So back in our code base, let's create a `wercker.yml` file. This is how we'll configure out build on Wercker and build the hugo site.


```yml
box: debian
build:
  steps:
    - arjen/hugo-build:
        version: "0.15"
        theme: hugo-minimalist-theme
        flags: --buildDrafts=true
deploy:
  steps:
    - install-packages:
        packages: git ssh-client
    - uetchy/gh-pages:
        token: $GIT_TOKEN
        repo: mhartington/mhartington.github.io
        path: public
        domain: mhartington.io
```

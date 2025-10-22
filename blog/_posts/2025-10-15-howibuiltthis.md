---
layout: post
ytembedid: DbFgUtIenfs
song: Tomodachi Life - Mii Editor
artist: Asuka Itō
title: How I Built This Site
tags: ['programming','computers']
date: 2025-10-15 16:56:26 +0100
---
hi :)
today i wanna talk about how i made this cool website
## part 1: jekyll and hyding the mess
if you're not familiar with working with static websites but wanna give them a bit more ooomph, i really recommend checking out [Jekyll](https://jekyllrb.com). Jekyll is a static website generator that also helps you with blogging!

![](/img/posts/2025-10-15-howibuiltthis/jekyll1.jpg)

at first, i was managing my website like i would any other simple html webpage. the issue was that, as it grew more complex, it became what we the portuguese call a "gatafunhada". worse than a simple mess, it becomes unsustainable. i had elements that were common between all pages and had to be equalized all the time. if i wanted to do a mini blog in here something had to change.

![](/img/posts/2025-10-15-howibuiltthis/jekyll2.jpg)

with Jekyll, this all changes. instead, i can just write my own layouts for simple pages or blog posts and all i have to worry is about the main content. if there's something on those common elements on both of the bars you see on the left and the right i need to update, i can just update the layout and it will update for all other pages.

## part 2: IN A LINK TO THE PAST, YOU EQUIP AN ITEM, PRESS A BUTTON AND YOU'RE THERE
this was all good with jekyll but this added another layer of overhead whenever i wanted to upload changes from my website to neocities.

some of this could be mitigated using the Neocities CLI utility but that's still a pretty manual setup! you have to push the website over to neocities EVERY TIME you make a little change. that doesn't quite scale...

![](/img/posts/2025-10-15-howibuiltthis/neocitiescli.png)

nevertheless, neocities cli would prove to be a crucial piece. i decided i'd move development of this website over to github and have deployment be done through github actions. let's see how that's setup:

```yml
name: CI/CD
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build
        uses: actions/jekyll-build-pages@v1
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: site
          path: _site
          if-no-files-found: error
  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: head
      - name: Install Neocities CLI
        run: gem install neocities -v 0.0.20
      - name: Get artifact
        uses: actions/download-artifact@v4
        with:
          name: site
          path: _site
      - name: Upload artifact to Neocities
        run: |
          mkdir -p $HOME/.config/neocities
          echo "{\"API_KEY\":\"${{ secrets.NEOCITIES_API_KEY }}\",\"SITENAME\":\"shodantltwb\"}" > $HOME/.config/neocities/config.json
          neocities push _site
```

As you can see, this pipeline has two main steps:
1. the build stage where we use the "jekyll-build-pages" action to build our website for us, then upload the generated website as an artifact
2. the deployment stage, where we install the neocities cli, configure it and upload the contents of the artifact to our website.

notice the configuration step specifically:
```yml
...
        run: |
          mkdir -p $HOME/.config/neocities
          echo "{\"API_KEY\":\"${{ secrets.NEOCITIES_API_KEY }}\",\"SITENAME\":\"shodantltwb\"}" > $HOME/.config/neocities/config.json
...
```

You'll need to plug your Neocities API key into the cli preemptively, which you can get by going to https://neocities.org/settings/{your_site_name}#apikey. Don't reveal this to anyone! if you did so accidentally, generate a new one immediately!

To make sure you also don't reveal this accidentally in the pipeline, we'll make use of repository secrets on GitHub. On your repository's "Settings", you'll want to go "Secrets and Variables" > "Actions" > "Repository Secrets", then create one for your Neocities API Key, like so:

![](/img/posts/2025-10-15-howibuiltthis/ghsecrets.png)

This way, once the pipeline runs, your secret will be used to push the website to neocities and will never be exposed!

I'm thinking maybe at a later date changing to a private instance of GitLab as GitHub is known to crawl even private repositories for code to train Copilot but in the meantime this is a good solution.

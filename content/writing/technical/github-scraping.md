---
title: Automated Data Scraping with Github Actions
slug: github-scraping
subtitle: Data Scraping without a Database
categories: ['Tech']
date: 2020-01-21
description: A neat trick I discovered from Mikeal Rogers
---

> Dec 2020 Edit: You can see a live example of this in [my own GitHub profile readme](https://github.com/sw-yx/sw-yx)

A common need I have in open source community work, especially with static site generators and the JAMstack, is scraping and updating data. For example, in the [Svelte Community](https://svelte-community.netlify.com/code) site we scrape the GitHub star count and last update, and ditto [Gatsby Starters](https://www.gatsbyjs.org/starters/). Of course, you could grab data clientside, and whatever you can't do clientside, you can throw up a serverless function to do this. 

But sometimes it just makes sense to scrape data *once* instead of every time your users access your site, especially if that data requires tokens your users may not have. Typically you'd set up a cronjob and send the data into a database somewhere. With [GitHub Actions](https://github.com/features/actions), you can do this all inside GitHub, AND save a version controlled history of all data.

I noticed [Mikeal Rogers doing exactly this for his Daily OSS watcher](https://github.com/mikeal/daily) project, and so finally took some time to check out his code and make a minimal repro so others can take it as a base.

## Demo

You can see my **demo in action** here: https://github.com/sw-yx/gh-action-data-scraping.

- The action workflow is https://github.com/sw-yx/gh-action-data-scraping/blob/master/.github/workflows/scrape.yml
- The node script that is executed is https://github.com/sw-yx/gh-action-data-scraping/blob/master/action.js
- The scraped data is https://github.com/sw-yx/gh-action-data-scraping/tree/master/data

For those new to npm, there is a simple [npm script](https://www.freecodecamp.org/news/introduction-to-npm-scripts-1dbb2ae01633/) defined in `package.json`. This is so you can manually run it while writing and testing your code. The action workflow calls this same exact action to reduce any discrepancies.

## The Script

Straight to the point:

```yaml
on:
  schedule:
    - cron:  '0 8 * * *' # 8am daily. Ref https://crontab.guru/examples.html
name: Scrape Data
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master # check out the repo this action is in, so that you have all prior data
    - name: Build
      run: npm install # any dependencies you may need
    - name: Scrape
      run: npm run action # actually run your npm script for scraping
      # env:
      #   WHATEVER_TOKEN: ${{ secrets.YOU_WANT }}
    - uses: mikeal/publish-to-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub sets this for you
```

The basic idea, in English, is:

- You set a cron triggered github action ([cron examples](https://crontab.guru/examples.html) - max frequency every 5 mins)
- Your action checks out your repo with https://github.com/actions/checkout so that it has existing/prior data
- `npm install` and run your scrape script, which writes files to somewhere in your repo. 
- check it back in with https://github.com/mikeal/publish-to-github-action

That's it! Look ma, no database!

As part of your workflow, you can also fire off a static site build after this action completes, or weekly, or whenever else you like.

## Limits

You can do whatever you like with this, including taking screenshots of sites!

The limits I can think of are the limits of GitHub and GitHub Actions:

- The max frequency of cronjobs on GitHub actions is every 5 minutes. For more frequent scraping, you will have to turn elsewhere.
- GitHub has a [soft storage limit of 1GB](https://www.quora.com/What-is-the-max-storage-limit-per-repository-in-GitHub)
  - You can [work around this with Git LFS](https://twitter.com/mikeal/status/1219739811159801856) if you have to!
- Actions are free for public repos, but incur costs for private repos
  - [You get 6 Concurrent jobs, 1000 API requests an hour, and each job can take up to 6(!) hours](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/about-github-actions#usage-limits)

In addition to these limits, GitHub Actions should not be used for:

- Content or activity that is illegal or otherwise prohibited by their Terms of Service or Community Guidelines.
- Cryptomining
- Serverless computing
- Activity that compromises GitHub users or GitHub services.
- Any other activity unrelated to the production, testing, deployment, or publication of the software project associated with the repository where GitHub Actions are used. In other words, be cool, don’t use GitHub Actions in ways you know you shouldn’t. 

Be a good citizen, **don't abuse it and F this up for the rest of us**!

## More

I'm looking for more great usecases for GH actions:

- https://www.edwardthomson.com/blog/github_actions_advent_calendar.html
- https://github.com/sdras/awesome-actions
- more?
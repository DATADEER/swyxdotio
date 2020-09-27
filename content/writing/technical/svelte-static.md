---
title: 'Static Svelte: JavaScript Blogging with 93% less JavaScript'
subtitle: Why Svelte is a Perfect Fit For Blogging
slug: svelte-static
categories: ['Tech', 'Svelte']
date: 2019-09-13
---

This blog now uses [Svelte & Sapper](https://sapper.svelte.dev/) as a static site generator, where it [previously used React & Gatsby](https://5d7699e172ae430007210374--scout-videos-51664.netlify.com/writing/moving-to-novela). This is achieved through [Sapper's `sapper export` feature](https://sapper.svelte.dev/docs#sapper_export).

Through the magic of [Netlify's Immutable Deploys](https://www.netlify.com/blog/2018/10/05/netlify-and-the-functional-immutable-reactive-deploy/?utm_source=blog&utm_medium=swyxdotio&utm_campaign=devex), we can directly compare them on representative live URL's for a recent blogpost of mine:

- [With React/Gatsby](https://5d7699e172ae430007210374--scout-videos-51664.netlify.com/writing/netlify-redirects-i18n): **138kb**
- [With Svelte/Sapper](https://5d7c46e60da4524431f76aef--scout-videos-51664.netlify.com/writing/netlify-redirects-i18n/): **9kb**
- For the curious, I also recently wrote about using [Next.js as a Static Site Generator](https://scotch.io/@sw-yx/using-nextjs-as-a-static-site-generator-for-netlify) as well, but don't expect much difference.

> ⚠️To be very clear - this post is not even close to being an apples to apples benchmark, and I spend some effort below explaining why it isn't. If you aren't prepared to read this post with a critical mind and understanding that this is a random work in progress report, please stop here.

## Screenshots

### Gatsby version (138kb)

![screenshot of Gatsby site with 138kb of JS](/assets/sveltegatsby.jpeg)

### Sapper version (9kb)

![screenshot of Sapper site with 9kb of JS](/assets/sveltesvelte.jpeg)

## Differences and Similarities

### Similarities

First, the similarities must be noted, for any reasonable discussion to be had. For example, the _reductio ad finem_ conclusion of the premise of this post is to use any old static site generator like [Jekyll or Hugo](https://www.staticgen.com/) since it will result in _no_ JS payload.

**Costs and Benefits of JS Blogging**

However, both Gatsby and Sapper offer clientside rehydration, which makes the subsequent page navigation very fast. It is also easy to:

- progressively add dynamic features without a major refactor
- code in reusable components
- and leverage the vast JS ecosystem to add functionality.

Yes, these are all available through other means. I'm aware.

This is by no means a free tradeoff: as [Addy Osmani recently noted](https://addyosmani.com/blog/rehydration/), an "uncanny valley" of rehydration exists, and [Alex Russell consistently warns](https://infrequently.org/2018/09/the-developer-experience-bait-and-switch/) that better developer experience is not without its costs. [Nicole Sullivan recently noted](https://www.youtube.com/watch?v=dvtfNpt75aA&t=2s) we don't even have the language and terminology yet to intelligently discuss tradeoffs between first and subsequent loads. These matter in different contexts.

**Syntax Highlighting**

As for Sapper JS size, I originally used [PrismJS](https://prismjs.com/), which adds ~19kb of JS, beacuse as a developer you can pry syntax highlighting from my cold dead hands. So it wouldn't be fair to compare a site without syntax highlighting to my old site with one.

However, there are no-JS solutions for syntax highlighting, since I don't intend readers to edit the code in runtime. [Andrew Branch recently twote](https://mobile.twitter.com/atcb/status/1158480783666888704) about how to preprocess ALL your syntax-highlighting using some VSCode API's. I put this off as too complicated, but [Khrome](https://khrome.dev) tipped me off that the open source [shiki](https://github.com/octref/shiki) library from the Vue ecosystem does the same thing and so I managed to get that in.

**High End Lighthouse Score**

The default, no throttling Lighthouse scores are totally fine. I'm not going to be read by the next billion users when I don't even have 200 subscribers on [my mailing list](https://tinyletter.com/swyx). These are all going to vary based on the specific implementation details of the site (in particular, how much time/effort I spend on it) so don't pay this _too_ much mind, but not that they're all... _alright_. Definitely not a failing grade:

- [Gatsby version](https://5d7699e172ae430007210374--scout-videos-51664.netlify.com/writing/netlify-redirects-i18n):
  - Performance: 100
  - Accessibility: 96
  - Best Practices: 93
  - SEO: 89
  - PWA: yes
- [Sapper version](https://5d7c46e60da4524431f76aef--scout-videos-51664.netlify.com/writing/netlify-redirects-i18n/):
  - Performance: 100
  - Accessibility: 100
  - Best Practices: 93
  - SEO: 89
  - PWA: some things missing (i didn't spend time on a manifest)

## Differences

There are material differences in implementation that make the two sites not comparable.

**Low End Lighthouse Score**

The (mobile, 4x applied throttling) Lighthouse scores tell a somewhat different story. This is the same as we had in the screenshots. So I won't repeat.

**Developer Experience**

Time spent getting things to work is an important cost to factor in. Sapper is by far less mature, and not designed to have an upfront content ingestion pipeline with pluggable lifecycles. I spent a significant amount of time coding up the markdown ingestion for my site, which in Gatsby is as trivial as adding a couple of source and transform plugins. It's another involved process to pipe this data in to templates, made harder by the fact that I had to make [my own site generator](https://npm.im/ssg) in the process, but Gatsby is also quite involved where one has to create the right templates and queries and `gatsby-node` files. Kind of a wash here until Gatsby Themes mature, and even then the lack of a standard format may hurt mass adoption or intra ecosystem compatibility (imagine 2-3 warring factions of Gatsby Themes! Good or bad? Unclear)

There are other missing pieces I have to code up myself as well, for example a PWA or [RSS feed plugin](https://twitter.com/Rich_Harris/status/1172869555611459584).

**Image Optimization**

As [Una Kravets has noted](https://mobile.twitter.com/Una/status/687690138550288384), images are a critically important part of web performance. [Gatsby-Image](https://www.gatsbyjs.org/packages/gatsby-image/) not only preprocesses images to resize them down via standard techniques, but also helps you rapidly [load superfast images for a nice progressive upgrade effect](https://using-gatsby-image.gatsbyjs.org/). My colleague [Phil Hawksworth recently wrote on CSS Tricks about how to achieve your own lazy loading](https://css-tricks.com/tips-for-rolling-your-own-lazy-loading/) - so you definitely don't need Gatsby for this - but it is nice to have a blessed, maintained, tested, well documented, adaptable approach.

This Sapper site does not (yet!) do any of that. There are open source [Svelte Image](https://svelte-image.matyunya.now.sh/) components I have yet to investigate.

**Styling**

Finally, the Sapper site does not use CSS-in-JS. This is a very controversial topic with pros and cons so I put it last. In particular, if you have not spent time appreciating [some basic facts of CSS in JS from proponents](https://mxstbr.com/thoughts/css-in-js/), as well as [some nuances in implementation](https://github.com/styled-components/styled-components/issues/2377), then your debate will not be very well informed.

The old site has a [Dark Theme Toggle](https://github.com/sw-yx/gatsby-theme-dev-blog/blob/master/packages/gatsby-theme-dev-blog/src/components/Header/ThemeToggler.js). Also a responsive mobile navigation. This Sapper implementation does not.

However, I expect [animations to be a lot easier with Svelte](https://svelte.dev/tutorial/animate) as it is a first class citizen. This frees me up to play a lot more with animations in future.

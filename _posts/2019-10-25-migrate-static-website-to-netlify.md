---
layout: post
title:  "Migrate a Static Website to Netlify"
category: IT
tags: publishing
bigimg: /img/bigimg_ttl.jpg
credname: Chris Ried
credurl: https://unsplash.com/photos/bN5XdU-bap4
comments: true
excerpt: Recently I got to know Netlify which is a provider for building, deploying, and hosting static websites. I tried it out and decided to migrate my blog to Netlify. The necessary steps are described here.
---

This blog was offered as a static website hosted with a classic - or call it old fashioned - hosting provider at the time I began writing this blog post.
Recently I got to know [Netlify][netlify-home], which looks like it could simplify the whole publishing workflow related to static websites.
I played around for some time and decided to migrate my blog.
The necessary steps are documented here.

For me there is currently no reason to question the decision of using a static website nor the static site generator used for this.
Therefore please note that the blog as post if focused on the reasons why I chose Netlify as well as the build, deployment and hosting of a static website with Netlify.

# Starting Point

First of all let's have a short look at the situation before the migration.
The tech stack for my personal blog consisted of [Jekyll][jekyll-home] as static site generator, [GitHub][github-home] for hosting the Git repository with all source files, [Travis CI][travisci-home] for builds and deployments, and a small website hosting package offered by [Ionos][ionos-home].

This is a setup I liked, because it's pretty similar to what we do in Software Engineering.
The process was automated, i.e. whenever there was a push to the master branch, the build process was triggered and took care of generating the static website representing my blog.
Once finished, the result was copied to the webspace and the new version of the blog was published.

# Considerations

There was no urgent reason to change something in the build and deployment process nor in the hosting solution.
Nevertheless reading about Netlify made me curious what it is and what it would offer.

What I found out is that it could take over the parts which were previously solved by Travis CI and Ionos in my setup.
In other words it's a solution for running static website builds which can be triggered directly from the version control system resp. the code hosting platform - which is GitHub in my case.
After such a build finishes, the result is automatically deployed to Netlify, i.e. it also takes over the responsibility to host the static website.
One of the first benefits for me is the automatic use of the Netlify CDN which could speed up page load times.
The cache is invalidated automatically whenever a new version of the static website is published.

But the most promising features for me are other ones.
The first one is the availability of a publish history with the possibility to rollback to a previous version with just one click.

The second one is the feature of preview publishing.
Whenever you create a pull request, Netlify hooks in, performs some checks, runs a build and publishes to a separate environment which is available under an automatically generated URL.
Related information is provided directly in the pull request if you use GitHub.

But to be honest - besides those cool features, the price was a relevant criteria for the decision to migrate to Netlify.
I'm willing to invest time and effort in the migration itself, but I don't want to increase ongoing costs for my blog.
And this is the good news.
For small static websites like my blog with low traffic and no other contributors, the offering of Netlify is for free in the starter plan.
If you want to have a look at the pricing tiers, please check the documentation on the [Netlify website][netlify-pricing].



[netlify-home]: https://www.netlify.com/
[netlify-pricing]: https://www.netlify.com/pricing/
[ionos-home]: https://www.ionos.de/
[jekyll-home]: https://jekyllrb.com/
[github-home]: https://github.com/
[travisci-home]: https://travis-ci.org/
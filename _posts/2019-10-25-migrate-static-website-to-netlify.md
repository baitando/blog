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

# Migration Plan

With the decision made, it's now time to plan the migration.
The necessary steps are:

1. Create a Netlify account.
2. Take care of SEO pitfalls.
2. Create the site on Netlify and let it execute the initial publish.
3. Configure the existing domain for Netlify.
4. Do some tests to verify that everything works as desired.
5. Remove previously used infrastructure which is not necessary any more.

# Create Netlify Account

This step is not really worth describing it.
Just go to the [signup page][netlify-signup] of Netlify.
There are several possibilities how you can create an account.
I registered with my GitHub account.

![Sign Up For a Netlify Account](/img/netlify_signup.png){: .center-image }

You will then receive an email with a confirmation link.
Simply click this link and your account will be ready to use.

# Take Care of SEO Pitfalls

When I had a more detailed look at the features offered by Netlify some SEO considerations came to my mind - especially the topic of duplicate content.
If you never heard of this it's definitely worth to read about this.
Basically search engines recognize, if the same content is published on different URLs.
If such duplicate content is recognized, it gets penalized which decreases the rating.

This is a problem related to features offered by Netlify.
Let's have a closer look at them and find a solution.

1. Netlify makes a site available using a subdomain of Netlify as well as your custom domain if you configure one.
The consequence is that my blog is available on [https://baitando.netlify.com](https://baitando.netlify.com) as well as on [https://baitando.de](https://baitando.de).
Since the content is the same, we will run into the duplicate content issue.
This is of course also an issue for the migration, since the old hosting remains active when the setup on Netlify is done.
2. Netlify can create previews for pull requests.
For me this is a really cool feature. If you turn it on, a preview is automatically built and deployed on a URL defined by Netlify.
The consequence is, that we now have several additional copies of the site.
Since most of the content is the same (except the changes done in those branches), we will run into the duplicate content issue again.

The second topic was asked at least twice in the Netlify community forum as you can see in [this][seo-post-1] post from May 2019 and [this][seo-post-2] post from August 2019.
One of the Netlify support team members pointed out in both posts, that Netlify automatically takes care of preventing deploy previews from being indexed by sending the `X-Robots-Tag: noindex` header in those cases.
This seems not to be documented yet (at least I did not find anything else than the two forum posts in my research).

I tried it out with a deploy preview and can confirm that the header is set.
As described in the [Google documentation][seo-header], this should have the desired effect of not crawling those pages.

Nevertheless I recommend to set the canonical tag in the `head` section of each page.
This is in general a good idea and in addition to that I want to make sure that my SEO does not depend on an undocumented feature of Netlify.
This canonical tag tells the crawler where the original content is located.
Setting the tag gives higher priority to the linked page.
For further details on this topic please have a look e.g. at the [documentation provided by Google][seo-canonical].

```html
<link rel="canonical" href="https://www.baitando.de/2019-10-25-migrate-static-website-to-netlify" />
```

This link points to the published instance of my blog and is the same on all deployed instances.
A crawler would then always be pointed to the published page instead of the one belonging to the deploy preview or the one using the Netlify URL.

In my case this was tag was already part of the generation process, i.e. there was no measure necessary.
If not it would not be a big deal to implement this from scratch.
So let's have a short look at how this looks like in my blog.
Please note that this description is valid at the time I write this post.
The files and the layout of the blog source may change in the future.

In Jekyll you can define variables in a file called `_config.yml`.
There we define the site URL as variable `url`.
In the page generation template, this variable is picked up and extended with the concrete site path.
The page template is `_layouts/base.html` which includes the `head` definitions from `_includes/head.html`.
There we have a definition like the one shown below which generates the canonical tag.

```html
<link rel="canonical" href="{{ site.url }}{{ page.url }}" />
```

# Create Site

Once your account is ready and you log in, you will recognize that a team was automatically created.
Such a team seems to be a container for zero, one or more sites.
A team consists of members who are able to manage the sites assigned to the team.
For my use case this is not relevant since I'm the only one contributing to the blog.

![Team Dashboard on Netlify](/img/netlify_create_site.png){: .center-image }

As you see the list of sites is empty.
So let's create the site now by clicking on the button and following the wizard.

![Team Dashboard on Netlify](/img/netlify_create_site-1.png){: .center-image }

After clicking the button you have to select the code hosting plaform you use.
According to the options offered, Netlify supports GitHub, GitLab and Bitbucket.
I use GitHub.

![Team Dashboard on Netlify](/img/netlify_create_site-2.png){: .center-image }

Once you click on one of the options, you have to grant access to your account on the selected platform.
For details about the permissions you can check the [Netlify documentation][netlify-vcspermissions].
Then select the repository containing your source files.

![Team Dashboard on Netlify](/img/netlify_create_site-3.png){: .center-image }

As last step for now we have to do some configuration.
The first settings stay on default in my case.
As owner of this site the automatically created team is pre-selected.
Since you have only one team in the starter plan, there is nothing to change for me.
I want to publish my blog from the master branch.
So this setting remains on the default too.

![Team Dashboard on Netlify](/img/netlify_create_site-4.png){: .center-image }

The other settings depend on the static site generator you use.
In my case this is Jekyll.
The screenshot show the settings required for this one.
If you are not sure what to set here, you can check common configurations listed in the [Netlify documentation][netlify-buildcmds].

![Team Dashboard on Netlify](/img/netlify_create_site-5.png){: .center-image }

Now we can finish the configuration with a click on the deploy button.
This will trigger the first build.
If this is successful, your static website will automatically be published on Netlify.

![Team Dashboard on Netlify](/img/netlify_create_site-8.png){: .center-image }

The result will be available on a Netlify subdomain, which is provided on the page once the process finished.
You can now check the result by following the link provided there.
Since we already took care of SEO pitfalls, there is no problem in having another URL for your blog.
The canonical URL is set and crawlers will therefore know where to find the original post.

As an optional step you can change the name of your site from the automatically generated one by a meaningful one.
In my case I chose the name `baitando`.
Please keep in mind that changing the site name will also change the Netlify subdomain of your site, i.e. in my case the initial subdomain `condescending-pike-cc8578.netlify.com` was automatically changed to `baitando.netlify.com`.

# Domain Configuration

Now comes the change which will effectively switch from the previous hosting provider to Netlify.
To achieve this, we have to adjust the DNS first and to configure the domain on Netlify next.

There are several ways to handle this, which are also documented in the [Netlify documentation][netlify-dns].

1. If the domain is not registered yet, you can do that with Netlify.
If you do so, Netlify will automatically do the DNS configuration for you. 
Since we are migrating an existing website this is obviously no option here.
2. If you already registered the domain elsewhere, you can switch to the DNS of Netlify.
The result is similar to the previous option and makes it possible to e.g. use branch specific subdomains which are added by Netlify on the fly.
3. If you already registered the domain elsewhere and you don't want to use the DNS of Netlify, you can point a separate DNS to Netlify for your domain.

I want to keep the impact of the migration as small as possible.
If I would choose the second approach, I would have to migrate all existing blog unrelated DNS settings related to this domain too.
Therefore I decided to follow the third approach.

The necessary steps are described in the [documentation][netlify-externaldns].
If you use already used a subdomain which is not `www`, the DNS adjustment is pretty simple.
You just have to set the `CNAME` of the subdomain to your Netlify subdomain and everything should work fine from a DNS point of view.

The subdomain I use is `www` which leads to the necessity of additional settings.
If you register a `www` subdomain in Netlify, the Apex domain is automatically added also.
In my case I configured `www.baitando.de` and `baitando.de` (the Apex domain) was automatically added.
In addition to the `CNAME` for the `www` subdomain I had to configure the DNS for the Apex domain. 

Some providers offer the possibility to register `ANAME` or `ALIAS` records which are similar to the `CNAME` but on the Apex level.
Unfortunately Ionos does not do this.
Therefore I had to point the `A` record directly to the IP of the Netlify load balancer as suggested in the screenshot below.

![Team Dashboard on Netlify](/img/netlify_dns-config.png){: .center-image }

The downside of pointing to a static IP like we have to do with the `A` record is, that this contradicts the idea of the CDN which leverages different IPs depending on the region of the request.
Luckily this is no problem in a setup like mine.
The primary URL of my blog is [https://www.baitando.de](https://www.baitando.de).
This URL is always the target.
All other requests like [https://baitando.de](https://baitando.de) are automatically redirected.
The sitemap also uses the primary URL.

The consequence is, that there would be only a single request which gets redirected to the subdomain, if someone would use the Apex domain in the request.
All following requests are then directed to the subdomain and make use of the CDN, since we set a `CNAME` there.
The `A` record is therefore not really a problem from a performance point of view.

Once the DNS changes are done it could be necessary to wait for some time, because the DNS propagation may take some time.
During this time you will see a message similar to the one in the screenshot below.

![Team Dashboard on Netlify](/img/netlify_dns-dnsincomplete.png){: .center-image }

The next step is to get a certificate for the custom domain which is used to ensure transport layer security.
You can configure this in the certificate section by uploading an existing certificate.
I recommend to use a Let's Encrypt certificate, which is a great and easy solution to obtain widely trusted certificates.
The mechanism to get one is built in Netlify, i.e. after the DNS propagation finished you can trigger the creation process.
If the DNS propagation is not already finished, you will see a message similar to the one in the screenshot below.

![Team Dashboard on Netlify](/img/netlify_dns-certincomplete.png){: .center-image }

After the certificate was issued successfully, Netlify configures it for the use with your domain.
This will then look similar to the screenshot below.

![Team Dashboard on Netlify](/img/netlify_dns-certcomplete.png){: .center-image } 

# Further Performance Tuning

I recognized that Netlify can do additional optimizations related to minification and lossless image compression.
If you want to try out those optimizations you can enable them in the site settings section.
There you can select which optimizations to apply during deployment.

![Team Dashboard on Netlify](/img/netlify_asset_optimization.png){: .center-image }

Please keep in mind that changes in this configuration get effective with the next deployment.
To check the results immediately you need to trigger a new deployment manually.

[netlify-home]: https://www.netlify.com/
[netlify-signup]: https://app.netlify.com/signup
[netlify-pricing]: https://www.netlify.com/pricing/
[netlify-vcspermissions]: https://docs.netlify.com/configure-builds/repo-permissions-linking/
[netlify-buildcmds]: https://docs.netlify.com/configure-builds/common-configurations
[netlify-dns]: https://docs.netlify.com/domains-https/custom-domains/#assign-a-domain-to-a-site
[netlify-externaldns]: https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/
[ionos-home]: https://www.ionos.de/
[jekyll-home]: https://jekyllrb.com/
[github-home]: https://github.com/
[travisci-home]: https://travis-ci.org/
[seo-steps]: https://www.codesections.com/blog/netlify/
[seo-post-1]: https://community.netlify.com/t/avoid-seo-duplicated-content-between-master-url-and-actual-production-url/850
[seo-post-2]: https://community.netlify.com/t/preferred-method-to-block-seo-for-branch-subdomains/2859
[seo-header]: https://developers.google.com/search/reference/robots_meta_tag
[seo-canonical]: https://support.google.com/webmasters/answer/139066

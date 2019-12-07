---
layout: post
title:  "Migrate a Static Website to Netlify"
category: IT
tags: publishing
bigimg: /img/bigimg_fast.jpg
credname: Marc-Olivier Jodoin
credurl: https://unsplash.com/photos/NqOInJ-ttqM
comments: true
excerpt: Recently I got to know Netlify which is a provider for building, deploying, and hosting static websites. I tried it out and decided to migrate my blog to Netlify. The necessary steps are described here.
---

This blog was offered as a static website hosted with a classic (or call it old fashioned) hosting provider at the time I began writing this blog post.

Recently I got to know [Netlify][netlify-home], which looks like it could simplify the whole publishing workflow related to static websites.
I played around for some time and decided to migrate my blog.
The necessary steps are documented here.

Please note that I'm simply a user migrating to Netlify and not related to Netlify in any other way.
I do not benefit in any way from writing this blog post.
I don't want to convince you to use Netlify.
But if you come to the conclusion that you want to migrate, feel free to consider my experiences documented here.

# Starting Point

First of all let's have a short look at the situation before the migration.
The tech stack for my personal blog consisted of [Jekyll][jekyll-home] as static site generator, [GitHub][github-home] for hosting the Git repository with all source files, [Travis CI][travisci-home] for builds and deployments, and a small website hosting package offered by [Ionos][ionos-home].

I liked this setup because the build and deployment process and the necessary tooling are similar to what we do in Software Engineering.

Whenever there was a push to the master branch, the build process was automatically triggered and took care of generating the static website representing my blog.
Once finished, the result was copied to the webspace and the new version of the blog was available.

# Considerations and Motivation

There was no urgent reason to change something in the build and deployment process nor in the hosting solution.
Nevertheless, reading about Netlify made me curious what it is and what it would offer.

What I found out is that it could take over the parts which were previously solved by Travis CI and Ionos in my setup.
In other words it's a solution for running static website builds which can be triggered directly from the version control system resp. the code hosting platform - which is GitHub in my case.

After such a build finishes, the result is automatically deployed within Netlify, i.e. it also takes over the hosting of the static website.

The relevant features for me discovered so far are:

* Netlify integrates a Content Delivery Network (CDN) and completely takes over management, invalidation etc.
I did not use one yet, but this could be a performance booster.
Such a CDN is not really necessary for a small hobby blog like mine, but if it's available and easy to use, I'll take it.
* Netlify keeps track of the deploy history and offers the possibility to roll back to a previous version easily.
* Netlify offers deploy previews.
When a pull request is created, Netlify builds the source branch and deploys the result to a separate environment for preview purposes.

But to be honest - besides cool features, the price was a relevant criteria for the decision to migrate to Netlify.
I'm willing to invest time and effort in the migration itself, but I don't want to increase ongoing costs for my blog.

And this is the good news.
For small static websites like my blog with low traffic and no other contributors, the offering of Netlify is for free in the starter plan.
If you want to have a look at the pricing tiers, please check the documentation on the [Netlify website][netlify-pricing].

# Migration Plan

With the decision made, it's time to plan the migration.
The necessary steps I discovered are:

1. Create a Netlify account.
2. Take care of SEO pitfalls.
2. Create the site on Netlify and let it execute the initial publish.
3. Configure the existing domain and a certificate for transport level security.
4. Do some tests to verify that everything works as desired.
5. Remove previously used infrastructure which is not necessary any more.

I'll explain the tasks of all steps one after another.

# Create Netlify Account

This step is not really worth describing it.
Just go to the [signup page][netlify-signup] of Netlify.
There are several possibilities how you can create an account.
I registered with my GitHub account.

![Sign Up For a Netlify Account](/img/netlify_signup.png){: .center-image }

You will then receive an email with a confirmation link.
Simply click this link and your account will be ready to use.

# Take Care of SEO Pitfalls

When I had a more detailed look at the features offered by Netlify, some Search Engine Optimization (SEO) considerations came to my mind - especially the topic of duplicate content.
Search engines penalize if multiple URLs deliver the same content if no additional measures are taken.
This influences the ranking for the search in a negative way.
If you want to get familiar with the details I recommend a short Google research on this topic.

The duplicate content topic could be a problem when we think about the features offered by Netlify.
Let's have a closer look at the relevant ones and find a solution.

1. Netlify makes a site available using a Netlify subdomain as well as your custom domain, if you configure one.
The consequence is that my blog is available on [https://baitando.netlify.com](https://baitando.netlify.com) as well as on [https://baitando.de](https://baitando.de).
Since the content is the same, we will run into the duplicate content issue.
This is of course also an issue for the migration, since the old hosting remains active during the setup phase on Netlify.
2. Netlify can create deploy previews for pull requests.
This is a useful feature, which is active by default. When a pull request is opened, Netlify generates the source branch and deploys the result to a separate environment available under another Netlify subdomain.
This leads to additional copies of the site.
Since most of the content is the same (except the changes done in those branches), we will run into the duplicate content issue again.

The second topic was asked at least twice in the Netlify community forum as you can see in [this][seo-post-1] post from May 2019 and [this][seo-post-2] post from August 2019.
One of the Netlify support team members pointed out, that Netlify automatically takes care of preventing deploy previews from being indexed by sending the `X-Robots-Tag: noindex` header.
This seems not to be documented yet.
At least I did not find anything else than the two forum posts in my research.

I tried it out with a deploy preview and I can confirm that the header is sent in the response of a deploy preview.
As described in the [Google documentation][seo-header], this should have the desired effect of not crawling those pages.

Nevertheless I recommend to take another measure with setting the canonical link in the `head` section of each page.
This canonical tag tells the crawler where the original content is located.
For further details on this topic please have a look e.g. at the [documentation provided by Google][seo-canonical].

Setting this link is in general a good idea.
Besides that I want to make sure that SEO related topics do not depend on an undocumented feature of Netlify.

```html
<link rel="canonical" href="https://www.baitando.de/2019-10-25-migrate-static-website-to-netlify" />
```

The snippet above points to the published instance of my blog and is the same on all deployed instances.
A similar link needs to be part of the `head` section of all pages.
A crawler would then always be pointed to the published page instead of the one belonging to the deploy preview or the one using the Netlify URL.

In my case this tag was already present, i.e. there was nothing to do.

# Create Site

Once your account is ready and you log in, you will recognize that a team was automatically created.
Such a team seems to be a container for sites.
A team consists of members who are able to manage the sites assigned to the team.
For my use case this is not relevant since I'm the only one contributing to the blog.

![Team Dashboard on Netlify](/img/netlify_team.png){: .center-image }

The list of sites is empty.
So let's create one by clicking on the button.
Then follow the three step wizard.

![Select the Git Provider as First Step of Creating a Site on Netlify](/img/netlify_create_site-1.png){: .center-image }

After clicking the button, you have to select the code hosting platform.
Netlify supports GitHub, GitLab and Bitbucket.
I use GitHub.

![Select the Git Repository as Second Step of Creating a Site on Netlify](/img/netlify_create_site-2.png){: .center-image }

Once you click on one of the options, you have to grant access to your account on the selected platform.
For details about the requested permissions, you can check the [Netlify documentation][netlify-vcspermissions].
Then select the repository containing the source files of the website.

![Basic Site Configuration as Third Step of Creating a Site on Netlify](/img/netlify_create_site-3.png){: .center-image }

As last step for now we have to do some configuration.
The first settings stay on default in my case.
The site is by default owned by the automatically created team.
Since you have only one team in the starter plan, there is nothing to change for me.
I want to publish my blog from the master branch.
So this setting remains on the default too.

![Build and Deploy Configuration as Third Step of Creating a Site on Netlify](/img/netlify_create_site-4.png){: .center-image }

The other settings depend on the static site generator you use.
In my case this is Jekyll.
The screenshot show the settings required for this one.
If you are not sure what to set here, you can check common configurations listed in the [Netlify documentation][netlify-buildcmds].

![Initial Deployment of the Newly Created Netlify Site](/img/netlify_create_site-5.png){: .center-image }

Now we can finish the configuration with a click on the deploy button.
This will trigger the first build.
If this is successful, your static website will automatically be published on Netlify.

![Site Overview in the Netlify Site Dashboard](/img/netlify_create_site-8.png){: .center-image }

The result will be available on a Netlify subdomain, which is provided on the page once the process finished.
You can now check the result by following the link provided there.

Since we already took care of SEO pitfalls, there is no problem in having another URL for your blog.
The canonical URL is set and crawlers will therefore know where to find the original post.

As an optional step you can change the name of your site in the site settings section.
In my case I chose the name `baitando`.
Please keep in mind that changing the site name will also change the Netlify subdomain of your site.
In my case it was automatically changed from `condescending-pike-cc8578.netlify.com` to `baitando.netlify.com`.

# Domain Configuration

The next step is to switch from the previous hosting provider to Netlify.
To achieve this, we have to adjust the DNS entries first.
After that we can configure the domain in the domain settings area on Netlify.

There are several ways to handle this. 
They are also documented in the [Netlify documentation][netlify-dns].

1. If the domain is not registered yet, you can do that with Netlify.
In this case Netlify will automatically handle DNS configuration. 
Since I am migrating an existing website this is obviously no option.
2. If you already registered the domain elsewhere, you can switch to the DNS of Netlify.
The result is similar to the previous option and makes it possible to e.g. use branch specific subdomains which are added by Netlify on the fly.
3. If you already registered the domain elsewhere and you don't want to use the DNS of Netlify, you can point a separate DNS to Netlify.

I wanted to keep the impact as small as possible.
To avoid a migration of blog unrelated DNS settings, I decided to use the third approach.

The configuration of an external DNS is described in the [documentation][netlify-externaldns].
If you use already used a subdomain which is not `www`, the DNS adjustment is pretty simple.
You just have to set the `CNAME` of the subdomain to your Netlify subdomain and everything should work fine from a DNS point of view.

The subdomain I use is `www` which leads to the necessity of additional settings.
If you register a `www` subdomain in Netlify, the Apex domain is automatically added also.
In my case I configured `www.baitando.de` and `baitando.de` (the Apex domain) was automatically added.
In addition to the `CNAME` for the `www` subdomain I had to configure the DNS for the Apex domain. 

Some providers offer the possibility to register `ANAME` or `ALIAS` records which are similar to the `CNAME` but on the Apex level.
Unfortunately Ionos does not offer this.
Therefore I had to point the `A` record directly to the IP of the Netlify load balancer as suggested in the screenshot below.

![Configuration of an Apex Domain for Netlify](/img/netlify_dns-config.png){: .center-image }

The downside of pointing to a static IP with the `A` record is, that this contradicts the idea of the CDN which leverages different IPs depending on the region of the request.

Luckily this is no problem in a setup like mine.
The primary URL of my blog is [https://www.baitando.de](https://www.baitando.de).
This URL is always the target.
All other requests like [https://baitando.de](https://baitando.de) should be automatically redirected there.
This redirect will be described later in this blog post.
The sitemap also points to the primary URL.

In case of a request sent to [https://baitando.de](https://baitando.de), a redirect to [https://www.baitando.de](https://www.baitando.de) occurs.
From this point on, the CDN works.
The `A` record is therefore not really a problem from a performance point of view.

Once the DNS changes are done it could be necessary to wait for some time, because the DNS propagation may take some time.
During this time you will see a message similar to the one in the screenshot below.

![Domain Settings if DNS Propagation Did Not Finish Yet](/img/netlify_dns-dnsincomplete.png){: .center-image }

The next step is to get a certificate for the custom domain.
You can configure this in the certificate section by uploading an existing certificate or by requesting one from [Let's Encrypt][letsencrypt].

I recommend the latter, which is a great and easy solution to obtain widely trusted certificates for free.
The mechanism to get one is integrated in Netlify.
After the DNS propagation finished, you can trigger the creation process.
If this is not the case, you will see a message similar to the one in the screenshot below.

![Certificate Settings if DNS Propagation Did Not Finish Yet](/img/netlify_dns-certincomplete.png){: .center-image }

After the certificate was issued successfully, Netlify configures it for your domain.
This will look similar to the screenshot below.

![Certificate Details](/img/netlify_dns-certcomplete.png){: .center-image }

You may have noticed, that a hint similar to the one below is shown in the configuration section.
It says that you can add a redirect rule, which redirects from your Netlify subdomain to your custom domain.
One benefit of this is related to SEO, because you avoid that a crawler scans both - your custom domain as well as the Netlify subdomain.

![Team Dashboard on Netlify](/img/netlify_dns-seohint.png){: .center-image }

In my case this would be no problem, because the canonical URL is set as described in the preparation part in the beginning of this post.
Nevertheless I don't want my published blog to be available on two different URLs.
Therefore I created the file `_redirects` in the root directory of my repository.
Further details on the Netlify redirect configuration is available in the [documentation][netlify-redirects].

```
https://baitando.netlify.com/* https://www.baitando.de/:splat 301!
```

Now this file needs to be included in the generated static website.
Using Jekyll this is achieved with the `include` setting in `_config.yml`. 

# Further Performance Tuning

I recognized that Netlify can do additional optimizations related to minification and lossless image compression.
If you want to try out such optimizations you can enable them in the site settings area.

![Asset Optimization Settings](/img/netlify_asset_optimization.png){: .center-image }

Please keep in mind, that changes in this configuration get effective with the next deployment.
To check the results immediately you need to trigger a new deployment manually.

# Status Badge

Netlify offers a status badge which shows the current build and deployment status of the website.
The markdown snippet for embedding the status badge in a markdown file is provided in the configuration section of the site in Netlify.
The area containing the snippet looks similar to the screenshot below.

![Netlify Status Badge Showing the Build and Deployment Status](/img/netlify_badge.png){: .center-image }

I added this status badge in the `README.md` file of my blog as you can see on [GitHub][baitando-repo].

# Pull Request Previews

One of the cool features provided by Netlify is the GitHub integration together with previews for pull requests.
Whenever a pull request is created, Netlify builds and deploys the sources in this branch and deploys the result to a separate Netlify subdomain.

Additional information is shown directly in the pull request like in the screenshot below, which was taken immediately after the pull request for this blog post was created.
Feel free to have a look at the pull request of this blog post directly on [GitHub][baitando-pr].

![Ongoing Netlify Pull Request Checks on GitHub](/img/netlify_pr-inprogress.png){: .center-image }

Once Netlify finished processing of this pull request, the box will look like the one below.
You will find some information related to build and deployment, which is equivalent to what you see in the Netlify dashboard.
If you click on the "Details" link in the row related to the deploy preview, you will be redirected to the Netlify subdomain to which the preview was deployed.

![Result of Netlify Pull Request Checks on GitHub](/img/netlify_pr-finished.png){: .center-image }

In my case the URL of the base URL of the deploy preview is [https://deploy-preview-2--baitando.netlify.com](https://deploy-preview-2--baitando.netlify.com).
According to the [documentation][netlify-deploypreview] the URL foolows the schema `https://deploy-preview-<pr-id>--<site-name>.netlify.com`.

Don't worry about SEO and duplicate content.
Like previously described, Netlify sends the `x-robots-tag: noindex` header for such deploy previews.
As a second measure, we made sure that the `canonical` link is set as described in the preparation part related to SEO pitfalls.

# Performance Improvements

Now let's check some numbers related to performance.
Due to the fact that my blog has not that many visitors, this is not that important.
Nevertheless I am a Software Engineer who is happy with good, stable solutions with good performance.

The numbers below are some values from loading one of the blog posts before and after the migration.
I simply opened the developer console in the browser and requested the same blog post several times to get a feeling for usual loading times which may vary from one request to another.
Then I picked the values of a request with a usual response time and wrote them down.
I think this gives at least an indication of how performance changed due to the migration.

First I disabled caching in the developer console of the browser and did some measurements in different scenarios.

| Scenario                                  | Load Time | Transferred | Requests |
|-------------------------------------------|-----------|-------------|----------|
| Old without caching                       | 1.79 s    | 1.3 MB      | 33       |
| New without caching without optimization  | 1.26 s    | 1.2 MB      | 32       |
| New without caching with optimization     | 1.19 s    | 1.2 MB      | 25       |

After that I enabled caching in the web console and did some more measurements.

| Scenario                                  | Load Time | Transferred | Requests |
|-------------------------------------------|-----------|-------------|----------|
| Old with caching                          | 675 ms    | 8.6 KB      | 33       |
| New with caching without optimization     | 634 ms    | 5.8 KB      | 32       |
| New with caching with optimization        | 433 ms    | 853 B       | 25       |

These values indicate that performance improved a bit.
This is what I hoped for, because I did not use a CDN before the migration and now switched to a solution with a CDN.
Based on these results it seems like it was at least not the wrong decision to move to Netlify from a performance point of view.

# Cleaning Up

Now that the migration itself finished, it is time to clean up.
I won't go into details here, but the things I had to do are listed below.

* Deactivate the repository on Travis.
* Remove the `.travis.yml` and `deploy_rsa.enc` files.
* Delete all data on the old webspace.

# Conclusion

In this post I described the necessary steps to migrate my blog to Netlify.
This should be more or less also valid for any other static website.

The migration was easily doable and the solution itself looks good from my current point of view.
I did not regret to migrate and I hope I won't in the future.

I will gather some more experience with writing my next blog posts.
If my current impression is confirmed, I will most likely migrate some other websites I do on a voluntary basis.

[baitando-repo]: https://github.com/baitando/blog
[baitando-pr]: https://github.com/baitando/blog/pull/2
[letsencrypt]: https://letsencrypt.org
[netlify-home]: https://www.netlify.com/
[netlify-signup]: https://app.netlify.com/signup
[netlify-pricing]: https://www.netlify.com/pricing/
[netlify-vcspermissions]: https://docs.netlify.com/configure-builds/repo-permissions-linking/
[netlify-buildcmds]: https://docs.netlify.com/configure-builds/common-configurations
[netlify-dns]: https://docs.netlify.com/domains-https/custom-domains/#assign-a-domain-to-a-site
[netlify-externaldns]: https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/
[netlify-redirects]: https://docs.netlify.com/routing/redirects/
[netlify-deploypreview]: https://docs.netlify.com/site-deploys/overview/#branches-and-deploys
[ionos-home]: https://www.ionos.de/
[jekyll-home]: https://jekyllrb.com/
[github-home]: https://github.com/
[travisci-home]: https://travis-ci.org/
[seo-steps]: https://www.codesections.com/blog/netlify/
[seo-post-1]: https://community.netlify.com/t/avoid-seo-duplicated-content-between-master-url-and-actual-production-url/850
[seo-post-2]: https://community.netlify.com/t/preferred-method-to-block-seo-for-branch-subdomains/2859
[seo-header]: https://developers.google.com/search/reference/robots_meta_tag
[seo-canonical]: https://support.google.com/webmasters/answer/139066

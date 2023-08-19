---
Title: Create a site like this for free
date: 2023-08-14 2:58PM
categories: [Misc]
---

## Introduction
Many people opt to use Wordpress or something like that, while Wordpress is certainly great, it is too powerful for what a site like this needs to do. The only thing this site does is read off Markdown file, convert it to HTML and display it with nice styling. There are other functions as well like search but that is completely powered by front end. All in all, if you just need a simple website like this, you may opt to use static site generators instead

There are many static site generators. Hugo, Jekyll, Gridsome, Eleventy, Pelician and much more. They are all great in their own regards. The one I use is called Jekyll because it integrates well with Github.

## Prequisites
* Functional knowledge of git and github. You should know how to create a Github repo, clone/pull and commit/push as well as [how to setup SSH keys](/posts/What-You-Need-To-Know-About-Using-SSH-With-Github-and-Gitlab)
* Need to have a Nepali citizenship to apply for free domain
* Knowhow of how Markdown works and how to write in one

## Setup your blog locally

First, follow the tutorials on [Jekyll website](https://jekyllrb.com/docs/installation/) to setup Ruby and Jekyll to work depending on your platform

Next, Follow the tutorials on [Jekyll with Chirpy](https://chirpy.cotes.page/posts/getting-started/)

Use the **Using the Chirpy Starter** option and follow along

I could have incorporated all that information in this article on how to setup step by step but it would not only be duplication of effort, but also things may get outdated in the future and you should always make sure to follow the official guide.

All in all, you should have a working blog site locally on your machine when you run ``bundle exec jekyll serve`` in your computer before you go on with more information.

Once you create and are happy with your blog, push it to the Github from where you cloned.

Some small side note:
* Leave ``url`` and ``baseurl`` in your ``_config.yml`` blank.
* Your newly forked Github repo must be public.
* ``timezone: Asia/Kathmandu`` without quotes.
* If you are running Windows and appear to be having issues, put the following in your ``_config.yml``

```yml
sass:
  sourcemap: never

parallel_localization: false
```

## Create a new account in Cloudflare
* Sign up to [Cloudflare](https://dash.cloudflare.com/sign-up)
* Log in
* Click on **Add a site** and place your domain name you wish to create
* Select the free plan on the bottom
* After you are done, you will see two Nameserver records provided by Cloudflare. Take a note on that.
Suppose the nameserver provided by cloudflare in my case were

```
boyname.ns.cloudflare.com
girlname.ns.cloudflare.com
```

## Claim your domain from register.com.np
* Go to [register.com.np](https://register.com.np) and claim the domain
* Put in the nameserver records received from Cloudflare in your **ns1** and **ns2** field
* Provide your details and citizenship and whatnot
* Wait few business days for it to be manually verified

Note: Say you added a site ``mysite.com.np`` in cloudflare but it got rejected by register.com.np. All you have to do is delete this site from Cloudflare and create a new site. You do not have to worry about the ns record you provided because it is common for all the sites you add in your account. On the plus side you will get an email when your .com.np domain got verified from Cloudflare.

## Setup Github Pages
* Go to the Repo settings in Github
* Go to ``Pages`` section from the left side of the screen
* Select ``Github Action`` in a dropdown under ``Build and Deployment``/``Source`` the default may be ``Deploy from a Branch``
* Do a git push one more time
* Now go to ``yourusername.github.io`` and your website should show up

You may right now put in your domain name in the ``Custom Domain`` section which is okay but it will not work unless you add DNS records in your cloudflare

## Setup DNS records in Cloudflare to point to Github
* Login to Cloudflare
* Select your site
* Select ``DNS Settings`` at the right side under ``Quick Actions``
* Add the following records,
```
| Record Type | Name              | Content                | Proxy Status | TTL   |
|-------------|-------------------|------------------------|--------------|-------|
| CNAME       | yourdomain.com.np | yourusername.github.io | DNS Only     | 5 min |
| CNAME       | www               | yourdomain.com.np      | Proxied      | Auto  |
```

## Enforce HTTPS in Github
If you have done everything right, now you can go to github pages section as previously. Wait about 5 minutes when the DNS check is pending. After the DNS check is confirmed, you can check on the ``Enforce HTTPS``. Remember to set Proxy status to DNS only or it will give you an error
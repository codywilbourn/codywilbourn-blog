---
title: "Migrating to Hugo"
date: 2018-06-04T15:58:55-05:00
---

I recently migrated my blog from Wordpress.com to a static site powered
by [Hugo](https://gohugo.io/) on [Netlify](https://www.netlify.com/).

While I had no problems with Wordpress.com with respect to hosting, ease
of setup or use, I did not care for the plugin situation. Specifically,
Wordpress has a very limited whitelist of Javascript functionality and plugins
on the personal plan, and this leads to some weirdness around behavior.

<!--more-->
For example, I can install the Wordpress Mailchimp plugin to collect
emails for post updates, but only the pop-up version. If a reader closes
the popup, the only way to get it back is to delete the cookie it sets--which
is done by no one ever. Mailchimp the product allows for a form to collect
emails, but HTML forms are disabled on Wordpress personal plan except the
contact form. That leaves the Wordpress version of "Get notified on post
updates" as a permanent submission form, meaning there are two different email
lists to maintain. That Wordpress form also received 5-7 spam signups per week,
unless I'm mistaken in how popular `outlook.com` addresses are among the tech
crowd these days.

I could have upgraded to the "Business" tier for around $30 per month to
fix the plugin situation, but that's a little rich for a personal blog.
Self-hosting a Wordpress install was not something I wanted to sign
myself up for--uptime, patching, scaling.

HostGator promises a similar CDN-cache situation to Wordpress.com,
along with the ability to install plugins on the basic plan,
though I was put off by the add-on fee for HTTPS. It's 2018, HTTPS with a
SNI cert should be in the standard offerings category and that felt like a
warning flag. Come to find out that [Hostgator still stores passwords in plaintext](https://www.reddit.com/r/sysadmin/comments/8bh07o/its_2018_and_hostgator_still_stores_passwords_in/),
and that made Hostgator a non-starter.

I was clued in to Netlify and also [Forestry.io](https://forestry.io) by some
tweets that Baron Schwartz ([@xaprb](https://twitter.com/xaprb))
had made about the service. The solution seemed like what I needed,
even though it would be a slightly heavier lift in migration.

# The Migration Process

I've reordered the steps here from what I did to make more sense and have
fewer back-and-forth jumps.

## Get the content out of Wordpress

There exists a [Wordpress-to-hugo-exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter), which
is a Wordpress plugin. Being on the Wordpress.com personal plan, I couldn't
install a custom plugin. There are instructions on setting up a Docker Compose
setup to mirror Wordpress.com, but it's marked as untested on the README and
requires matching the WP version on Wordpress.com.

Instead, I found [a blog post](http://justindunham.net/migrating-from-wordpress-to-hugo/)
referencing a project called [exitwp](https://github.com/thomasf/exitwp).
It seemed to do what I wanted, and being a Python project I figured I could
overcome any issues that cropped up. The goal was to export it into Jekyll,
and then have Hugo interpret that content.

ExitWP is a Python 2.7 application, download it and work out of the clone
directory.

{{< highlight shell >}}
mkvirtualenv exitwp
pip install -r pip_requirements.txt
{{< / highlight >}}

ExitWP has a configuration file, called `config.yaml`. Inside there's a
`body_replace` section, which does additional text substitutions. I chose
to replace `[code]` with the `highlight` shortcode for Jekyll.

{{< highlight yaml >}}
body_replace: {
    '\[code.*="(.*)"\]': '{% highlight \1 %}',
    '\[\/code\]': '{% endhighlight %}',
}
{{< / highlight >}}

Wordpress.com allows site owners to export their content to XML. That XML
file goes into the `wordpress-xml` directory of `exitwp`. Calling ExitWP with
`python exitwp.py` will extract the blog posts to
`build/jekyll/$DOMAIN`.

This is a bit of a guess and check number based on the syntax used in the blog
posts. I originally tried converting the `[caption]` tags to HTML, but couldn't
get the syntax right. I decided the few instances of the caption tag weren't
worth trying to fix my regex, and elected to continue and fix that manually
later.

The Wordpress export will not export images, so those still needed to be
downloaded.

Some grep-xargs magic to download them. `wget -P` sets the download directory.
I had performed this step after setting up Hugo and getting the content
mostly imported, which is why `static/img` is the download directory.

{{< highlight shell >}}
grep files.wordpress.com $POSTS_DIRECTORY | sed 's/.*(\(https:.*\)).*/\1/' |  xargs -I{} wget -P static/img/ {}
{{< / highlight >}}

Then all of the posts needed updating to be relative links

{{< highlight shell >}}
# In-place edit to the relative url path, which is img for me
find . -type f -exec sed -i.bak 's/https.*.wordpress.com\/20[0-9][0-9]\/[0-9][0-9]\/\(.*\)/\/img\/\1/' {} \;
# Check everything, then remove the .bak files
{{< / highlight >}}

## Setup Hugo

Hugo is easy to setup on a Mac. `brew install hugo`.

After Hugo was installed, I could import the "Jekyll" blog I had.

{{< highlight shell >}}
hugo import jekyll $PATH_TO_EXITWP/build/jekyll/$DOMAIN /tmp/blog
{{< / highlight >}}

I wasn't sure what sort of trash could get generated by the import
(ends up none at all, it's a valid install), but the tool wants a
clean directory. Importing directly into my empty repo from Github was
a no-no, so I set it up in temp and then moved the files into the folder
destined for Github.

Hugo takes a `config.toml` file, which is where themes are specified,
along with template variables, which as far as I can tell is called
"front matter". I spent a bit of time learning how that works.


## Clean Up Markdown
With the double conversion I performed I ended up with some weird Markdown.
It seems that most lines containing a `*`, like bullet points or emphasis,
contained extra newlines. Rendering-wise, this changes a `<li>text</li>` to
`<li><p>text</p></li>`. Not a huge deal.

I also had to clean up the markdown to use some Hugo-specific shortcodes.

* Those image captions were changed to be captions on the "figure" shortcode
* Youtube, twitter shortcodes for embeds, rather than Wordpress's
* Update all links to be relative

The last one was a big update. Wordpress.com lets you easily link to other
pages, but the export is all via permalinks. If I ever wanted to change the
URL structure, the links would break. So I went and replaced each link with
the Hugo shortcode, `relref $page`. The value of `page` is the name
of the Markdown file, without the extension. With the export I had performed,
the post titles were all of the format `yyyy-mm-dd-slug`, which meant I could
convert the old url of `yyyy/mm/dd/slug` with some find & replaces.

## Selecting a Theme

I went with the "Ananke" theme from the Hugo tutorial. As a starting point,
it has a bunch of features and looks fairly decent, though it will need
a few changes. In particular, I'm not thrilled with stock inline code
formatting and its mobile/iPad layout. The [tachyons.io](http://tachyons.io/)
CSS framework class it uses will take some getting used to.

I replaced the default hero image with an image from [Subtle Patterns](http://subtlepatterns.com).

I spent some time understanding how the theme is rendered and overridden.
Specifically, anything in the `layouts` directory will overwrite whatever is
is in `themes/<themename>/layouts/`. It's a nice method to override your
particular theme and allow for updates of it, but that customization doesn't
appear to lend itself to frequent testing of new themes. I copied a few of the
page partials and modified them to add a basic Mailchimp subscription form at
the bottom of my posts as a way of playing with the templating system.

## Local Testing

Hugo comes with a quick server to experiment with changes. `hugo server`
will start a server on `localhost:1313`. It watches for file modifications and
automatically refreshes your browser on changes.

# Deploying to Netlify


## Build Settings
The Netlify account setup was fairly simple. Authorize against Github
and point the builds at your repo.

The build configuration was a bit tricky because screenshots get out
of date. What ended up working for me was to use the `netlify.toml` file
to dictate the configuration. With Hugo's recent change, the `public`
directory only gets created when `HUGO_ENV=production`. Setting
the `HUGO_VERSION` means I won't undergo any unintended upgrades.

{{< highlight toml >}}
[build]
publish = "public"
command = "hugo"

[context.production.environment]
HUGO_VERSION = "0.41"
HUGO_ENV = "production"
{{< / highlight >}}

## Netlify Forms

Netlify allows you to have forms embedded on your site, something mostly
restricted by Wordpress.com. As stated in their documentation, forms
must contain a special `netlify` keyword: `<form name='contact' netlify>`
and Netlify's servers will substitute appropriate parameters.

Netlify forms can be forwarded to Slack, Webhook, or email, so choosing
email gave the same behavior as the Wordpress contact form.

## Netlify Testing

At this point, the Netlify site built and had a randomly assigned subdomain
on netlify.com. This let me test most things, though my blog `config.toml`
had a few values set off the intended `baseUrl`, meaning my choice was to
either update the template to be relative urls, or wait until production
to verify.

## Migrating DNS

I moved from the Wordpress name servers to the Netlify nameservers. The DNS
functionality is currently in beta, and it shows when compared to other
DNS providers. For example, you cannot import a zone file or CSV, meaning
every record must be added by hand. Fixing a mistake in MX priority or TTL
requires deleting and re-adding the record. On that note, TTL could not be
managed in Wordpress, so I gained some neat functionality. I expect management
will get easier after Netlify drops the `beta` moniker.

I had some problems with stale upstream DNS caches that lasted about 16-20
hours. Instead of loading my new site, it served up the old IP addresses.
There's not much anyone can do about that other than wait or not change
name servers.

## Enabling SSL

Netlify provides a Let's Encrypt certificate on their CDN for your site.
I enabled it, though had some issues where it added a cert for `*.netlify.com`
(i.e. for my freebie subdomain) rather than `codywilbourn.com`. I wrote
into support and they corrected the issue same-day.

After the cert issue was ironed out, I was able to enforce HTTPS.

# Overall

Hugo has a wonderful quick-start and the ease of dropping in a few themes to
play with was similar to the first-run experience of Wordpress, albeit with
more command line involvement. I'm no designer, so the selection of templates
went a long way to help get me started.

Netlify was incredibly simple to get started with, once I got the
configuration. The DNS management, one-click SSL and HTTPS reinforcement was
so easy to work with.

I mentioned Forestry.io and I intend to use the service after
understanding Hugo and my site layout more. A CMS for a static site
generator is incredibly exciting, though I'm enjoying writing this blog post
in raw Markdown. Wordpress.com personal tier has Markdown support, but it's
not the same feeling as other Markdown editors.

If you're curious and would like to check out this site's source code,
you can find it on [github.com/codywilbourn/codywilbourn-blog](github.com/codywilbourn/codywilbourn-blog)


